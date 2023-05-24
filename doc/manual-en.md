cron4j 2.2 manual

cron4j 2.2 manual
=================

### Index

1.  [Quickstart](#p01)
2.  [Scheduling patterns](#p02)
3.  [How to schedule, reschedule and deschedule a task](#p03)
4.  [How to schedule a system process](#p04)
5.  [How to schedule processes from a file](#p05)
6.  [Building your own task](#p06)
7.  [Building your own collector](#p07)
8.  [Building your own scheduler listener](#p08)
9.  [Executors](#p09)
10.  [Manual task launch](#p10)
11.  [Working with Time Zones](#p11)
12.  [Daemon threads](#p12)
13.  [Predictor](#p13)
14.  [Cron parser](#p14)

1\. Quickstart
--------------

The cron4j main entity is the **_scheduler_**. With a [_it.sauronsoftware.cron4j.Scheduler_](api/it/sauronsoftware/cron4j/Scheduler.html) instance you can execute _tasks_ at fixed moments, during all the year. A scheduler can execute a task once a minute, once every five minutes, Friday at 10:00, on February the 16th at 12:30 but only if it is Saturday, and so on.

The use of the cron4j scheduler is a four steps operation:

1.  Create your _Scheduler_ instance.
2.  Schedule your actions. To schedule an action you have to tell the scheduler _what_ it has to do and _when_. You can specify _what_ using a _java.lang.Runnable_ or a [_it.sauronsoftware.cron4j.Task_](api/it/sauronsoftware/cron4j/Task.html) instance, and you can specify _when_ using a scheduling pattern, which can be represented with a string or with a [_it.sauronsoftware.cron4j.SchedulingPattern_](api/it/sauronsoftware/cron4j/SchedulingPattern.html) instance.
3.  Starts the scheduler.
4.  Stops the scheduler, when you don't need it anymore.

Consider this simple example:

import it.sauronsoftware.cron4j.Scheduler;

public class Quickstart {

	public static void main(String\[\] args) {
		// Creates a Scheduler instance.
		Scheduler s = new Scheduler();
		// Schedule a once-a-minute task.
		s.schedule("\* \* \* \* \*", new Runnable() {
			public void run() {
				System.out.println("Another minute ticked away...");
			}
		});
		// Starts the scheduler.
		s.start();
		// Will run for ten minutes.
		try {
			Thread.sleep(1000L \* 60L \* 10L);
		} catch (InterruptedException e) {
			;
		}
		// Stops the scheduler.
		s.stop();
	}

}

This example runs for ten minutes. At every minute change it will print the sad (but true) message "Another minute ticked away...".

Some other key concepts:

*   You can schedule how many tasks you want.
*   You can schedule a task when you want, also after the scheduler has been started.
*   You can change the scheduling pattern of an already scheduled task, also while the scheduler is running (_reschedule_ operation).
*   You can remove a previously scheduled task, also while the scheduler is running (_deschedule_ operation).
*   You can start and stop a scheduler how many times you want.
*   You can schedule from a file.
*   You can schedule from any source you want.
*   You can supply listeners to the scheduler in order to receive events about the executed task.
*   You can control any ongoing task.
*   You can manually launch a task, without using a scheduling pattern.
*   You can change the scheduler working Time Zone.
*   You can validate your scheduling patterns before using them with the scheduler.
*   You can predict when a scheduling pattern will cause a task execution.

[Back to index](#pIndex)

2\. Scheduling patterns
-----------------------

A UNIX crontab-like pattern is a string split in five space separated parts. Each part is intended as:

1.  **Minutes sub-pattern**. During which minutes of the hour should the task been launched? The values range is from 0 to 59.
2.  **Hours sub-pattern**. During which hours of the day should the task been launched? The values range is from 0 to 23.
3.  **Days of month sub-pattern**. During which days of the month should the task been launched? The values range is from 1 to 31. The special value "L" can be used to recognize the last day of month.
4.  **Months sub-pattern**. During which months of the year should the task been launched? The values range is from 1 (January) to 12 (December), otherwise this sub-pattern allows the aliases "jan", "feb", "mar", "apr", "may", "jun", "jul", "aug", "sep", "oct", "nov" and "dec".
5.  **Days of week sub-pattern**. During which days of the week should the task been launched? The values range is from 0 (Sunday) to 6 (Saturday), otherwise this sub-pattern allows the aliases "sun", "mon", "tue", "wed", "thu", "fri" and "sat".

The star wildcard character is also admitted, indicating "every minute of the hour", "every hour of the day", "every day of the month", "every month of the year" and "every day of the week", according to the sub-pattern in which it is used.

Once the scheduler is started, a task will be launched when the five parts in its scheduling pattern will be true at the same time.

Scheduling patterns can be represented with [_it.sauronsoftware.cron4j.SchedulingPattern_](api/it/sauronsoftware/cron4j/SchedulingPattern.html) instances. Invalid scheduling patterns are cause of [_it.sauronsoftware.cron4j.InvalidPatternException_](api/it/sauronsoftware/cron4j/InvalidPatternException.html)s. The _SchedulingPattern_ class offers also a static [_validate(String)_](api/it/sauronsoftware/cron4j/SchedulingPattern.html#validate(java.lang.String)) method, that can be used to validate a string before using it as a scheduling pattern.

Some examples:

**5 \* \* \* \***  
This pattern causes a task to be launched once every hour, at the begin of the fifth minute (00:05, 01:05, 02:05 etc.).

**\* \* \* \* \***  
This pattern causes a task to be launched every minute.

**\* 12 \* \* Mon**  
This pattern causes a task to be launched every minute during the 12th hour of Monday.

**\* 12 16 \* Mon**  
This pattern causes a task to be launched every minute during the 12th hour of Monday, 16th, but only if the day is the 16th of the month.

Every sub-pattern can contain two or more comma separated values.

**59 11 \* \* 1,2,3,4,5**  
This pattern causes a task to be launched at 11:59AM on Monday, Tuesday, Wednesday, Thursday and Friday.

Values intervals are admitted and defined using the minus character.

**59 11 \* \* 1-5**  
This pattern is equivalent to the previous one.

The slash character can be used to identify step values within a range. It can be used both in the form _\*/c_ and _a-b/c_. The subpattern is matched every _c_ values of the range _0,maxvalue_ or _a-b_.

**\*/5 \* \* \* \***  
This pattern causes a task to be launched every 5 minutes (0:00, 0:05, 0:10, 0:15 and so on).

**3-18/5 \* \* \* \***  
This pattern causes a task to be launched every 5 minutes starting from the third minute of the hour, up to the 18th (0:03, 0:08, 0:13, 0:18, 1:03, 1:08 and so on).

**\*/15 9-17 \* \* \***  
This pattern causes a task to be launched every 15 minutes between the 9th and 17th hour of the day (9:00, 9:15, 9:30, 9:45 and so on... note that the last execution will be at 17:45).

All the fresh described syntax rules can be used together.

**\* 12 10-16/2 \* \***  
This pattern causes a task to be launched every minute during the 12th hour of the day, but only if the day is the 10th, the 12th, the 14th or the 16th of the month.

**\* 12 1-15,17,20-25 \* \***  
This pattern causes a task to be launched every minute during the 12th hour of the day, but the day of the month must be between the 1st and the 15th, the 20th and the 25, or at least it must be the 17th.

Finally cron4j lets you combine more scheduling patterns into one, with the pipe character:

**0 5 \* \* \*|8 10 \* \* \*|22 17 \* \* \***  
This pattern causes a task to be launched every day at 05:00, 10:08 and 17:22.

[Back to index](#pIndex)

3\. How to schedule, reschedule and deschedule a task
-----------------------------------------------------

The simplest manner to build a task is to implement the well-known _java.lang.Runnable_ interface. Once the task is ready, it can be scheduled with the [_it.sauronsoftware.cron4j.Scheduler_](api/it/sauronsoftware/cron4j/Scheduler.html).[_schedule(String, Runnable)_](api/it/sauronsoftware/cron4j/Scheduler.html#schedule(java.lang.String,%20java.lang.Runnable)) method. This method throws an [_it.sauronsoftware.cron4j.InvalidPatternException_](api/it/sauronsoftware/cron4j/InvalidPatternException.html) when the supplied string does not represent a valid scheduling pattern.

Another way to build a task is to extend the [_it.sauronsoftware.cron4j.Task_](api/it/sauronsoftware/cron4j/Task.html) abstract class, which is more powerful and let the developer access some other cron4j features. This is better discussed in the "[Building your own task](#p06)" paragraph. _Task_ instances can be scheduled with the [_schedule(String, Task)_](api/it/sauronsoftware/cron4j/Scheduler.html#schedule(java.lang.String,%20it.sauronsoftware.cron4j.Task)) and the [_schedule(SchedulingPattern, Task)_](api/it/sauronsoftware/cron4j/Scheduler.html#schedule(it.sauronsoftware.cron4j.SchedulingPattern,%20it.sauronsoftware.cron4j.Task)) methods.

Scheduling methods available in the scheduler always return an ID used to recognize and retrieve the scheduled operation. This ID can be used later to reschedule the task (changing its scheduling pattern) with the [_reschedule(String, String)_](api/it/sauronsoftware/cron4j/Scheduler.html#reschedule(java.lang.String,%20java.lang.String)) or the [_reschedule(String, SchedulingPattern)_](api/it/sauronsoftware/cron4j/Scheduler.html#reschedule(java.lang.String,%20it.sauronsoftware.cron4j.SchedulingPattern)) methods, and to deschedule the task (remove the task from the scheduler) with the [_deschedule(String)_](api/it/sauronsoftware/cron4j/Scheduler.html#deschedule(java.lang.String)) method.

The same ID can also be used to retrieve the scheduling pattern associated with a scheduled task, with the [_getSchedulingPattern(String)_](api/it/sauronsoftware/cron4j/Scheduler.html#getSchedulingPattern(java.lang.String)) method, or to retrieve the task itself, with the [_getTask(String)_](api/it/sauronsoftware/cron4j/Scheduler.html#getTask(java.lang.String)) method.

[Back to index](#pIndex)

4\. How to schedule a system process
------------------------------------

System processes can be easily scheduled using the [ProcessTask](api/it/sauronsoftware/cron4j/ProcessTask.html) class:

ProcessTask task = new ProcessTask("C:\\\\Windows\\\\System32\\\\notepad.exe");
Scheduler scheduler = new Scheduler();
scheduler.schedule("\* \* \* \* \*", task);
scheduler.start();
// ... 

Arguments for the process can be supplied by using a string array instead of a single command string:

String\[\] command = { "C:\\\\Windows\\\\System32\\\\notepad.exe", "C:\\\\File.txt" };
ProcessTask task = new ProcessTask(command);
// ...

Environment variables for the process can be supplied using a second string array, whose elements have to be in the _NAME=VALUE_ form:

String\[\] command = { "C:\\\\tomcat\\\\bin\\\\catalina.bat", "start" };
String\[\] envs = { "CATALINA\_HOME=C:\\\\tomcat", "JAVA\_HOME=C:\\\\jdks\\\\jdk5" };
ProcessTask task = new ProcessTask(command, envs);
// ...

The default working directory for the process can be changed using a third parameter in the constructor:

String\[\] command = { "C:\\\\tomcat\\\\bin\\\\catalina.bat", "start" };
String\[\] envs = { "CATALINA\_HOME=C:\\\\tomcat", "JAVA\_HOME=C:\\\\jdks\\\\jdk5" };
File directory = "C:\\\\MyDirectory";
ProcessTask task = new ProcessTask(command, envs, directory);
// ...

If you want to change the default working directory but you have not any environment variable, the _envs_ parameter of the constructor can be set to _null_:

ProcessTask task = new ProcessTask(command, null, directory);

When _envs_ is _null_ the process inherits every environment variable of the current JVM:

Environment variables and the working directory can also be set by calling the _[setEnvs(String\[\])](api/it/sauronsoftware/cron4j/ProcessTask.html#setEnvs(java.lang.String[]))_ and [_setDirectory(java.io.File)_](api/it/sauronsoftware/cron4j/ProcessTask.html#setDirectory(java.io.File)) methods.

The process standard output and standard error channels can be redirected to files by using the [_setStdoutFile(java.io.File)_](api/it/sauronsoftware/cron4j/ProcessTask.html#setStdoutFile(java.io.File)) and [_setStderrFile(java.io.File)_](api/it/sauronsoftware/cron4j/ProcessTask.html#setStderrFile(java.io.File)) methods:

ProcessTask task = new ProcessTask(command, envs, directory);
task.setStdoutFile(new File("out.txt"));
task.setStderrFile(new File("err.txt"));

In a siminal manner, the standard input channel can be read from an existing file, calling the _[setStdinFile(java.io.File)](api/it/sauronsoftware/cron4j/ProcessTask.html#setStdinFile(java.io.File))_ method:

ProcessTask task = new ProcessTask(command, envs, directory);
task.setStdinFile(new File("in.txt"));

5\. How to schedule processes from a file
-----------------------------------------

The cron4j scheduler can also schedule a set of processes from a file.

You have to prepare a file, very similar to the ones used by the UNIX crontab, and register it in the scheduler calling the [_scheduleFile(File)_](api/it/sauronsoftware/cron4j/Scheduler.html#scheduleFile(java.io.File)) method. The file can be descheduled by calling the [_descheduleFile(File)_](api/it/sauronsoftware/cron4j/Scheduler.html#descheduleFile(java.io.File)) method. Scheduled files can be retrieved by calling the [_getScheduledFiles()_](api/it/sauronsoftware/cron4j/Scheduler.html#getScheduledFiles()) method.

Scheduled files are parsed every minute. The scheduler will launch every process declared in the file whose scheduling pattern matches the current system time.

Syntax rules for cron4j scheduling files are reported in the "[Cron parser](#p13)" paragraph.

[Back to index](#pIndex)

6\. Building your own task
--------------------------

A _java.lang.Runnable_ object is the simplest task ever possible, but to gain control you need to extend the [_it.sauronsoftware.cron4j.Task_](api/it/sauronsoftware/cron4j/Task.html) class. In the plainest form, implementing _Runnable_ or extending _Task_ are very similar operations: while the first requires a _run()_ method, the latter requires the implementation of [_execute(TaskExecutionContext)_](api/it/sauronsoftware/cron4j/Task.html#execute(it.sauronsoftware.cron4j.TaskExecutionContext)). The _execute(TaskExecutionContext)_ method provides a [_it.sauronsoftware.cron4j.TaskExecutionContext_](api/it/sauronsoftware/cron4j/TaskExecutionContext.html) instance, which the _Runnable.run()_ method does not provide. The context can be used in the following ways:

*   A task can communicate with its executor, by notifying its internal state with a textual description. This is called _status tracking_. If you are interested in supporting status tracking in your task, you have to override the [_supportsStatusTracking()_](api/it/sauronsoftware/cron4j/Task.html#supportsStatusTracking()) method, which should return _true_. Once this has been done, within the _execute(TaskExecutionContext)_ method you are enabled to call the context [_setStatusMessage(String)_](api/it/sauronsoftware/cron4j/TaskExecutionContext.html#setStatusMessage(java.lang.String)) method. This will propagate your task status message to its executor. The status message, through the executor, can be retrieved by an external user (see the "[Executors](#p09)" paragraph).
    
*   A task can communicate with its executor, by notifying its completeness level with a numeric value. This is called _completeness tracking_. If you are interested in supporting completeness tracking in your task, you have to override the [_supportsCompletenessTracking()_](api/it/sauronsoftware/cron4j/Task.html#supportsCompletenessTracking()) method, which should return _true_. Once this has been done, within the _execute(TaskExecutionContext)_ method you are enabled to call the context [_setCompleteness(double)_](api/it/sauronsoftware/cron4j/TaskExecutionContext.html#setCompleteness(double)) method, with a value between 0 and 1. This will propagate your task completeness level to its executor. The completeness level, through the executor, can be retrieved by an external user (see the "[Executors](#p09)" paragraph).
    
*   A task can be optionally paused. If you are interested in supporting pausing and resuming in your task, you have to override the [_canBePaused()_](api/it/sauronsoftware/cron4j/Task.html#canBePaused()) method, which should return _true_. Once this has been done, within the _execute(TaskExecutionContext)_ method you have to periodically call the context [_pauseIfRequested()_](api/it/sauronsoftware/cron4j/TaskExecutionContext.html#pauseIfRequested()) method. This will pause the task execution until it will be resumed (or stopped) by an external user (see the "[Executors](#p09)" paragraph).
    
*   A task can be optionally stopped. If you are interested in supporting stopping in your task, you have to override the [_canBeStopped()_](api/it/sauronsoftware/cron4j/Task.html#canBeStopped()) method, which should return _true_. Once this has been done, within the _execute(TaskExecutionContext)_ method you have to periodically call the context [_isStopped()_](api/it/sauronsoftware/cron4j/TaskExecutionContext.html#isStopped()) method. This will return _true_ when the execution has be demanded to be stopped by an external user (see the "[Executors](#p09)" paragraph). Then it's your responsibility to handle the event, by letting your task gently stop its ongoing activities.
    
*   Through the context, the task can retrieve the scheduler, calling [_getScheduler()_](api/it/sauronsoftware/cron4j/TaskExecutionContext.html#getScheduler()).
    
*   Through the context, the task can retrieve its executor, calling [_getTaskExecutor()_](api/it/sauronsoftware/cron4j/TaskExecutionContext.html#getTaskExecutor()).
    

A custom task can be [scheduled](#p03), [launched immediately](#p10) or returned by a [task collector](#p07).

[Back to index](#pIndex)

7\. Building your own collector
-------------------------------

You can build and plug within the scheduler your own task source, via the _task collector_ API.

The cron4j scheduler supports the registration of one or more [_it.sauronsoftware.cron4j.TaskCollector_](api/it/sauronsoftware/cron4j/TaskCollector.html) instances, with the [_addTaskCollector(TaskCollector)_](api/it/sauronsoftware/cron4j/Scheduler.html#addTaskCollector(it.sauronsoftware.cron4j.TaskCollector)) method. Registered collectors can be retrieved with the scheduler [_getTaskCollectors()_](api/it/sauronsoftware/cron4j/Scheduler.html#getTaskCollectors()) method. A previously registered collector can be removed from the scheduler with the [_removeTaskCollector(TaskCollector)_](api/it/sauronsoftware/cron4j/Scheduler.html#removeTaskCollector(it.sauronsoftware.cron4j.TaskCollector)) method. Collectors can be added, queried or removed at every moment, also when the scheduler is started and it is running.

Each registered task collector is queried by the scheduler once a minute. The scheduler calls the collector [_getTasks()_](api/it/sauronsoftware/cron4j/TaskCollector.html#getTasks()) method. The implementation must return a [_it.sauronsoftware.cron4j.TaskTable_](api/it/sauronsoftware/cron4j/TaskTable.html) instance. A _TaskTable_ is a table that associates tasks and scheduling patterns. The scheduler, once the table has been retrieved, will examine the reported entries, and it will execute every task whose scheduling pattern matches the current system time.

A custom collector can be used to tie the scheduler with an external task source, i.e. a database or a XML file, which can be managed and changed in its contents also at run time.

[Back to index](#pIndex)

8\. Building your own scheduler listener
----------------------------------------

The [_it.sauronsoftware.cron4j.SchedulerListener_](api/it/sauronsoftware/cron4j/SchedulerListener.html) API can be used to listen to scheduler events.

The _SchedulerListener_ interface requires the implementation of the following methods:

*   [_taskLaunching(TaskExecutor)_](api/it/sauronsoftware/cron4j/SchedulerListener.html#taskLaunching(it.sauronsoftware.cron4j.TaskExecutor))  
    This one is called every time a task is launched by the scheduler.
    
*   [_taskSucceeded(TaskExecutor)_](api/it/sauronsoftware/cron4j/SchedulerListener.html#taskSucceeded(it.sauronsoftware.cron4j.TaskExecutor))  
    This one is called every time a task execution has been successfully completed.
    
*   [_taskFailed(TaskExecutor, Throwable)_](api/it/sauronsoftware/cron4j/SchedulerListener.html#taskFailed(it.sauronsoftware.cron4j.TaskExecutor,%20java.lang.Throwable))  
    This one is called every time a task execution fails due to an uncaught exception.
    

See the "[Executors](#p09)" paragraph for more info about task executors.

Once your _SchedulerListener_ instance is ready, you can register it on a _Scheduler_ object by calling its [_addSchedulerListener(SchedulerListener)_](api/it/sauronsoftware/cron4j/Scheduler.html#addSchedulerListener(it.sauronsoftware.cron4j.SchedulerListener)) method. Already registered listeners can be removed by calling the [_removeSchedulerListener(SchedulerListener)_](api/it/sauronsoftware/cron4j/Scheduler.html#removeSchedulerListener(it.sauronsoftware.cron4j.SchedulerListener)) method. The scheduler can also give back any registered listener, with the [_getSchedulerListeners()_](api/it/sauronsoftware/cron4j/Scheduler.html#getSchedulerListeners()) method.

_SchedulerListener_s can be added and removed at every moment, also while the scheduler is running.

[Back to index](#pIndex)

9\. Executors
-------------

The scheduler, once it has been started and it is running, can be queried to give back its _executors_. An executor is similar to a thread. Executors is used by the scheduler to execute tasks.

By calling the [_Scheduler_](api/it/sauronsoftware/cron4j/Scheduler.html).[_getExecutingTasks()_](api/it/sauronsoftware/cron4j/Scheduler.html#getExecutingTasks()) method you can obtain the currently ongoing executors.

You can also obtain an executor through a [_SchedulerListener_](api/it/sauronsoftware/cron4j/SchedulerListener.html) instance (see the "[Building your own scheduler listener](#p08)" paragraph).

Each executor, represented by a [_it.sauronsoftware.cron4j.TaskExecutor_](api/it/sauronsoftware/cron4j/TaskExecutor.html) instance, performs a different task execution.

The task can be retrieved with the [_getTask()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#getTask()) method.

The executor status can be checked with the [_isAlive()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#isAlive()) method: it returns _true_ if the executor is currently running.

If the executor is running, the current thread can be paused until the execution will be completed, calling the [_join()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#join()) method.

The [_supportsStatusTracking()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#supportsStatusTracking()) method returns _true_ if the currently executing task supports _status tracking_. It means that the task communicates to the executor its status, represented by a string. The current status message can be retrieved by calling the executor [_getStatusMessage()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#getStatusMessage()) method.

The [_supportsCompletenessTracking()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#supportsCompletenessTracking()) method returns _true_ if the currently executing task supports _completeness tracking_. It means that the task communicates to the executor its own completeness level. The current completeness level can be retrieved by calling the executor [_getCompleteness()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#getCompleteness()) method. Returned values are between 0 (task just started and still nothing done) and 1 (task completed).

The [_canBePaused()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#canBePaused()) method returns _true_ if the currently executing task supports _execution pausing_. It means that the task execution can be paused by calling the executor [_pause()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#pause()) method. The pause status of the executor can be checked with the [_isPaused()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#isPaused()) method. A paused executor can be resumed by calling its [_resume()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#resume()) method.

The [_canBeStopped()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#canBeStopped()) method returns _true_ if the currently executing task supports _execution interruption_. It means that the task execution can be stopped by calling the executor [_stop()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#stop()) method. The interruption status of the executor can be checked with the [_isStopped()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#isStopped()) method. Stopped executors cannot be resumed.

The [_getStartTime()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#getStartTime()) method returns a time stamp reporting the start time of the executor, or a value less than 0 if the executor has not been yet started.

The [_getScheduler()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#getScheduler()) method returns the scheduler which is the owner of the executor.

The [_getGuid()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#getGuid()) method returns a textual GUID for the executor.

Executors offer also an event-driven API, through the [_it.sauronsoftware.cron4j.TaskExecutorListener_](api/it/sauronsoftware/cron4j/TaskExecutorListener.html) class. A _TaskExecutorListener_ can be added to a _TaskExecutor_ with its [_addTaskExecutorListener(TaskExecutorListener)_](api/it/sauronsoftware/cron4j/TaskExecutor.html#addTaskExecutorListener(it.sauronsoftware.cron4j.TaskExecutorListener)) method. Listeners can be removed with the [_removeTaskExecutorListener(TaskExecutorListener)_](api/it/sauronsoftware/cron4j/TaskExecutor.html#removeTaskExecutorListener(it.sauronsoftware.cron4j.TaskExecutorListener)) method, and they can also be retrieved with the [_getTaskExecutorListeners()_](api/it/sauronsoftware/cron4j/TaskExecutor.html#getTaskExecutorListeners()) method. A _TaskExecutorListener_ must implement the following methods:

*   [_executionPausing(TaskExecutor)_](api/it/sauronsoftware/cron4j/TaskExecutor.html#executionPausing(it.sauronsoftware.cron4j.TaskExecutor))  
    Called when the executor is requested to pause the ongoing task. The given parameter represents the source _TaskExecutor_ instance.
    
*   [_executionResuming(TaskExecutor)_](api/it/sauronsoftware/cron4j/TaskExecutor.html#executionResuming(it.sauronsoftware.cron4j.TaskExecutor))  
    Called when the executor is requested to resume the execution of the previously paused task. The given parameter represents the source _TaskExecutor_ instance.
    
*   [_executionStopping(TaskExecutor)_](api/it/sauronsoftware/cron4j/TaskExecutor.html#executionStopping(it.sauronsoftware.cron4j.TaskExecutor))  
    Called when the executor is requested to stop the task execution. The given parameter represents the source _TaskExecutor_ instance.
    
*   [_executionTerminated(TaskExecutor, Throwable)_](api/it/sauronsoftware/cron4j/TaskExecutor.html#executionTerminated(it.sauronsoftware.cron4j.TaskExecutor,%20java.lang.Throwable))  
    Called when the executor has completed the task execution. The first parameter represents the source _TaskExecutor_ instance, while the second is the optional exception that has caused the task to be terminated. If the task has been completed successfully, the given value is _null_.
    
*   [_statusMessageChanged(TaskExecutor, String)_](api/it/sauronsoftware/cron4j/TaskExecutor.html#statusMessageChanged(it.sauronsoftware.cron4j.TaskExecutor,%20java.lang.String))  
    Called every time the execution status message changes. The first parameter represents the source _TaskExecutor_ instance, while the second is the new message issued by the task.
    
*   [_completenessValueChanged(TaskExecutor, double)_](api/it/sauronsoftware/cron4j/TaskExecutor.html#completenessValueChanged(it.sauronsoftware.cron4j.TaskExecutor,%20double))  
    Called every time the execution completeness value changes. The first parameter represents the source _TaskExecutor_ instance, while the second is the new completeness value (between 0 and 1) issued by the task.
    

[Back to index](#pIndex)

10\. Manual task launch
-----------------------

If the scheduler is started and running, it is possible to manually launch a task, without scheduling it with a pattern. The method is [_Scheduler_](api/it/sauronsoftware/cron4j/Scheduler.html).[_launch(Task)_](api/it/sauronsoftware/cron4j/Scheduler.html#launch(it.sauronsoftware.cron4j.Task)). The task will be immediately launched, and a [_TaskExecutor_](api/it/sauronsoftware/cron4j/TaskExecutor.html) instace is returned to the caller. The returned object can be used to control the task execution (see the "[Executors](#p09)" paragraph).

[Back to index](#pIndex)

11\. Working with Time Zones
----------------------------

Scheduler instances, by default, work with the system default Time Zone. I.e. a scheduling pattern whose value is _0 2 \* \* \*_ will activate its task at 02:00 AM according to the default system Time Zone. The scheduler can be requested to work with a different Time Zone, which is not the system default one. Call [_Scheduler_](api/it/sauronsoftware/cron4j/Scheduler.html).[_setTimeZone(TimeZone)_](api/it/sauronsoftware/cron4j/Scheduler.html#setTimeZone(java.util.TimeZone)) and [_Scheduler_](api/it/sauronsoftware/cron4j/Scheduler.html).[_getTimeZone()_](api/it/sauronsoftware/cron4j/Scheduler.html#getTimeZone()) to control this feature.

Once the default Time Zone has been changed, system current time is adapted to the supplied zone before comparing it with registered scheduling patterns. The result is that any supplied scheduling pattern is treated according to the specified Time Zone. Suppose this situation:

*   System time: 10:00
*   System time zone: GMT+1
*   Scheduler time zone: GMT+3

The scheduler, before comparing system time with patterns, translates 10:00 from GMT+1 to GMT+3. It means that 10:00 becomes 12:00. The resulted time is then used by the scheduler to activate tasks. So, in the given configuration at the given moment, any task scheduled as _0 12 \* \* \*_ will be executed, while any _0 10 \* \* \*_ will not.

[Back to index](#pIndex)

12\. Daemon threads
-------------------

The Java Virtual Machine exits when the only threads running are all daemon threads. The cron4j scheduler can be configured to spawn only daemon threads. To control this feature call the [_Scheduler_](api/it/sauronsoftware/cron4j/Scheduler.html).[_setDaemon(boolean)_](api/it/sauronsoftware/cron4j/Scheduler.html#setDaemon(boolean)) method. This method must be called before the scheduler is started. Default value is _false_. To check the scheduler current daemon status call the [_Scheduler_](api/it/sauronsoftware/cron4j/Scheduler.html).[_isDaemon()_](api/it/sauronsoftware/cron4j/Scheduler.html#isDaemon()) method.

[Back to index](#pIndex)

13\. Predictor
--------------

The [_it.sauronsoftware.cron4j.Predictor_](api/it/sauronsoftware/cron4j/Predictor.html) class is able to predict when a scheduling pattern will be matched.

Suppose you want to know when the scheduler will execute a task scheduled with the pattern _0 3 \* jan-jun,sep-dec mon-fri_. You can predict the next _n_ execution of the task using a Predictor instance:

String pattern = "0 3 \* jan-jun,sep-dec mon-fri";
Predictor p = new Predictor(pattern);
for (int i = 0; i < n; i++) {
	System.out.println(p.nextMatchingDate());
}

[Back to index](#pIndex)

14\. Cron parser
----------------

The [_it.sauronsoftware.cron4j.CronParser_](api/it/sauronsoftware/cron4j/CronParser.html) class can be used to parse crontab-like formatted file and character streams.

If you want to schedule a list of tasks declared in a crontab-like file you don't need the _CronParser_, since you can do it by adding the file to the scheduler, with the [_Scheduler_](api/it/sauronsoftware/cron4j/Scheduler.html).[_scheduleFile(File)_](api/it/sauronsoftware/cron4j/Scheduler.html#scheduleFile(File)) method.

Consider to use the _CronParser_ if the _Scheduler.scheduleFile(File)_ method is not enough for you. In example, you may need to fetch the task list from a remote source which is not representable as a _java.io.File_ object (a document on a remote server, a DBMS result set and so on). To solve the problem you can implement your own [_it.sauronsoftware.cron4j.TaskCollector_](api/it/sauronsoftware/cron4j/TaskCollector.html), getting the advantage of the _CronParser_ to easily parse any crontab-like content.

You can parse a whole file/stream, but you can also parse a single line.

A line can be empty, can contain a comment or it can be a scheduling line.

A line containing no characters or a line with only space characters is considered an empty line.

A line whose first non-space character is a number sign (#) is considered a comment.

Empty lines and comment lines are ignored by the parser.

Any other kind of line is parsed as a scheduling line.

A valid scheduling line respects the following structure:

scheduling-pattern \[options\] command \[args\]

*   _scheduling-pattern_ is a valid scheduling pattern, according with the definition given by the [_it.sauronsoftware.cron4j.SchedulingPattern_](api/it/sauronsoftware/cron4j/SchedulingPattern.html) class.
*   _options_ is a list of optional information used by cron4j to prepare the task execution environment. See below for a more detailed description.
*   _command_ is a system valid command, such an executable call.
*   _args_ is a list of optional arguments for the command.

After the scheduling pattern item, other tokens in each line are space separated or delimited with double quotation marks (").

Double quotation marks delimited items can take advantage of the following escape sequences:

*   \\" - quotation mark
*   \\\\ - back slash
*   \\/ - slash
*   \\b - back space
*   \\f - form feed
*   \\n - new line
*   \\r - carriage return
*   \\t - horizontal tab
*   \\u_four-hex-digits_ - the character at the given Unicode index

The _options_ token collection can include one or more of the following elements:

*   IN:_file-path_ - Redirects the command standard input channel to the specified file.
*   OUT:_file-path_ - Redirects the command standard output channel to the specified file.
*   ERR:_file-path_ - Redirects the command standard error channel to the specified file.
*   ENV:_name_\=_value_ - Defines an environment variable in the scope of the command.
*   DIR:_directory-path_ - Sets the path of the working directory for the command. This feature is not supported if the executing JVM is less than 1.3.

It is also possible to schedule the invocation of a method of a Java class in the scope of the parser ClassLoader. The method has to be static and it must accept an array of strings as its sole argument. To invoke a method of this kind the syntax is:

scheduling-pattern java:className#methodName \[args\]

The _#methodName_ part can be omitted: in this case the _main(String\[\])_ method will be assumed.

Please note that static methods are invoked within the scheduler same JVM, without spawning any external process. Thus IN, OUT, ERR, ENV and DIR options can't be applied.

Invalid scheduling lines are discarded without blocking the parsing procedure, but an error message is sent to the application standard error channel.

Valid examples:

0 5 \* \* \* sol.exe
0,30 \* \* \* \* OUT:C:\\ping.txt ping 10.9.43.55
0,30 4 \* \* \* "OUT:C:\\Documents and Settings\\Carlo\\ping.txt" ping 10.9.43.55
0 3 \* \* \* ENV:JAVA\_HOME=C:\\jdks\\1.4.2\_15 DIR:C:\\myproject OUT:C:\\myproject\\build.log C:\\myproject\\build.bat "Nightly Build"
0 4 \* \* \* java:mypackage.MyClass#startApplication myOption1 myOption2

[Back to index](#pIndex)
