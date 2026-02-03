+++
author = "Thomas Evensen"
title = "Version 0.6.0"
date = "2026-02-03"
tags = ["changelog","version 0.6.0"]
categories = ["changelog"]
+++

### Version 0.6.0 - Feb 3, 2026 (beta release)

Development Update:

The navigation within the application is undergoing frequent updates. Although not yet released, it is anticipated to be available within the next few days. The Zoomable view, which opens when a row is selected and Enter is pressed, will now remain in the background and automatically update when traversing the file table.

This enhancement will facilitate easier viewing of photos and key information, tagging of photos, and automatic updates of the Zoomable view when a closer examination is required.


<div class="alert alert-secondary" role="alert">

The previous issue has been resolved. The problem was related to the Sandbox entitlement of the application. The application is signed and notarized by Apple and is fully Sandboxed. It utilizes the default `/usr/bin/rsync` when copying files from source to destination. For further information, please refer to the user documentation. The application is non-destructive and only copies files, without any deletion.

Please verify the SHA 256 if you decide to try it out.

</div>
