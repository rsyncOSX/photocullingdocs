+++
author = "Thomas Evensen"
title = "Sequrity"
date = "2026-02-05"
tags = ["sequrity"]
categories = ["technical details"]
+++

## Security Scoped URLs: Architecture & Implementation Details

### Overview

Security-scoped URLs are a cornerstone of macOS app sandbox security. PhotoCulling uses them extensively to gain persistent access to user-selected folders and files while maintaining sandbox compliance. This section provides a comprehensive walkthrough of how they work in the application.

### What Are Security-Scoped URLs?

A security-scoped URL is a special form of file URL that:
- Can be created only from user-granted file access (via file pickers or drag-drop)
- Grants an app temporary or persistent access to files outside the app sandbox
- Must be explicitly "accessed" and "released" to work properly
- Can optionally be serialized as a "bookmark" for persistent access

**Key API:**
```swift
// Start accessing a security-scoped URL (required before file operations)
url.startAccessingSecurityScopedResource() -> Bool

// Stop accessing it (must be paired)
url.stopAccessingSecurityScopedResource()

// Serialize for persistent storage
try url.bookmarkData(options: .withSecurityScope, ...)

// Restore from serialized bookmark
let url = try URL(resolvingBookmarkData: bookmarkData, 
                  options: .withSecurityScope, ...)
```

### Architecture in PhotoCulling

PhotoCulling implements a multi-layer security-scoped URL system with two primary workflows:

#### Layer 1: Initial User Selection (OpencatalogView)

When users select a folder via the file picker, `OpencatalogView` handles the initial security setup:

**File:** [PhotoCulling/Views/CopyFiles/OpencatalogView.swift](PhotoCulling/Views/CopyFiles/OpencatalogView.swift)

```swift
struct OpencatalogView: View {
    @Binding var selecteditem: String
    @State private var isImporting: Bool = false
    let bookmarkKey: String  // e.g., "destBookmark"
    
    var body: some View {
        Button(action: { isImporting = true }) {
            Image(systemName: "folder.fill")
        }
        .fileImporter(isPresented: $isImporting,
                      allowedContentTypes: [.directory],
                      onCompletion: { result in
                          handleFileSelection(result)
                      })
    }
    
    private func handleFileSelection(_ result: Result<URL, Error>) {
        switch result {
        case let .success(url):
            // STEP 1: Start accessing immediately after selection
            guard url.startAccessingSecurityScopedResource() else {
                Logger.process.errorMessageOnly("Failed to start accessing resource")
                return
            }
            
            // STEP 2: Store the path for immediate use
            selecteditem = url.path
            
            // STEP 3: Create and persist bookmark for future launches
            do {
                let bookmarkData = try url.bookmarkData(
                    options: .withSecurityScope,
                    includingResourceValuesForKeys: nil,
                    relativeTo: nil
                )
                // Store bookmark in UserDefaults
                UserDefaults.standard.set(bookmarkData, forKey: bookmarkKey)
                Logger.process.debugMessageOnly("Bookmark saved for key: \(bookmarkKey)")
            } catch {
                Logger.process.warning("Could not create bookmark: \(error)")
            }
            
            // STEP 4: Stop accessing (will be restarted when needed)
            url.stopAccessingSecurityScopedResource()
            
        case let .failure(error):
            Logger.process.errorMessageOnly("File picker error: \(error)")
        }
    }
}
```

**Key Points:**
- ✅ Access/release happen in the same scope (guaranteed cleanup)
- ✅ Bookmark created while resource is being accessed (more reliable)
- ✅ Path stored in `@Binding` for immediate UI feedback
- ⚠️ Access is briefly held (during bookmark creation), then released

#### Layer 2: Persistent Restoration (ExecuteCopyFiles)

When the app needs to use previously selected folders, `ExecuteCopyFiles` restores access from the bookmark:

**File:** [PhotoCulling/Model/ParametersRsync/ExecuteCopyFiles.swift](PhotoCulling/Model/ParametersRsync/ExecuteCopyFiles.swift)

