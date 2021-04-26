# async method :

 - compilation generates custom  `AsyncStateMachine` that implements interface `IAsyncStateMachine`, the arguments of the method are the properties of the  `AsyncStateMachine`

 - create the instance of `AsyncStateMachine`

   `stateMachine.<>t__builder = AsyncTaskMethodBuilder.Create();`

 - call builder start method

   `stateMachine.<>t__builder.Start(ref stateMachine);`

- return the task which is the property of the builder

  `return stateMachine.<>t__builder.Task;`

# method process:

builder.Start 

AsyncMethodBuilderCore.Start(ref stateMachine); 

- save `ExecutionContext` of the current thread
- save `SynchronizationContext` of the current thread
- call the `MoveNext`  which in the state machine
- restore the current thread with `ExecutionContext` and  `SynchronizationContext` which    previously saved.

# MoveNext() process

- synchronously execute some segments which the placement is in front of awaitable segment.
- get the awaiter from the awaiting method
- check if the async method is completed
  - false, call `<>t__builder.AwaitUnsafeOnCompleted(ref awaiter, ref stateMachine);`
  - return
- execute the last segments which the placement is behind the awaiting method.
- return

# AwaitUnsafeOnCompleted()

- call `AsyncTaskMethodBuilder<VoidTaskResult>.AwaitUnsafeOnCompleted`

  - box state machine to `AsyncStateMachineBox<>`  `IAsyncStateMachineBox` 

    - set the property `(ExecutionContext)Context` from the current thread
    - set the property `(IAsyncStateMachine)StateMachine` from the argument

  - determine the awaiter type. etc `ITaskAwaiter\IConfiguredTaskAwaiter\IStateMachineBoxAwareAwaiter`

    - `ITaskAwaiter` `TaskAwaiter`

      `TaskAwaiter.UnsafeOnCompletedInternal`  `continueOnCapturedContext=true`

    - `IConfiguredTaskAwaiter`  `ConfiguredTaskAwaiter`

      `TaskAwaiter.UnsafeOnCompletedInternal` `continueOnCapturedContext=ConfiguredTaskAwaiter.continueOnCapturedContext`

    - `IStateMachineBoxAwareAwaiter`

      `IStateMachineBoxAwareAwaiter.AwaitUnsafeOnCompleted`

    - default `ICriticalNotifyCompletion`

      `awaiter.UnsafeOnCompleted`

# TaskAwaiter.UnsafeOnCompletedInternal

- call `task.UnsafeSetContinuationForAwait`
  - determine `continueOnCapturedContext`
    - true
      - get the `SynchronizationContext` from the current thread
      - if the original type **is not**  `SynchronizationContext` 
        - generate `SynchronizationContextAwaitTaskContinuation`
        - `AddTaskContinuation` or directly call `SynchronizationContextAwaitTaskContinuation.Run`
          - SynchronizationContextAwaitTaskContinuation.SynchronizationContext.Post
        - return
      - get the `TaskScheduler` from `TaskScheduler.InternalCurrent` and the `TaskScheduler` **is not** `TaskScheduler.Default`
        - generate `TaskSchedulerAwaitTaskContinuation`
        - `AddTaskContinuation` or directly call `TaskSchedulerAwaitTaskContinuation.Run`
          - create continuation task with `TaskScheduler`
          - continuationtask.ScheduleAndStart
        - return
    - false
      - `AddTaskContinuation` or directly call `ThreadPool.UnsafeQueueUserWorkItemInternal`

# Task.FinishContinuations()

- `Task.RunContinuations(Task.m_continuationObject);`
  - `AwaitTaskContinuation.RunOrScheduleAction` or `TaskContinuation.Run` etc..

# Task.TrySetResult

- `Task.FinishContinuations();`