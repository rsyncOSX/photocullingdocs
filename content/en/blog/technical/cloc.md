+++
author = "Thomas Evensen"
title = "Number of files"
date = "2026-02-05"
tags = ["number of files"]
categories = ["technical details"]
+++

Numbers updated: February 5, 2026 (version 0.6.1). There is also a [Quality Analysis Report](https://github.com/rsyncOSX/PhotoCulling/blob/main/QUALITY_ANALYSIS.md) of the code for the app.

PhotoCulling depends only on the standard Swift and SwiftUI toolchain—no external libraries.

```
cloc DecodeEncodeGeneric/Sources ParseRsyncOutput/Sources RsyncArguments/Sources PhotoCulling/PhotoCulling RsyncProcessStreaming/Sources
      73 text files.
      73 unique files.                              
       7 files ignored.

github.com/AlDanial/cloc v 2.08  T=0.02 s (2984.0 files/s, 325129.1 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Swift                           73           1092           1085           5777
-------------------------------------------------------------------------------
SUM:                            73           1092           1085           5777
-------------------------------------------------------------------------------
```

#### Main Repository

- PhotoCulling (https://github.com/rsyncOSX/PhotoCulling) - the main repository for PhotoCulling

##### Swift Packages used by RsyncUI

All packages track the `main` branch and are updated to latest revisions as of v0.6.1:

1. **RsyncProcessStreaming** - Streaming process handler
   - Repository: https://github.com/rsyncOSX/RsyncProcessStreaming
   - Purpose: Real-time rsync output streaming and progress tracking

2. **DecodeEncodeGeneric** - Generic JSON codec
   - Repository: https://github.com/rsyncOSX/DecodeEncodeGeneric
   - Purpose: Reusable JSON encoding/decoding utilities

3. **ParseRsyncOutput** - Rsync output parser
   - Repository: https://github.com/rsyncOSX/ParseRsyncOutput
   - Purpose: Extract statistics from rsync output
     
4. **RsyncArguments** - Rsync argument builder
   - Repository: https://github.com/rsyncOSX/RsyncArguments
   - Purpose: Type-safe rsync command generation



  