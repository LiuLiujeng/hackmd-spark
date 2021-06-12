# How do I configure Amazon EMR to run a PySpark job using Python 3.4 or 3.6?

在眾多的 Amazon EMR release versions，cluster instances 和 system application 預設使用不同版本的 Python：

* Amazon EMR release versions 4.6.0-5.19.0: 
  * Python 3.4 is installed on the cluster instances. Python 2.7 is the system default.
* Amazon EMR release versions 5.20.0 and before 5.30.0:
  * Python 3.6 is installed on the cluster instances. For 5.20.0-5.29.0, Python 2.7 is the system default.
* Amazon EMR release versions 5.30.0 and later:
  * Python 3 is the system default.

若要升級 PySpark 使用的 Python 版本，將 spark-env classification 的 PYSPARK_PYTHON 環境變數指定為安裝 Python 3.4 或 3.6 的目錄。

## On a running cluster

### Amazon EMR release version 5.21.0 and later

Submit a [reconfiguration request](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-configure-apps-running-cluster.html) with a configuration object similar to the following:

```
[
  {
     "Classification": "spark-env",
     "Configurations": [
       {
         "Classification": "export",
         "Properties": {
            "PYSPARK_PYTHON": "/usr/bin/python3"
          }
       }
    ]
  }
]
```

### Amazon EMR release version 4.6.0-5.20.x
1. [Connect to the master node using SSH](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-connect-master-node-ssh.html).

2. Run the following command to change the default Python environment:
```
sudo sed -i -e '$a\export PYSPARK_PYTHON=/usr/bin/python3' /etc/spark/conf/spark-env.sh
```

3. Run the pyspark command to confirm that PySpark is using the correct Python version:
```
[hadoop@ip-X-X-X-X conf]$ pyspark
```
The output shows that PySpark is now using the same Python version that is installed on the cluster instances. Example:
```
Python 3.4.8 (default, Apr 25 2018, 23:50:36)

Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 2.3.1
      /_/

Using Python version 3.4.8 (default, Apr 25 2018 23:50:36)
SparkSession available as 'spark'.
```
Spark uses the new configuration for the next PySpark job.


## On a new cluster

[Add a configuration object](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-configure-apps.html) similar to the following when you launch a cluster using Amazon EMR release version 4.6.0 or later:
```
[
  {
     "Classification": "spark-env",
     "Configurations": [
       {
         "Classification": "export",
         "Properties": {
            "PYSPARK_PYTHON": "/usr/bin/python3"
          }
       }
    ]
  }
]
```

## References
* [原文出處](https://aws.amazon.com/tw/premiumsupport/knowledge-center/emr-pyspark-python-3x/)
* [關於如何在 EMR 設置 Spark](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-spark-configure.html)