```swift
@Observable @MainActor
final class ExecuteCopyFiles {
    func getAccessedURL(fromBookmarkKey key: String, 
                       fallbackPath: String) -> URL? {
        // STEP 1: Try to restore from bookmark first
        if let bookmarkData = UserDefaults.standard.data(forKey: key) {
            do {
                var isStale = false
                
                // Resolve bookmark with security scope
                let url = try URL(
                    resolvingBookmarkData: bookmarkData,
                    options: .withSecurityScope,
                    relativeTo: nil,
                    bookmarkDataIsStale: &isStale
                )
                
                // STEP 2: Start accessing the resolved URL
                guard url.startAccessingSecurityScopedResource() else {
                    Logger.process.errorMessageOnly(
                        "Failed to start accessing bookmark for \(key)"
                    )
                    return tryFallbackPath(fallbackPath, key: key)
                }
                
                Logger.process.debugMessageOnly(
                    "Successfully resolved bookmark for \(key)"
                )
                
                // Check if bookmark became stale (update if needed)
                if isStale {
                    Logger.process.warning("Bookmark is stale for \(key)")
                    // Optionally refresh bookmark here
                }
                
                return url
                
            } catch {
                Logger.process.errorMessageOnly(
                    "Bookmark resolution failed for \(key): \(error)"
                )
                return tryFallbackPath(fallbackPath, key: key)
            }
        }
        
        // STEP 3: Fallback to direct path access if no bookmark
        return tryFallbackPath(fallbackPath, key: key)
    }
    
    private func tryFallbackPath(_ fallbackPath: String, 
                                key: String) -> URL? {
        Logger.process.warning(
            "No bookmark found for \(key), attempting direct path access"
        )
        
        let fallbackURL = URL(fileURLWithPath: fallbackPath)
        
        // Try direct path access (works if recently accessed)
        guard fallbackURL.startAccessingSecurityScopedResource() else {
            Logger.process.errorMessageOnly(
                "Failed to access fallback path for \(key)"
            )
            return nil
        }
        
        Logger.process.debugMessageOnly(
            "Successfully accessed fallback path for \(key)"
        )
        
        return fallbackURL
    }
}
```

**Key Points:**
- ✅ Tries bookmark first (most reliable)
- ✅ Falls back to direct path if bookmark fails
- ✅ Detects stale bookmarks via `isStale` flag
- ✅ Starts access only after successful resolution
- ⚠️ Caller is responsible for stopping access after use

#### Layer 3: Active File Operations (ScanFiles)

When scanning files, the security-scoped URL access is properly managed:

**File:** [PhotoCulling/Actors/ScanFiles.swift](PhotoCulling/Actors/ScanFiles.swift)

```swift
actor ScanFiles {
    func scanFiles(url: URL) async -> [FileItem] {
        // CRITICAL: Must start access before any file operations
        guard url.startAccessingSecurityScopedResource() else {
            return []
        }
        
        // Guarantee cleanup with defer (Swift best practice)
        defer { url.stopAccessingSecurityScopedResource() }
        
        // Now safe to access files
        let manager = FileManager.default
        let contents = try? manager.contentsOfDirectory(
            at: url,
            includingPropertiesForKeys: [...],
            options: [.skipsHiddenFiles]
        )
        
        // Process contents and return
        return processContents(contents)
    }
}
```

**Key Points:**
- ✅ Uses `defer` for guaranteed cleanup
- ✅ Access is granted only during actual file operations
- ✅ Prevents leaking security-scoped access
- ✅ Actor isolation ensures thread-safe operations

### Complete End-to-End Flow

```
User selects folder via picker
    ↓
[OpencatalogView]
    1. startAccessingSecurityScopedResource()
    2. Store path in UI binding
    3. Create bookmark from URL
    4. Save bookmark to UserDefaults
    5. stopAccessingSecurityScopedResource()
    ↓
[Later: User initiates copy task]
    ↓
[ExecuteCopyFiles.performCopyTask()]
    1. getAccessedURL(fromBookmarkKey: "destBookmark", ...)
        a. Retrieve bookmark from UserDefaults
        b. URL(resolvingBookmarkData:options:.withSecurityScope)
        c. url.startAccessingSecurityScopedResource()
        d. Return accessed URL (or nil)
    2. Append URL path to rsync arguments
    3. Execute rsync process
    ↓
[ScanFiles.scanFiles()]
    1. url.startAccessingSecurityScopedResource()
    2. defer { url.stopAccessingSecurityScopedResource() }
    3. Scan directory contents
    4. Return file items
    ↓
[After operations complete]
    Access is automatically cleaned up via defer/scope
```

### Security Model

PhotoCulling's security-scoped URL implementation adheres to Apple's sandbox guidelines:

| Aspect | Implementation | Benefit |
|--------|----------------|---------|
| **User Consent** | Files only accessible after user selection in picker | User controls what app can access |
| **Persistent Access** | Bookmarks serialized for cross-launch access | UX: Users don't re-select folders each launch |
| **Temporary Access** | Access explicitly granted/revoked with start/stop | Resources properly released after use |
| **Scope Management** | `defer` ensures cleanup even on errors | Prevents resource leaks |
| **Fallback Strategy** | Direct path access if bookmark fails | Graceful degradation |
| **Audit Trail** | OSLog captures all access attempts | Security debugging and compliance |

