---
title: ScheduledTasks with Powershell
author: w3ich3rt
date: 2022-02-16 14:35:00 +0100
categories: [Linux]
tags: [linux, ssh, config, shell]
math: true
mermaid: true
image:
  path: /assets/img/windows.png
  #lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Windows Logo
---

Finally, I was able to take a look at the Core Edition of Windows 2019, where I had to build a monitoring for the pending updates on the system. Since the monitoring agent only allows a timeout of 30 seconds for scripts, I had to take the small detour via the "*scheduled tasks*". For me an odyssey in the Powershell. So here is a little HowTo for you.

## Show tasks

To view the tasks you can use the cmdlet 'Get-ScheduledTask'. Without parameters it shows a list of all tasks. If you are looking for a specific one, you can filter it via `-TaskName` and `-TaskPath`.

```powershell
Get-ScheduledTask -TaskPath "\Microsoft\Windows\WDI\"

TaskPath TaskName State
-------- -------- -----
\Microsoft\Windows\WDI\ ResolutionHost Ready
```

If you need more information `Get-ScheduledTaskInfo` may be used. Just append the `-TaskName` and you will get some info about the last and the next run.

```powershell
Get-ScheduledTaskInfo -TaskPath "\Microsoft\Windows\WDI\" -TaskName ResolutionHost

LastRunTime : 09/24/2021 08:42:42
LastTaskResult : 267009
NextRunTime :
NumberOfMissedRuns : 0
TaskName : ResolutionHost
TaskPath : \Microsoft\Windows\WDI\
PSComputerName :
```

### LastTaskResult - Convert

The value for LastTaskResult is unfortunately not yet in HEX notation and thus cannot yet be used for research without further ado. However, this can be done with the following snippet :-)

```powershell
 $("0x "+"{0:x}" -f $(Get-ScheduledTaskInfo -TaskPath "\Microsoft\Windows\WDI\" -TaskName ResolutionHost).LastTaskResult)

0x41301
```

Held against Google or this table, already very helpful:
```powershell
0x0:     The operation completed successfully.
0x1:     Incorrect function called or unknown function called.
0x2:     File not found.
0xa:     The environment is incorrect.
0x41300:     Task is ready to run at its next scheduled time.
0x41301:     Task is currently running.
0x41302:     Task is disabled.
0x41303:     Task has not yet run.
0x41304:     There are no more runs scheduled for this task.
0x41305:     One or more of the properties that are needed to run this task on a schedule have not been set.
0x41306:     Task is terminated.
0x41307:     Either the task has no triggers or the existing triggers are disabled or not set.
0x41308:     Event triggers do not have set run times.
0x41309:     A tasks trigger is not found.
0x8004130A:     One or more of the properties required to run this task have not been set.
0x8004130B:     There is no running instance of the task.
0x8004130C:     The Task Scheduler service is not installed on this computer.
0x8004130E:     The task object could not be opened.
0x8004130F:     Credentials became corrupted (*)'}
0x8004131F:     An instance of this task is already running.
0x80070002:     Basically something like file not available (2147942402)'}
0x800704DD:     The service is not available (is Run only when an user is logged on checked?)'}
0xC000013A:     The application terminated as a result of a CTRL+C.
0xC06D007E:     Unknown software exception'}
0x80041310:     Unable to establish existence of the account specified.
0x80041311:     Corruption was detected in the Task Scheduler security database; the database has been reset.
0x80041312:     Task Scheduler security services are available only on Windows NT.
0x80041313:     The task object version is either unsupported or invalid.
0x80041314:     The task has been configured with an unsupported combination of account settings and run time options.
0x80041315:     The Task Scheduler Service is not running.
0x80041316:     The task XML contains an unexpected node.
0x80041317:     The task XML contains an element or attribute from an unexpected namespace.
0x80041318:     The task XML contains a value which is incorrectly formatted or out of range.
0x80041319:     The task XML is missing a required element or attribute.
0x8004131A:     The task XML is malformed.
0x0004131B:     The task is registered, but not all specified triggers will start the task.
0x0004131C:     The task is registered, but may fail to start. Batch logon privilege needs to be enabled for the task principal.
0x8004131D:     The task XML contains too many nodes of the same type.
0x8004131E:     The task cannot be started after the trigger end boundary.
0x8004131F:     An instance of this task is already running.
0x80041320:     The task will not run because the user is not logged on.
0x80041321:     The task image is corrupt or has been tampered with.
0x80041322:     The Task Scheduler service is not available.
0x80041323:     The Task Scheduler service is too busy to handle your request. Please try again later.
0x80041324:     The Task Scheduler service attempted to run the task, but the task did not run due to one of the constraints in the task definition.
0x00041325:     The Task Scheduler service has asked the task to run.
0x80041326:     The task is disabled.
0x80041327:     The task has properties that are not compatible with earlier versions of Windows.
0x80041328:     The task settings do not allow the task to start on demand.
0xC000013A:     The application terminated as a result of a CTRL+C.
```

## Task Create
This is where the real fun starts :-). Since the objects need to be nested a bit, it helps to work with variables:

```powershell
# The action to be executed.
$action = New-ScheduledTaskAction -Execute "Script"
# What triggers the job.
#RepetitionInterval provides for regular repetition (here every three hours)
$trigger = New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Hours 3)
# Create the task "MyTask" - A -TaskPath can also be specified here.
Register-ScheduledTask -Action $action -Trigger $trigger -TaskName "MyTask" -Description "I am a description."
```

## Run task
Via `Start-ScheduledTask` you can directly start a task once.

## Delete task
Delete a task again is relatively simple. For this you only have to execute an Unregister-ScheduledTask for this task.

```powershell
Unregister-ScheduledTask -TaskName "JobName"
```
After that, you will be asked if you really want this.

# Conclusion
It's a littlebit confusing, but quite affordable :) 
