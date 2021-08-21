# bug

```json
OKOK{
  "status": "ERROR",
  "result": {
    "message": "Ask timed out on [Actor[akka://SnappyLeadJobServer/user/context-supervisor/snappyContext1618979701885345806#1894701446]] after [10000 ms]",
    "errorClass": "akka.pattern.AskTimeoutException",
    "stack": ["akka.pattern.PromiseActorRef$$anonfun$1.apply$mcV$sp(AskSupport.scala:334)", 
              "akka.actor.Scheduler$$anon$7.run(Scheduler.scala:117)", 
              "scala.concurrent.Future$InternalCallbackExecutor$.unbatchedExecute(Future.scala:601)", 
              "scala.concurrent.BatchingExecutor$class.execute(BatchingExecutor.scala:109)", 
              "scala.concurrent.Future$InternalCallbackExecutor$.execute(Future.scala:599)", 
              "akka.actor.LightArrayRevolverScheduler$TaskHolder.executeTask(Scheduler.scala:474)", 
              "akka.actor.LightArrayRevolverScheduler$$anon$8.executeBucket$1(Scheduler.scala:425)", 
              "akka.actor.LightArrayRevolverScheduler$$anon$8.nextTick(Scheduler.scala:429)", 
              "akka.actor.LightArrayRevolverScheduler$$anon$8.run(Scheduler.scala:381)", 
              "java.lang.Thread.run(Thread.java:748)"]
  }
}
```

