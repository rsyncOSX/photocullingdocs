+++
author = "Thomas Evensen"
title = "Swift concurrency"
date = "2025-03-01"
tags = ["swift concurrency", "asynchronous"]
categories = ["technical details"]
+++

{{% pageinfo color="info" %}}

My understanding of Swift concurrency is modest. If you want deeper coverage, I recommend exploring articles from authors who specialize in this topic.

{{% /pageinfo %}}

RsyncUI is a GUI app; most work happens on the main thread. Heavier tasks run on threads from the cooperative thread pool (CTP) without blocking the UI. The Swift runtime manages the executors and CTP. There are three kinds of executors:

- the *main executor* manages jobs on the main thread
- the *global concurrent executor* and the *serial executor* execute jobs on threads from the CTP

Most work in RsyncUI runs on the main thread. SwiftUI keeps UI updates there by default. Examples include:

- preparing and executing `rsync` synchronization tasks (including building arguments)
- monitoring progress and termination of `rsync` tasks
    - the collection of output from rsync is performed by an actor, while the actual reporting of the number of files transferred is executed on a main thread for UI updates
- some write and read operations

### Swift Version 6 and the New Concurrency Model

Swift version 6 introduced strict concurrency checking. By enabling *Swift 6 language mode* and *strict concurrency checking*, Xcode helps identify and resolve possible data races at compile time.

<div class="alert alert-danger" role="alert">

Several compiler directives in Xcode influence concurrency. Review them before migrating an existing project. I recommend the Swift concurrency posts by [Matt Massicotte](https://www.massicotte.org) and [Antoine van der Lee](https://www.avanderlee.com).

</div>

Quote from swift.org: *"More formally, a data race occurs when one thread accesses memory while the same memory is being modified by another thread. The Swift 6 language mode eliminates these issues by preventing data races at compile time."*

RsyncUI adheres to the Swift 6 concurrency model.

### Swift Concurrency and Asynchronous Execution

Swift makes asynchronous code more approachable through `async`, `await`, and `actor`. RsyncUI adopts these features even though most work can remain on the main thread—as long as it does not block the UI.

Asynchronous work can happen on the main thread or on background threads from the CTP. On the main thread, structured concurrency with `async`/`await` is key; every `await` yields control so other tasks can proceed.

### Cooperative Thread Pool (CTP)

These RsyncUI tasks run *asynchronously* on CTP threads under the actor protocol:

- read synchronization tasks from file
	- JSON data *decoding*: asynchronous decoding that inherits the actor's thread
    - JSON data *encoding*: synchronous encoding on the *main thread*
- read and sort log records
- delete log records
- prepare *output from rsync* for display
- prepare *data from the log file* for display
- check for updates to RsyncUI

Adhering to the actor protocol, all access to actor properties must be asynchronous. RsyncUI has five actors, plus additional async functions; some also run on the main thread.

#### Structured Concurrency

Some functions use `async let` for structured concurrency. Multiple `async let` bindings run concurrently, and execution resumes once they all complete.

Structured concurrency also shapes ordering. Each `await` suspends until that async work finishes; sequential awaits run one after another.

#### Example of structured concurrency

About 800 of 1500 log records are selected for deletion. Two operations run concurrently on a background thread: deleting selected logs and updating the remaining records for display.

Deletion starts on the main thread. The user selects logs and taps Delete to begin or cancel. The asynchronous `deletelogs()` function is launched on the main thread.

```
Button("Delete", role: .destructive) {
      	Task {
              await deletelogs(selectedloguuids)
            }
  }
```
`deletelogs` starts on the main thread and continues on a background thread inside `Task {}`.

```
func deletelogs(_ uuids: Set<UUID>) async {
        Task {
        
            print("(1) start async let updatedRecords deletelogs")
            
            async let updatedRecords: [LogRecords]? = ActorReadLogRecordsJSON().deletelogs(
                uuids,
                logrecords: logrecords,
                profile: rsyncUIdata.profile,
                validhiddenIDs: validhiddenIDs
            )
            let records = await updatedRecords
            
            print("(2) awaited updatedRecords deletelogs from (1))")
            print("(3) start async let updatedRecords updatelogsbyhiddenID)")
            
            async let updatedLogs: [Log]? = ActorReadLogRecordsJSON().updatelogsbyhiddenID(records, hiddenID)
            logrecords = records
            logs = await (updatedLogs ?? [])
            
            print("(4) awaited updatedLogs from (3)")

            WriteLogRecordsJSON(rsyncUIdata.profile, records)
            selectedloguuids.removeAll()
        }
    }
```
The debug windows in Xcode display the following:

The actors also print whether they execute on the main thread. The first `async let` statement initiates execution, and the subsequent `await` statement for the result above (2) suspends the function's execution until the asynchronous result is computed. The `await` statement is crucial for suspending the execution of the function until the asynchronous result is available. And then the next (3) and (4).

```
(1) start async let updatedRecords deletelogs
ActorReadLogRecordsJSON: deletelogs() NOT on main thread, currently on <NSThread: 0xa49e3c200>{number = 18}
ActorReadLogRecordsJSON: DEINIT
(2) awaited updatedRecords deletelogs from (1))
(3) start async let updatedRecords updatelogsbyhiddenID)
ActorReadLogRecordsJSON: updatelogsbyhiddenID() NOT on main thread, currently on <NSThread: 0xa49e3c280>{number = 17}
ActorReadLogRecordsJSON: DEINIT
(4) awaited updatedLogs from (3)
WriteLogRecordsJSON: writeJSONToPersistentStore file:///Users/thomas/.rsyncosx/VPxxxxxxxx/WDBackup/logrecords.json
WriteLogRecordsJSON DEINIT
ActorReadLogRecordsJSON: updatelogsbyhiddenID() NOT on main thread, currently on <NSThread: 0xa4bb72800>{number = 19}
ActorReadLogRecordsJSON: DEINIT
```

