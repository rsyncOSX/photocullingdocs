+++
author = "Thomas Evensen"
title = "Getting started"
date = "2025-02-12"
tags = ["getting started"]
categories = ["general information"]
+++

New users should read *Important* first, then see *Latest version of rsync* for installation guidance.

### Concealing Actions

Concealing actions reduces menu clutter and keeps focus on common tasks.

Use `⌘S` or the Task main menu to show or hide Charts, Quick task, log view, and other options that are hidden by default. The primary Sidebar menu is also context-sensitive.

### New Tasks and Verification

RsyncUI can synchronize data to:

- local attached disk
- remote servers on the Internet or local LAN
    - require *passwordless login* by ssh-key

#### Local attached disk

To sync to a local attached disk, connect the disk, add *source* and *destination*, and you are ready for your first task.

#### Remote server and passwordless login

To sync to a remote server (internet or LAN), set up passwordless SSH. With SSH keys in place, add *source*, *destination*, *login id*, and *server name* and you can synchronize. Without passwordless login, additional setup is required.

#### Verify tasks

<div class="alert alert-danger" role="alert">

*Always* verify a new or changed task. After adding or editing a task, or after changing rsync parameters, select *"Verify tasks"* from the primary Sidebar menu. 

</div>

Select the task and press the *play* button on the toolbar. Executing the *play* button includes the --dry-run parameter for rsync, which is an estimation run.

{{< figure src="/images/verifyatask/verifytask.png" alt="" position="center" style="border-radius: 8px;" >}}

#### Aborting tasks

RsyncUI executes the command-line tool `rsync` and monitors progress and termination.

You can abort a task at any time. Allow the abort to complete and cleanup to finish before starting another task; otherwise the app may become unresponsive.

### Errors in tagging data

Sometimes you may get the error:
```
There are errors in tagging data
for synchronize ID XXXX
Most likely number of rows
> 20 lines and no data to synchronize
```

The message confirms that all data appears synchronized. RsyncUI bases this on rsync summary output. If there are more than ~20 lines yet no data tagged for sync, the message reminds you to double-check. To verify, run the task without an estimate.

Version 3.x of rsync:

{{< figure src="/images/gettingstarted/linesver3x.png" alt="" position="center" style="border-radius: 8px;" >}}

Version openrsync:

{{< figure src="/images/gettingstarted/linesopenrsync.png" alt="" position="center" style="border-radius: 8px;" >}}

#### The Sidebar Menu options

Two Sidebar menu options depend on task properties. If any task meets the requirement, the option appears.

- *Snapshot*: available only for *snapshot* tasks
	- requires rsync 3.x
- *Restore*: available for *synchronize* and *snapshot* tasks where the *destination* is on a *remote server*
	- also available for openrsync, but only for synchronize tasks

<div class="alert alert-secondary" role="alert">

If the Sidebar is hidden, click the Hide Sidebar icon (top left) or toggle *Hide the Sidebar on startup* in Settings.

</div>
