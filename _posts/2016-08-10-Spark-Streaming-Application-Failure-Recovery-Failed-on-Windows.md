### 1. What Happened?
I made a Application using Spark Streaming. When I tested the Fault Tolerant Feature using ```WriteAheadLog``` mechanism of Spark on **Windows**(yeah, windows), I found out that the application could not recover from failure. What makes it strange is that, the application works well until I killed the driver manually, and the driver restarted and failed. I have read the source code, and the reason is as follows:

### 2. Environment
* Windows server 2012
* Spark: 1.6.1
* Hadoop: 2.6.3
* Language: C#

### 3. Full Debug Log
```
16/08/03 12:26:14 INFO scheduler.TaskSetManager: Lost task 0.3 in stage 4.0 (TID 85) on executor cnazdev04.fareast.corp.microsoft.com: org.apache.spark.SparkException (Could not read data from write ahead log record FileBasedWriteAheadLogSegment(hdfs://CNAZDEV02:19000/checkpoint/receivedData/0/log-1470197944406-1470198004406,189,59)) [duplicate 3]
16/08/03 12:26:14 ERROR scheduler.TaskSetManager: Task 0 in stage 4.0 failed 4 times; aborting job
16/08/03 12:26:14 INFO cluster.YarnClusterScheduler: Cancelling stage 4
16/08/03 12:26:14 INFO cluster.YarnClusterScheduler: Stage 4 was cancelled
16/08/03 12:26:14 INFO scheduler.DAGScheduler: ResultStage 4 (count at NativeMethodAccessorImpl.java:-2) failed in 5.624 s
16/08/03 12:26:14 INFO scheduler.DAGScheduler: Job 3 failed: count at NativeMethodAccessorImpl.java:-2, took 5.691028 s
java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.spark.api.csharp.CSharpBackendHandler.handleMethodCall(CSharpBackendHandler.scala:156)
	at org.apache.spark.api.csharp.CSharpBackendHandler.handleBackendRequest(CSharpBackendHandler.scala:103)
	at org.apache.spark.api.csharp.CSharpBackendHandler.channelRead0(CSharpBackendHandler.scala:30)
	at org.apache.spark.api.csharp.CSharpBackendHandler.channelRead0(CSharpBackendHandler.scala:27)
	at io.netty.channel.SimpleChannelInboundHandler.channelRead(SimpleChannelInboundHandler.java:105)
	at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:308)
	at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:294)
	at io.netty.handler.codec.MessageToMessageDecoder.channelRead(MessageToMessageDecoder.java:103)
	at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:308)
	at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:294)
	at io.netty.handler.codec.ByteToMessageDecoder.channelRead(ByteToMessageDecoder.java:244)
	at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:308)
	at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:294)
	at io.netty.channel.DefaultChannelPipeline.fireChannelRead(DefaultChannelPipeline.java:846)
	at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:131)
	at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:511)
	at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:468)
	at io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:382)
	at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:354)
	at io.netty.util.concurrent.SingleThreadEventExecutor$2.run(SingleThreadEventExecutor.java:111)
	at io.netty.util.concurrent.DefaultThreadFactory$DefaultRunnableDecorator.run(DefaultThreadFactory.java:137)
	at java.lang.Thread.run(Thread.java:745)
Caused by: org.apache.spark.SparkException: Job aborted due to stage failure: Task 0 in stage 4.0 failed 4 times, most recent failure: Lost task 0.3 in stage 4.0 (TID 85, cnazdev04.fareast.corp.microsoft.com): org.apache.spark.SparkException: Could not read data from write ahead log record FileBasedWriteAheadLogSegment(hdfs://CNAZDEV02:19000/checkpoint/receivedData/0/log-1470197944406-1470198004406,189,59)
	at org.apache.spark.streaming.rdd.WriteAheadLogBackedBlockRDD.org$apache$spark$streaming$rdd$WriteAheadLogBackedBlockRDD$$getBlockFromWriteAheadLog$1(WriteAheadLogBackedBlockRDD.scala:143)
	at org.apache.spark.streaming.rdd.WriteAheadLogBackedBlockRDD$$anonfun$compute$1.apply(WriteAheadLogBackedBlockRDD.scala:167)
	at org.apache.spark.streaming.rdd.WriteAheadLogBackedBlockRDD$$anonfun$compute$1.apply(WriteAheadLogBackedBlockRDD.scala:167)
	at scala.Option.getOrElse(Option.scala:120)
	at org.apache.spark.streaming.rdd.WriteAheadLogBackedBlockRDD.compute(WriteAheadLogBackedBlockRDD.scala:167)
	at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:306)
	at org.apache.spark.CacheManager.getOrCompute(CacheManager.scala:69)
	at org.apache.spark.rdd.RDD.iterator(RDD.scala:268)
	at org.apache.spark.api.csharp.CSharpRDD.compute(CSharpRDD.scala:78)
	at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:306)
	at org.apache.spark.rdd.RDD.iterator(RDD.scala:270)
	at org.apache.spark.scheduler.ResultTask.runTask(ResultTask.scala:66)
	at org.apache.spark.scheduler.Task.run(Task.scala:89)
	at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:214)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
Caused by: java.lang.IllegalArgumentException: Pathname /C:/tmp/hadoop-t-zixzha/nm-local-dir/usercache/t-zixzha/appcache/application_1470197319438_0002/container_1470197319438_0002_02_000002/tmp/02ee4ba1-15c6-4950-bcd9-9038d0d9e7ed from C:/tmp/hadoop-t-zixzha/nm-local-dir/usercache/t-zixzha/appcache/application_1470197319438_0002/container_1470197319438_0002_02_000002/tmp/02ee4ba1-15c6-4950-bcd9-9038d0d9e7ed is not a valid DFS filename.
	at org.apache.hadoop.hdfs.DistributedFileSystem.getPathName(DistributedFileSystem.java:196)
	at org.apache.hadoop.hdfs.DistributedFileSystem.access$000(DistributedFileSystem.java:105)
	at org.apache.hadoop.hdfs.DistributedFileSystem$18.doCall(DistributedFileSystem.java:1118)
	at org.apache.hadoop.hdfs.DistributedFileSystem$18.doCall(DistributedFileSystem.java:1114)
	at org.apache.hadoop.fs.FileSystemLinkResolver.resolve(FileSystemLinkResolver.java:81)
	at org.apache.hadoop.hdfs.DistributedFileSystem.getFileStatus(DistributedFileSystem.java:1114)
	at org.apache.hadoop.fs.FileSystem.exists(FileSystem.java:1400)
	at org.apache.spark.streaming.util.FileBasedWriteAheadLog.initializeOrRecover(FileBasedWriteAheadLog.scala:227)
	at org.apache.spark.streaming.util.FileBasedWriteAheadLog.<init>(FileBasedWriteAheadLog.scala:72)
	at org.apache.spark.streaming.util.WriteAheadLogUtils$$anonfun$2.apply(WriteAheadLogUtils.scala:141)
	at org.apache.spark.streaming.util.WriteAheadLogUtils$$anonfun$2.apply(WriteAheadLogUtils.scala:141)
	at scala.Option.getOrElse(Option.scala:120)
	at org.apache.spark.streaming.util.WriteAheadLogUtils$.createLog(WriteAheadLogUtils.scala:140)
	at org.apache.spark.streaming.util.WriteAheadLogUtils$.createLogForReceiver(WriteAheadLogUtils.scala:110)
	at org.apache.spark.streaming.rdd.WriteAheadLogBackedBlockRDD.org$apache$spark$streaming$rdd$WriteAheadLogBackedBlockRDD$$getBlockFromWriteAheadLog$1(WriteAheadLogBackedBlockRDD.scala:138)
```
### 4. Key Point
```
Caused by: java.lang.IllegalArgumentException: Pathname /C:/tmp/hadoop-t-zixzha/nm-local-dir/usercache/t-zixzha/appcache/application_1470197319438_0002/container_1470197319438_0002_02_000002/tmp/02ee4ba1-15c6-4950-bcd9-9038d0d9e7ed from C:/tmp/hadoop-t-zixzha/nm-local-dir/usercache/t-zixzha/appcache/application_1470197319438_0002/container_1470197319438_0002_02_000002/tmp/02ee4ba1-15c6-4950-bcd9-9038d0d9e7ed is not a valid DFS filename.
```

