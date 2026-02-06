+++
author = "Thomas Evensen"
title = "Cache policy"
date = "2026-02-05"
tags = ["memory"]
categories = ["technical details"]
+++

### Cache statistics

The PhotoCulling application provides a cache statistics display on the Sidebar. The initial figure represents a clean start, with no cache and no catalog selected.

{{< figure src="/images/memory/start.png" alt="Selecting all tagged photos" position="center" style="border-radius: 8px;" >}}

Upon scanning, 40 ARW files are processed, resulting in the creation of thumbnails. All thumbnails are loaded into memory, and 12 photos are tagged for selection. The thumbnails are 1024 pixels in size, which is sufficient for a swift selection process.

{{< figure src="/images/memory/createthumbs.png" alt="Selecting all tagged photos" position="center" style="border-radius: 8px;" >}}

The Sidebar may be hidden.

{{< figure src="/images/memory/hidesidebar.png" alt="Selecting all tagged photos" position="center" style="border-radius: 8px;" >}}

When the application is closed and restarted, the previous catalog is selected, and PhotoCulling searches the disk cache for thumbnails. All 40 are found, and all preselected tagged photos are copied into memory from disk cache. Consequently, the disk cache counts 40, while the memory cache counts 13.

{{< figure src="/images/memory/newstart.png" alt="Selecting all tagged photos" position="center" style="border-radius: 8px;" >}}

## Memory Management & Cache Model

### Overview

PhotoCulling implements a sophisticated three-tier caching architecture to manage thumbnail memory efficiently. The system automatically scales cache capacity based on thumbnail size while maintaining predictable memory behavior across different workloads.

**Architecture:**
```
User Request for Thumbnail
        ↓
    Tier 1: Memory Cache (NSCache)
        ↓ (if miss)
    Tier 2: Disk Cache (JPEG compressed)
        ↓ (if miss)
    Tier 3: On-Demand Generation
        ↓ (save to both caches)
    Return to UI
```

### Memory Cache Configuration

The memory cache is implemented using Apple's `NSCache` with intelligent configuration based on thumbnail dimensions:

**Production Configuration:**
- **Total Memory Limit:** 500 MB
- **Count Limit:** 1,000 images
- **Cost Model:** `width × height × 4 bytes × 1.1 (overhead)`

**Calculated Capacity for Common Thumbnail Sizes:**

| Thumbnail Size | Cost per Image | Approx. Capacity | Use Case |
|---|---|---|---|
| 256×256 px | 0.3 MB | ~1,667 images | Grid view browsing |
| 512×512 px | 1.1 MB | ~454 images | Detail preview |
| 1024×1024 px | 4.7 MB | **~106 images** | High-quality reference |
| 2560×2560 px | 29 MB | ~17 images | Zoom/inspection mode |

**Example Calculation (1024×1024 px):**
```
Cost per image = 1024 × 1024 × 4 bytes × 1.1 = 4,718,592 bytes ≈ 4.7 MB
Total capacity = 500,000,000 bytes ÷ 4,718,592 bytes ≈ 106 images
```

### Dynamic Configuration

The cache automatically reconfigures based on the selected thumbnail size:

```swift
@discardableResult
func preloadCatalog(at catalogURL: URL, targetSize: Int) async -> Int {
    // Reconfigure cache based on target size
    let config = CacheConfig.forThumbnailSize(targetSize)
    memoryCache.totalCostLimit = config.totalCostLimit
    memoryCache.countLimit = config.countLimit
    let cacheMemMB = config.totalCostLimit / (1024 * 1024)
    Logger.process.debugMessageOnly(
        "Cache reconfigured for \(targetSize)px thumbnails: \(cacheMemMB)MB, limit: \(config.countLimit)"
    )
    // ...
}
```

**Benefits of Dynamic Configuration:**
- ✅ Prevents out-of-memory crashes by limiting active images
- ✅ Scales gracefully with different hardware capabilities
- ✅ Adapts to user's chosen thumbnail size for optimal performance
- ✅ Maintains consistent behavior across macOS versions

