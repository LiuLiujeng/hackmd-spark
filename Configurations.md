# Spark Configuration

## Spark Properties
控制大部分的 application parameters ，且可透過 [SparkConf()](https://spark.apache.org/docs/2.4.7/api/scala/index.html#org.apache.spark.SparkConf) object 來設定，或透過 Java system properties。

### 使用 SparkConf 來設定
SparkConf 允許你設定一些通用的參數，可以利用 set() 來隨意的操作。例如，我們可以在初始化 Spark application時，指定為兩個 thread，如下所示：
```
## Scala
val conf = new SparkConf()
             .setMaster("local[2]")
             .setAppName("CountingSheep")
val sc = new SparkContext(conf)

## Python
from pyspark import SparkConf, SparkContext
conf = SparkConf().set('spark.master', 'local[2]')
                  .set('spark.app.name', 'CountingSheep')
sc = SparkContext(conf=conf)
```

如果某個屬性需要指定持續時間的話，應該給予一個時間單位的值。接受以下格式：
* 25ms (milliseconds)
* 5s (seconds)
* 10m or 10min (minutes)
* 3h (hours)
* 5d (days)
* 1y (years)

若如果此屬性需要指定 byte size的話，應該使用大小單位進行配置。接受以下格式：
* 1b (bytes)
* 1k or 1kb (kibibytes = 1024 bytes)
* 1m or 1mb (mebibytes = 1024 kibibytes)
* 1g or 1gb (gibibytes = 1024 mebibytes)
* 1t or 1tb (tebibytes = 1024 gibibytes)
* 1p or 1pb (pebibytes = 1024 tebibytes)

### 動態讀取 Spark Properties
在一些情況下，你可以想要避免 hard-code 設定值在 SparkConf。那麼你可以在運行時提供設定值：
```
./bin/spark-submit --name "My app" --master local[4] --conf spark.eventLog.enabled=false
  --conf "spark.executor.extraJavaOptions=-XX:+PrintGCDetails -XX:+PrintGCTimeStamps" myApp.jar
```
Spark shell 和 [spark-submit](https://spark.apache.org/docs/2.4.7/submitting-applications.html) 支援兩個方式去動態讀取設定值。
第一個是 command line 的 options，例如上述的 --master。
`spark-submit` 透過 flag `--conf/-c` 可以接受任何 Spark 屬性，但是有部份針對 Spark application 的屬性必須使用特殊的 flag。
`spark-submit` 也可以讀取來自 conf/spark-defaults.conf 的內容，檔案內容範例如下(空格也可用等號取代)：
```
spark.master            spark://5.6.7.8:7077
spark.executor.memory   4g
spark.eventLog.enabled  true
spark.serializer        org.apache.spark.serializer.KryoSerializer
```
任何指定為 flag 或屬性文件中的值都將傳遞給 application 並與透過 SparkConf 指定的屬性合併。
直接在 SparkConf 上設置的屬性具有最高優先級，然後 flags 傳遞給 `spark-submit` 或 `spark-shell`，然後是 spark-defaults.conf 文件中的選項。

Spark 屬性可以分成兩類：一個是與 deploy 有關，例如：`spark.driver.memory` 或 `spark.executor.instances`。在運行時通過 SparkConf 以編程方式設置時，此類屬性可能不會受到影響，或者行為取決於您選擇的集群管理器(cluster manager)和部署模式(deploy mode)，所以這類屬性建議通過 configuration file 或`spark-submit` command line options 進行設置。
另一個類型主要是關於 Spark runtime 的控制，例如：`spark.task.maxFailures`。這種屬性可以以任何一種方式設置。

### Viewing Spark Properties
位於 `http://<driver>:4040` 的 Application Web UI 在 `Environment` 選項中列出了 Spark 屬性。這是一個有用的地方，可以用來檢查並確保你的屬性設置正確。

### Available Properties
請參照[子頁](https://hackmd.io/@yachu66/ByVLNnN9u)。

## Environment Variables
透過在每個 node 的 conf/spark-env.sh 腳本，可用來設定每台機器的設定值，比如 IP 地址等。

某些 Spark 設置可以通過環境變量進行配置，這些環境變量是從 Spark 安裝目錄中的 `conf/spark-env.sh` 腳本中讀取的。
在 Standalone 和 Mesos 模式下，這個文件可以提供機器特定的信息，比如主機名稱。當運行 local Spark Application 或提交腳本時也會讀取此文件內容。

要注意的是，預設情況下在安裝 Spark 時，`conf/spark-env.sh` 是不存在的。不管如何，你可以透過複製 `conf/spark-env.sh.template` 來建立此腳本。

可以在 `spark-env.sh` 中設置以下變數：

| Environment Variable | 意義 |
| -------- | -------- |
| JAVA_HOME | Java 安裝的位置(如果它不在你的`PATH`上) |
| PYSPARK_PYTHON | 用於 PySpark 的 driver 和 wokers 的 Python 執行檔(預設是 `python2.7`，其次為 `python`)。如果設置了屬性 `spark.pyspark.python` 的話，以此設定值為優先。|
| PYSPARK_DRIVER_PYTHON | 僅用於 PySpark 的 driver的 Python 執行檔(預設是 `PYSPARK_PYTHON`)。如果設置了屬性 `spark.pyspark.driver.python` 的話，以此設定值為優先。|
| SPARKR_DRIVER_R | R binary executable to use for SparkR shell (default is R). Property spark.r.shell.command take precedence if it is set |
| SPARK_LOCAL_IP | 要綁定的機器的 IP 地址。 |
| SPARK_PUBLIC_DNS | Spark 程序將向其他機器通告的主機名。 |

除了上述之外，還有用於設置 Spark [standalone cluster scripts](https://spark.apache.org/docs/2.4.7/spark-standalone.html#cluster-launch-scripts) 的選項，例如每台機器上使用的內核數和最大內存。

備註：在 `cluster` mode 下的 YARN 上運行 Spark 時，需要使用 `spark.yarn.appMasterEnv` 來設置環境變數。
`conf/spark-defaults.conf` 文件中的 `[EnvironmentVariableName]` 屬性。
被設置在 `spark-env.sh` 的環境變數，將不會反映在 cluster mode 的 YARN Application Master process 中。請觀看 [YARN-related Spark Properties](https://spark.apache.org/docs/2.4.7/running-on-yarn.html#spark-properties) 取得更多資訊。

## Logging
Spark 使用 [log4j](http://logging.apache.org/log4j/) 進行日誌記錄。你可以透過在資料夾 `conf` 新增一個 `log4j.properties` 檔案來設置。一個簡單的開始方式是，複製一個現有的 `log4j.properties.template`。

## Overriding configuration directory
如果要修改預設的 configuration directory :"SPARK_HOME/conf" 的話，你可以設定環境變數 `SPARK_CONF_DIR`， Spark 會使用此資料夾底下的設定檔(spark-defaults.conf, spark-env.sh, log4j.properties, etc) 。

## Inheriting Hadoop Cluster Configuration
如果你打算使用 Spark 從 HDFS 讀取和寫入，Spark 的 classpath 中應該包含兩個 Hadoop 配置文件：
* hdfs-site.xml, which provides default behaviors for the HDFS client.
* core-site.xml, which sets the default filesystem name.
這些配置文件的位置因 Hadoop 版本而異，但是一個常見的位置是在 `/etc/hadoop/conf` 裡面。有一些工具會即時創建配置，但提供了下載它們的副本的機制。

若要讓 Spark 對以上這些配置文件做讀取，請將 `$SPARK_HOME/conf/spark-env.sh` 中的 HADOOP_CONF_DIR 設置為包含配置文件的位置。

## Custom Hadoop/Hive Configuration

