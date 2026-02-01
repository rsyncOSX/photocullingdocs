+++
author = "Thomas Evensen"
title = "Number of files"
date = "2026-01-15"
tags = ["number of files"]
categories = ["technical details"]
+++

Numbers updated: January 15, 2026 (version 2.9.0).

RsyncUI depends only on the standard Swift and SwiftUI toolchain—no external libraries.

```
cloc DecodeEncodeGeneric/Sources ParseRsyncOutput/Sources RsyncArguments/Sources RsyncUI/RsyncUI RsyncUIDeepLinks/Sources SSHCreateKey/Sources RsyncProcessStreaming/Sources ProcessCommand/Sources RsyncAnalyse/Sources

     210 text files.
     209 unique files.                                          
      12 files ignored.

github.com/AlDanial/cloc v 2.06  T=0.07 s (3067.2 files/s, 358028.7 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Swift                          203           2824           2713          18434
C                                2             36             72            254
XML                              2              0              0             53
JSON                             1              0              0              6
C/C++ Header                     1              1              3              0
-------------------------------------------------------------------------------
SUM:                           209           2861           2788          18747
-------------------------------------------------------------------------------
```

#### Main Repository

- RsyncUI (https://github.com/rsyncOSX/RsyncUI) - the main repository for RsyncUI

##### Swift Packages used by RsyncUI

All packages track the `main` branch and are updated to latest revisions as of v2.9.0:

1. **SSHCreateKey** - SSH key generation and management
   - Repository: https://github.com/rsyncOSX/SSHCreateKey
   - Purpose: Public/private SSH key pair creation

2. **DecodeEncodeGeneric** - Generic JSON codec
   - Repository: https://github.com/rsyncOSX/DecodeEncodeGeneric
   - Purpose: Reusable JSON encoding/decoding utilities

3. **ParseRsyncOutput** - Rsync output parser
   - Repository: https://github.com/rsyncOSX/ParseRsyncOutput
   - Purpose: Extract statistics from rsync output

4. **RsyncUIDeepLinks** - Deep linking support
   - Repository: https://github.com/rsyncOSX/RsyncUIDeepLinks
   - Purpose: URL scheme handling for widgets and automation

5. **ProcessCommand** - Process execution wrapper
   - Repository: https://github.com/rsyncOSX/ProcessCommand
   - Purpose: Command-line process management

6. **RsyncArguments** - Rsync argument builder
   - Repository: https://github.com/rsyncOSX/RsyncArguments
   - Purpose: Type-safe rsync command generation

7. **RsyncProcessStreaming** - Streaming process handler
   - Repository: https://github.com/rsyncOSX/RsyncProcessStreaming
   - Purpose: Real-time rsync output streaming and progress tracking

8. **RsyncAnalyse** - Enhanced rsync output analysis
   - Repository: https://github.com/rsyncOSX/RsyncAnalyse
   - Purpose: Advanced parsing and analysis of rsync command output
  