### 5. Reason & Analysis
As you can see from the debug log and key point, the error occurs at ```org.apache.hadoop.hdfs.DistributedFileSystem.getPathName()```, the error occurs at Hadoop process. Here is the source code:
```
  private String getPathName(Path file) {
    checkPath(file);
    String result = file.toUri().getPath();
    if (!DFSUtil.isValidName(result)) {
      throw new IllegalArgumentException("Pathname " + result + " from " +
                                         file+" is not a valid DFS filename.");
    }
    return result;
  }
```
The Parameter ```Path file```, in this case is ```C:/tmp/hadoop```. In this function, it first ```checkPath```, nothing to say. Then another temperary viriable ```result``` is generated by ```file.toUri().getPath()```, in this case, the ```result``` is ```/C:/tmp/hadoop```. For now, everything is in peace. Then it will validate the ```result``` viriable by ```DFSUtil```. Here is the source code of ```DFSUtil.isValidName```
```
  /**
   * Whether the pathname is valid.  Currently prohibits relative paths, 
   * names which contain a ":" or "//", or other non-canonical paths.
   */
  public static boolean isValidName(String src) {
    // Path must be absolute.
    if (!src.startsWith(Path.SEPARATOR)) {
      return false;
    }
      
    // Check for ".." "." ":" "/"
    String[] components = StringUtils.split(src, '/');
    for (int i = 0; i < components.length; i++) {
      String element = components[i];
      if (element.equals(".")  ||
          (element.indexOf(":") >= 0)  ||
          (element.indexOf("/") >= 0)) {
        return false;
      }
  ...
```
From the comments, we can tell that the ```src``` is invalid as long as the string contain ```:```. However, ```/C:/tmp/hadoop``` is a legal path. Apparently, This code does not consider paths on **Windows**.