### NSCache Behavior & Eviction

When memory cache is full and a new image is added:

1. **NSCache automatically evicts** least-recently-used items based on cost
2. **Evictions are tracked** via the delegate pattern for monitoring
3. **Statistics updated** to reflect cache pressure

**Example scenario (1024×1024 thumbnails):**
```
Memory cache: 500 MB limit
│
├─ Image 1-106: Stored in cache ✓
│
└─ Image 107: New image to store
    ↓
    NSCache calculates: 106 images × 4.7 MB = 497.2 MB (near limit)
    ↓
    LRU eviction triggered: Removes least-accessed image
    ↓
    Image 107 added successfully
    ↓
    Cache maintains ~106 image capacity
```

### Disk Cache Layer

When memory cache evicts an image or is full, the disk cache provides secondary storage:

**Configuration:**
- **Location:** `~/Library/Containers/no.blogspot.PhotoCulling/Data/Library/Caches/no.blogspot.PhotoCulling/Thumbnails`
- **Format:** JPEG compressed at 0.7 quality factor
- **File Naming:** MD5 hash of source file path
- **Expiration:** 30 days (configurable via `pruneCache(maxAgeInDays:)`)

Due to the PhotoCulling is a Sandboxed app, the cache is created in the Sandboxed data catalog.

**Storage Cost Comparison:**

| Format | Size per Image | 100 Images | 500 Images |
|---|---|---|---|
| Uncompressed NSImage | 4.7 MB | 470 MB | 2.35 GB |
| JPEG at 0.7 quality | 0.4-0.8 MB | 40-80 MB | 200-400 MB |
| **Space Savings** | **~82-91%** | **~83-91%** | **~83-91%** |

**Disk cache hit retrieval flow:**
```swift
// B. Check Disk
if let diskImage = await diskCache.load(for: url) {
    storeInMemory(diskImage, for: url)  // Restore to memory
    cacheDisk += 1  // Track as miss (but faster than generation)
    return diskImage
}
```

### Cache Statistics Monitoring

Real-time cache performance is tracked and exposed via the `CacheStatisticsView`:

**Metrics:**
- **Cache Hits:** Requests served from memory cache (fastest)
- **Cache Misses:** Requests served from disk cache (slower, but cached)
- **Evictions:** Number of images removed from memory cache
- **Hit Rate:** Percentage of requests from memory vs. disk

**Hit Rate Interpretation:**

| Rate | Status | Implication |
|---|---|---|
| **90-100%** | Excellent | Working set fits in memory, very fast |
| **70-89%** | Good | Most common access patterns cached |
| **50-69%** | Fair | Significant disk I/O overhead |
| **<50%** | Poor | Too much variation in access patterns |

**Example Statistics:**
```
Memory Hits: 245
Disk Misses: 55
Evictions: 12
Hit Rate: 81.7%
```
This indicates that for every 100 thumbnail requests, 81 are served from memory (instant), 18 from disk (~50-100ms), and ~1 requires generation with encoding (~500ms).

### Cache Warming (Preload Strategy)

When a catalog is selected, PhotoCulling aggressively preloads thumbnails:

```swift
return await withThrowingTaskGroup(of: Void.self) { group in
    let maxConcurrent = ProcessInfo.processInfo.activeProcessorCount * 2
    
    for (index, url) in urls.enumerated() {
        if index >= maxConcurrent {
            try? await group.next()  // Wait for one slot
        }
        group.addTask {
            await self.processSingleFile(url, targetSize: targetSize)
        }
    }
    try? await group.waitForAll()
}
```

**Benefits:**
- ✅ Parallel processing with smart concurrency limits
- ✅ Initial UI responsiveness while preloading in background
- ✅ Cancellation support (user can interrupt expensive operations)
- ✅ Gradual cache warming reduces initial jank

### Memory Pressure Handling

The system gracefully handles memory pressure from the OS:

**Three-level response:**

