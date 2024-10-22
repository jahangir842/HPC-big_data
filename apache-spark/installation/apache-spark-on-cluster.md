

#### Step 4: Configure SSH Access
On the master node, generate an SSH key and distribute it to each worker node for passwordless access:

```bash
ssh-keygen -t rsa -P ""
ssh-copy-id -i ~/.ssh/id_rsa.pub user@worker-node-1
ssh-copy-id -i ~/.ssh/id_rsa.pub user@worker-node-2
ssh-copy-id -i ~/.ssh/id_rsa.pub user@worker-node-3
```

#### Step 5: Configure Spark
On the master node, modify the `conf/spark-env.sh` file to configure the Spark environment. If the file doesn't exist, create it from the template:

```bash
cp /opt/spark/conf/spark-env.sh.template /opt/spark/conf/spark-env.sh
```

Add the following configuration to set the master node IP and Java environment:

```bash
export SPARK_MASTER_HOST='master-node-ip'
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

Then, create a `slaves` file in the `conf` directory on the master node and add the IP addresses of your worker nodes:

```bash
echo 'worker-node-1-ip' >> /opt/spark/conf/slaves
echo 'worker-node-2-ip' >> /opt/spark/conf/slaves
echo 'worker-node-3-ip' >> /opt/spark/conf/slaves
```

#### Step 6: Start the Cluster
On the master node, start the Spark master process:

```bash
/opt/spark/sbin/start-master.sh
```

Next, on each worker node, start the worker process and connect it to the master node:

```bash
/opt/spark/sbin/start-slave.sh spark://master-node-ip:7077
```

#### Step 7: Verify the Setup
To check if the cluster is running, open a web browser and go to `http://master-node-ip:8080`. You should see the Spark master web interface showing the connected worker nodes.

#### Step 8: Submit a Spark Job
Create a simple Python script named `example.py` to test your setup:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("ExampleApp").getOrCreate()
data = [('Alice', 1), ('Bob', 2), ('Cathy', 3)]
df = spark.createDataFrame(data, ['Name', 'Age'])
df.show()
spark.stop()
```

Submit this job from the master node using the following command:

```bash
/opt/spark/bin/spark-submit --master spark://master-node-ip:7077 example.py
```

---

### Conclusion
Following these steps, you will successfully configure a functional Apache Spark cluster across multiple virtual machines, enabling you to run and manage PySpark applications efficiently. For more advanced configurations, consult the official Spark documentation.
