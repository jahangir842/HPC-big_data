To set up and run Apache PySpark on multiple virtual machines (VMs), you'll need to configure a Spark cluster. Here's a comprehensive guide to help you through the process:

### Prerequisites
1. **Virtual Machines**: Prepare four VMs. One will act as the master node, and the remaining three will serve as worker nodes.
2. **Java Development Kit (JDK)**: Apache Spark requires JDK 8 or later. Ensure Java is installed on all VMs.
3. **SSH Access**: Set up passwordless SSH access from the master node to all the worker nodes.

---

### Step-by-Step Guide

#### Step 1: Install Java on All VMs
Start by installing Java on each virtual machine:

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
```

Verify that Java is correctly installed:

```bash
java -version
```

#### Step 2: Download and Install Apache Spark
Download Spark on each VM. Choose a version that fits your needs (e.g., Spark 3.2.1 with Hadoop 3.2):

```bash
wget https://downloads.apache.org/spark/spark-3.2.1/spark-3.2.1-bin-hadoop3.2.tgz
tar xvf spark-3.2.1-bin-hadoop3.2.tgz
sudo mv spark-3.2.1-bin-hadoop3.2 /opt/spark
```

#### Step 3: Set Up Environment Variables
Add Spark and Java environment variables to the `.bashrc` file on each VM:

```bash
echo 'export SPARK_HOME=/opt/spark' >> ~/.bashrc
echo 'export PATH=$PATH:$SPARK_HOME/bin:$JAVA_HOME/bin' >> ~/.bashrc
echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> ~/.bashrc
source ~/.bashrc
```

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