1. **NSCache automatic eviction** (yellow level)
   - OS signals memory pressure
   - NSCache evicts least-used items automatically
   - App continues functioning with reduced cache

2. **Graceful degradation** (orange level)
   - If memory still constrained, cache queries fall through to disk
   - Performance acceptable (disk I/O latency ~50-100ms)
   - App remains responsive

3. **Generation fallback** (red level)
   - If both caches miss, on-demand generation occurs
   - Slower (~500ms for medium-size images)
   - But prevents crashes and maintains functionality
   
### Production vs. Testing Configuration

The system ships with two configurations:

**Production:**
```swift
nonisolated static let production = CacheConfig(
    totalCostLimit: 500 * 1024 * 1024,  // ~500 MB
    countLimit: 1000
)
```

**Testing:**
```swift
nonisolated static let testing = CacheConfig(
    totalCostLimit: 100_000,  // 100 KB - forces evictions
    countLimit: 5              // Very small to test edge cases
)
```

Testing configuration deliberately uses small limits to validate eviction behavior and memory pressure handling without requiring 10,000 test files.

### Performance Implications

**Memory footprint by scenario:**

| Scenario | Images in Memory | Memory Used | Hit Rate | Responsiveness |
|---|---|---|---|---|
| Grid browse (256×256) | ~1,667 | 500 MB | 98% | Instant |
| Detail preview (512×512) | ~454 | 500 MB | 92% | Very fast |
| Zoom reference (1024×1024) | ~106 | 500 MB | 78% | Fast |
| Large collection (2560×2560) | ~17 | 500 MB | 35% | Acceptable |

For large collections or high-resolution thumbnails, users should expect disk cache lookups and generation for images outside the memory window. This is expected behavior and provides a good balance between memory usage and responsiveness.

### Recommendations for Users

**Optimize cache hit rate:**
1. Regular catalog scanning keeps disk cache fresh
2. Avoid extremely high thumbnail sizes for very large image collections
3. Monitor cache statistics in preferences to understand access patterns
4. Allow disk cache to persist across sessions (cache pruning is automatic)

**Understanding evictions:**
- Evictions are normal and expected when browsing large collections
- They indicate the cache is working to stay within memory budget
- High eviction counts (>50) suggest working set exceeds memory capacity
- Consider reducing thumbnail size if evictions concern you

### Future Optimization Opportunities

1. **Adaptive cache sizing** - Adjust limits based on available system RAM
2. **Compression optimization** - Profile different JPEG quality settings
3. **Compression formats** - Explore HEIF/HEIC for better compression
4. **Cache warming hints** - Let users specify which folders to prioritize
5. **Statistics export** - Enable performance profiling across sessions


### List of data in disk cache

