+++
author = "Thomas Evensen"
title = "Number of files"
date = "2026-02-01"
tags = ["number of files"]
categories = ["technical details"]
+++

Numbers updated: February 1, 2026 (version 0.5.0). There is also a [Quality Analysis Report](https://github.com/rsyncOSX/PhotoCulling/blob/main/QUALITY_ANALYSIS.md) of the code for the app.

PhotoCulling depends only on the standard Swift and SwiftUI toolchain—no external libraries.

```
cloc DecodeEncodeGeneric/Sources ParseRsyncOutput/Sources RsyncArguments/Sources PhotoCulling/PhotoCulling RsyncProcessStreaming
/Sources
      74 text files.
      74 unique files.                              
       7 files ignored.

github.com/AlDanial/cloc v 2.08  T=0.02 s (3017.8 files/s, 307041.5 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Swift                           74           1036           1044           5449
-------------------------------------------------------------------------------
SUM:                            74           1036           1044           5449
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

  