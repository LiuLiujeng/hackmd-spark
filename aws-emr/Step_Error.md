# 如何對 Amazon EMR 中失敗的 Spark 步驟進行故障排除？

針對 deploy mode 的方式分為兩種。

## Client mode jobs

透過 **--deploy-mode client** 提交的 Spark jobs。
Step logs 提供關於 job parameters 和 step error messages。這些 log 會[存檔到 Amazon Simple Storage Service (Amazon S3)](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-manage-view-web-log-files.html#emr-manage-view-web-log-files-s3)。
例如：
* s3://aws-logs-111111111111-us-east-1/elasticmapreduce/j-35PUYZBQVIJNM/steps/s-2M809TD67U2IA/controller.gz
  * 此檔案包含了 spark-submit 指令。檢查這個 log 可以看到 job 的 parameters。
* s3://aws-logs-111111111111-us-east-1/elasticmapreduce/j-35PUYZBQVIJNM/steps/s-2M809TD67U2IA/stderr.gz
  * 此檔案提供了 driver logs. (When the Spark job runs in client mode, the Spark driver runs on the master node.)

要找出 step 失敗的原因，執行下列指令來下載 step logs，然後對其尋找警告或錯誤：
```
#Download the step logs:
aws s3 sync s3://aws-logs-111111111111-us-east-1/elasticmapreduce/j-35PUYZBQVIJNM/steps/s-2M809TD67U2IA/ s-2M809TD67U2IA/

#Open the step log folder:
cd s-2M809TD67U2IA/

#Uncompress the log file:
find . -type f -exec gunzip {} \;

#Get the yarn application id from the cluster mode log:
grep "Client: Application report for" * | tail -n 1

#Get the errors and warnings from the client mode log:
egrep "WARN|ERROR" *
```

例如下列檔案：
s3://aws-logs-111111111111-us-east-1/elasticmapreduce/j-35PUYZBQVIJNM/containers/application_1572839353552_0008/container_1572839353552_0008_01_000001/stderr.gz

記載了記憶體錯誤：
```
19/11/04 05:24:45 ERROR SparkContext: Error initializing SparkContext.
java.lang.IllegalArgumentException: Executor memory 134217728 must be at least 471859200. Please increase executor memory using the --executor-memory option or spark.executor.memory in Spark configuration.
```

利用此log資訊去解決錯誤。
如範例，要解決記憶體問題，在提交 job 時要給予 executor 更多的記憶體：
```
spark-submit --deploy-mode client --executor-memory 4g --class org.apache.spark.examples.SparkPi /usr/lib/spark/examples/jars/spark-examples.jar
```

## Cluster mode jobs

Check the stderr step log to identify the ID of the application that's associated with the failed step.
This log:
s3://aws-logs-111111111111-us-east-1/elasticmapreduce/j-35PUYZBQVIJNM/steps/s-2M809TD67U2IA/stderr.gz

identifies **application_1572839353552_0008**:
```
19/11/04 05:24:42 INFO Client: Application report for application_1572839353552_0008 (state: ACCEPTED)
```

Identify the application master logs. When the Spark job runs in cluster mode, the Spark driver runs inside the application master. The application master is the first container that runs when the Spark job executes. The following is an example list of Spark application logs.

In this list, **container_1572839353552_0008_01_000001** is the first container, which means that it's the application master.

s3://aws-logs-111111111111-us-east-1/elasticmapreduce/j-35PUYZBQVIJNM/containers/application_1572839353552_0008/container_1572839353552_0008_01_000001/stderr.gz

s3://aws-logs-111111111111-us-east-1/elasticmapreduce/j-35PUYZBQVIJNM/containers/application_1572839353552_0008/container_1572839353552_0008_01_000001/stdout.gz

s3://aws-logs-111111111111-us-east-1/elasticmapreduce/j-35PUYZBQVIJNM/containers/application_1572839353552_0008/container_1572839353552_0008_01_000002/stderr.gz

s3://aws-logs-111111111111-us-east-1/elasticmapreduce/j-35PUYZBQVIJNM/containers/application_1572839353552_0008/container_1572839353552_0008_01_000002/stdout.gz

After you identify the application master logs, download the logs to an Amazon EC2 instance. Then, search for warnings and errors. For example:
```
#Download the Spark application logs:
aws s3 sync s3://aws-logs-111111111111-us-east-1/elasticmapreduce/j-35PUYZBQVIJNM/containers/application_1572839353552_0008/ application_1572839353552_0008/

#Open the Spark application log folder:
cd application_1572839353552_0008/ 

#Uncompress the log file:
find . -type f -exec gunzip {} \;

#Search for warning and errors inside all the container logs. Then, open the container logs returned in the output of this command.
egrep -Ril "ERROR|WARN" . | xargs egrep "WARN|ERROR"
```

## References
* [原文出處](https://aws.amazon.com/tw/premiumsupport/knowledge-center/emr-spark-failed-step/)