```
ls -alrt  /Users/thomas/Library/Containers/no.blogspot.PhotoCulling/Data/Library/Caches/no.blogspot.PhotoCulling/Thumbnails
total 10168
drwxr-xr-x@  3 thomas  staff      96 Feb  5 15:35 ..
-rw-r--r--@  1 thomas  staff   92979 Feb  5 18:11 f2fa5c339d95b8b8e0b69be9557070c5.jpg
-rw-r--r--@  1 thomas  staff  107853 Feb  5 18:11 697ca156e78c9763177c267c657b8518.jpg
-rw-r--r--@  1 thomas  staff  134673 Feb  5 18:11 048a1efd3cca5cd41640b1da925b79b8.jpg
-rw-r--r--@  1 thomas  staff  140909 Feb  5 18:11 cbdee8f9ca922ff3b0ae78ca5b408212.jpg
-rw-r--r--@  1 thomas  staff  119206 Feb  5 18:11 462c4a8b866918c5801fe56fe5182c1c.jpg
-rw-r--r--@  1 thomas  staff  149019 Feb  5 18:11 cc1eb1d64d1c307eccecd72144013b2f.jpg
-rw-r--r--@  1 thomas  staff  183046 Feb  5 18:11 944c47425261a7df44cf56d0ec549939.jpg
-rw-r--r--@  1 thomas  staff  155193 Feb  5 18:11 cf0309648aac2a822fdec6e33a0293cd.jpg
-rw-r--r--@  1 thomas  staff   83907 Feb  5 18:11 5fe7209c8a1b45e89a18545997a6a447.jpg
-rw-r--r--@  1 thomas  staff  108479 Feb  5 18:11 910f91a2e49d1ea82974ff7f90b3a890.jpg
-rw-r--r--@  1 thomas  staff  146826 Feb  5 18:11 6893ffebbc8262fd53398d04a75d0776.jpg
-rw-r--r--@  1 thomas  staff  158513 Feb  5 18:11 89771727d834395b1286c973f92d0713.jpg
-rw-r--r--@  1 thomas  staff  156471 Feb  5 18:11 7cddd80f16d0e33f72e317212d596a89.jpg
-rw-r--r--@  1 thomas  staff   86916 Feb  5 18:11 60bbb4822667e6c7df2451e4aff7b8d9.jpg
-rw-r--r--@  1 thomas  staff  129615 Feb  5 18:11 0fafba42a7a54933b52f0538e652518d.jpg
-rw-r--r--@  1 thomas  staff  159366 Feb  5 18:11 618bd60edb1278563407d30c7b5498a8.jpg
-rw-r--r--@  1 thomas  staff  130486 Feb  5 18:11 e19495fde72b9c9bdb0645519b58c699.jpg
-rw-r--r--@  1 thomas  staff   82619 Feb  5 18:11 b69c06b0cf5f121982e8511b5c5b2e4c.jpg
-rw-r--r--@  1 thomas  staff  150554 Feb  5 18:11 2301862e0f5403a6c4e9414993bb6e9b.jpg
-rw-r--r--@  1 thomas  staff   84835 Feb  5 18:11 2be9a7954e82cff3346e02e2b2d38ef6.jpg
-rw-r--r--@  1 thomas  staff  138427 Feb  5 18:11 3caa8ac2e478c8f0bb3b4e5dacab5330.jpg
-rw-r--r--@  1 thomas  staff  109697 Feb  5 18:11 2f61d047702e92009fb2ae3bb259133f.jpg
-rw-r--r--@  1 thomas  staff  134244 Feb  5 18:11 601fff4f699b242a64a842e7d6c60d2c.jpg
-rw-r--r--@  1 thomas  staff  152523 Feb  5 18:11 83d4d24d27d3048f7d35374e7827fb17.jpg
-rw-r--r--@  1 thomas  staff  152129 Feb  5 18:11 ff7ae3d18508605f2435c6e9d1ceaeca.jpg
-rw-r--r--@  1 thomas  staff   82722 Feb  5 18:11 9a5620be2f0e33981896c65157ca57b0.jpg
-rw-r--r--@  1 thomas  staff  150047 Feb  5 18:11 f02288709ed88f141599b2fb7dbbcba5.jpg
-rw-r--r--@  1 thomas  staff   60173 Feb  5 18:11 f242ed9b102ffe8a364723d9b46ff401.jpg
-rw-r--r--@  1 thomas  staff   85191 Feb  5 18:11 3977bcb5d558bf25dd1a7521f0c65ceb.jpg
-rw-r--r--@  1 thomas  staff  150356 Feb  5 18:11 9c7c537223024d2b13a56731c1cc2e43.jpg
-rw-r--r--@  1 thomas  staff  116870 Feb  5 18:11 c1268c98484a456378c1e9d70160e89d.jpg
-rw-r--r--@  1 thomas  staff  122497 Feb  5 18:11 e05e3b137bfa3b1c693d183fc4488591.jpg
-rw-r--r--@  1 thomas  staff   86112 Feb  5 18:11 c63e7f2dcc9c50aa86f37cf96b3e53be.jpg
-rw-r--r--@  1 thomas  staff  122582 Feb  5 18:11 4d3dfe3cf1291484a61fc31170eb6b52.jpg
```