### 6. Why Spark Trigger This Error?
As I said on the top, the application worked well until the driver restarted and recovered from failure, the down. But, why this happened?
It's really complicated, I'll start at the ```WriteAheadLogBackedBlockRDD.compute``` function.
```
    // File: WriteAheadLogBackedBlockRDD.scala
    def compute()
    {
        ...
        if (partition.isBlockIdValid) 
        {
            getBlockFromBlockManager().getOrElse { getBlockFromWriteAheadLog() } // Here is the entry.
        }
        else {
            getBlockFromWriteAheadLog()
        }
    }
```
As you can see, Spark trys to get data block from ```getBlockFromBlockManager()```, However, the ApplicationMaster just recovered from failue, it detected that there are failed jobs left, then it try to get the block data. But, ```BlockManager``` is empty, then it trys to get block from WriteAheadLog by call the ```getBlockFromWriteAheadLog``` function. Here is what ```getBlockFromWriteAheadLog``` function does:
```
    def getBlockFromWriteAheadLog(): Iterator[T] = {
      var dataRead: ByteBuffer = null
      var writeAheadLog: WriteAheadLog = null
      try {
        // The WriteAheadLogUtils.createLog*** method needs a directory to create a
        // WriteAheadLog object as the default FileBasedWriteAheadLog needs a directory for
        // writing log data. However, the directory is not needed if data needs to be read, hence
        // a dummy path is provided to satisfy the method parameter requirements.
        // FileBasedWriteAheadLog will not create any file or directory at that path.
        // FileBasedWriteAheadLog will not create any file or directory at that path. Also,
        // this dummy directory should not already exist otherwise the WAL will try to recover
        // past events from the directory and throw errors.
        val nonExistentDirectory = new File(
          System.getProperty("java.io.tmpdir"), UUID.randomUUID().toString).getAbsolutePath
        writeAheadLog = WriteAheadLogUtils.createLogForReceiver(
          SparkEnv.get.conf, nonExistentDirectory, hadoopConf)
        dataRead = writeAheadLog.read(partition.walRecordHandle)
```
It's really clear in the comment: To read block from ```WriteAheadLog```, it needs a non-existent directory, then the ```new File(System.getProperty("java.io.tmpdir"), UUID.randomUUID().toString())``` is called. the path is what have shown above: ```C:\tmp\...```. 

So, the process will be dead once it trys to read data from WriteAheadLog on Windows.

### 7. Breif Summary
* Spark Streaming application will be dead once it trys to read data from WriteAhead on Windows.
* When the driver recovers from failure, it will read data from WriteAheadLog
* When the block not exists in BlockManager, it will read data from WriteAheadLog.
