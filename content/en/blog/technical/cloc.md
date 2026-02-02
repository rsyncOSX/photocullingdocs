+++
author = "Thomas Evensen"
title = "Number of files"
date = "2026-02-02"
tags = ["number of files"]
categories = ["technical details"]
+++

Numbers updated: February 2, 2026 (version 0.5.0). There is also a [Quality Analysis Report](https://github.com/rsyncOSX/PhotoCulling/blob/main/QUALITY_ANALYSIS.md) of the code for the app.

PhotoCulling depends only on the standard Swift and SwiftUI toolchain—no external libraries.

```
cloc DecodeEncodeGeneric/Sources ParseRsyncOutput/Sources RsyncArguments/Sources PhotoCulling/PhotoCulling RsyncProcessSt
reaming/Sources
      74 text files.
      74 unique files.                              
       9 files ignored.

github.com/AlDanial/cloc v 2.08  T=0.03 s (2218.8 files/s, 223771.8 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Swift                           74           1026           1043           5394
-------------------------------------------------------------------------------
SUM:                            74           1026           1043           5394
-------------------------------------------------------------------------------
```

#### Main Repository

- PhotoCulling (https://github.com/rsyncOSX/PhotoCulling) - the main repository for PhotoCulling

##### Swift Packages used by RsyncUI

All packages track the `main` branch and are updated to latest revisions as of v0.5.0:

1. **DecodeEncodeGeneric** - Generic JSON codec
   - Repository: https://github.com/rsyncOSX/DecodeEncodeGeneric
   - Purpose: Reusable JSON encoding/decoding utilities

2. **ParseRsyncOutput** - Rsync output parser
   - Repository: https://github.com/rsyncOSX/ParseRsyncOutput
   - Purpose: Extract statistics from rsync output
     
3. **RsyncArguments** - Rsync argument builder
   - Repository: https://github.com/rsyncOSX/RsyncArguments
   - Purpose: Type-safe rsync command generation

4. **RsyncProcessStreaming** - Streaming process handler
   - Repository: https://github.com/rsyncOSX/RsyncProcessStreaming
   - Purpose: Real-time rsync output streaming and progress tracking

  