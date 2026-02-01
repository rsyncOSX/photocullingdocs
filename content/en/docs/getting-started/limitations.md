+++
author = "Thomas Evensen"
title = "Limitations"
date = "2025-03-22"
tags = ["limitations"]
categories = ["general information"]
+++

There are a few limitations to keep in mind. None will corrupt data, but they may interrupt synchronization.

### Long running tasks and sleep

When the Mac sleeps, RsyncUI tasks stop. An active rsync process does not prevent the Mac from sleeping. If sleep occurs during synchronization, rsync will likely error and stop. Restart the task in RsyncUI; rsync will resume from where it left off.

If this happens, RsyncUI will not log the run because it only logs after receiving a termination signal from rsync.

<div class="alert alert-secondary" role="alert">

Adjust your Mac's sleep settings to avoid interruptions.

</div>

The above will also be true if you are using the Calendar function.

### The Schedule

A note about scheduling: the scheduler relies on the Timer library. See *Schedule* for details.

The scheduler automates chosen tasks while RsyncUI is running and you are logged in. If the Mac sleeps and a scheduled task is missed, it will be listed in the Schedule view after the Mac wakes.

### Aborting Tasks

RsyncUI launches the `rsync` command-line tool and monitors it.

You can abort a task at any time. Allow the abort to complete before starting another task; otherwise RsyncUI may become unresponsive.