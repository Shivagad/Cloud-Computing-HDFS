# 📊 Marks Data Analysis over Private Cloud (HDFS on Azure)

This project demonstrates a solution for analyzing large-scale marks data using **Hadoop HDFS**, **MapReduce**, and **custom private cloud infrastructure** deployed over a **Local Area Network (LAN)**. It is hosted on **Azure Virtual Machines**, enabling distributed storage, encryption, and statistical computation (average marks) with Hadoop ecosystem tools.

---

## 🚀 Project Overview

- 💾 **Store & process large marks datasets**
- 🔐 **Upload/download encrypted files**
- ☁️ **Hosted over a private cloud using Azure VMs**
- ⚙️ **Distributed using Hadoop HDFS and MapReduce**
- 📈 **Average marks calculation with custom Java code**

---

## 📡 System Architecture

- `1 Master Node` (NameNode, ResourceManager)
- `2 Data Nodes` (DataNode services)
- Hosted on `Azure Virtual Machines` in the same **Virtual Network**
- SSH enabled, passwordless login among nodes
- HDFS used for file segmentation and replication

<p>
  <img src="Screenshots/IMG-20250423-WA0014.jpg" alt="System Architecture" width="45%" style="margin-right: 10px;"/>
  <img src="Screenshots/IMG-20250423-WA0013.jpg" alt="VM Setup" width="45%"/>
</p>

---

## 🛠️ Technology Stack

| Component         | Technology Used           |
|------------------|---------------------------|
| Distributed FS   | Hadoop HDFS               |
| Processing       | Java MapReduce            |
| Encryption       | OpenSSL                   |
| Platform         | Azure Cloud (VMs)         |
| OS               | Ubuntu 20.04 LTS          |
| Scripts          | Bash                      |

---

## 🧰 System Requirements

- **Hardware:**
  - 1 Master + 2 DataNodes (each: 4GB RAM, 100GB HDD)
  - Stable LAN (Azure Virtual Network)

- **Software:**
  - Java JDK 8+
  - Hadoop 3.x
  - OpenSSL
  - SSH/SCP

---

## 📦 Java MapReduce Code – Average Marks Calculator

We use MapReduce to compute **average marks of students** from a file stored in HDFS.

### ✅ Input format (HDFS file):

```
Alice 85  
Bob 70  
John 100  
```

### ✅ Output:

```
Alice    85.0
Bob      70.0
John     100.0
```

### 🧾 Files:
- [`MarksMapper.java`](Src/MarksMapper.java) – emits (student, marks)
- [`MarksReducer copy.java`](Src/MarksReducer%20copy.java) – calculates average of student marks
- [`AverageMarksDriver.java`](Src/AverageMarksDriver.java) – configures and runs the job

> 💡 Code available in the [`Src`](Src) folder.

---

## ☁️ 1. Connect to Azure Virtual Machines

After deploying 3 Ubuntu VMs on Azure (Master, DataNode1, DataNode2):

On your local system (with .pem file or SSH key):

```bash
ssh username@<master_public_ip>
```
Repeat for datanodes:
```bash
ssh username@<datanode1_ip>
ssh username@<datanode2_ip>
```

<img src="Screenshots/IMG-20250423-WA0006.jpg" alt="SSH to VM" width="600px" height="400px"/>

---

## 🔧 2. Install Java & Hadoop on All Nodes

Java JDK:

```bash
sudo apt update
sudo apt install openjdk-8-jdk -y
java -version
```

Hadoop (repeat on each node):

```bash
wget https://downloads.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
tar -xzf hadoop-3.3.6.tar.gz
sudo mv hadoop-3.3.6 /usr/local/hadoop
```

Set environment variables (~/.bashrc):

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
source ~/.bashrc
```

---

## 🔐 3. Enable SSH & Passwordless SSH (from Master to DataNodes)

On Master:

```bash
ssh-keygen -t rsa
ssh-copy-id user@datanode1_ip
ssh-copy-id user@datanode2_ip
```

Also, edit `/etc/hosts` on all 3 machines for hostname resolution.

---

## 🛠️ 4. Configure Hadoop Cluster (Master & DataNodes)

Edit the following config files on all nodes (from `$HADOOP_HOME/etc/hadoop/`):

**core-site.xml:**
```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://master:9000</value>
  </property>
</configuration>
```

**hdfs-site.xml:**
```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///usr/local/hadoop/hdfs/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///usr/local/hadoop/hdfs/datanode</value>
  </property>
</configuration>
```

**mapred-site.xml:**
```bash
cp mapred-site.xml.template mapred-site.xml
```
```xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```

**yarn-site.xml:**
```xml
<configuration>
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>master</value>
  </property>
</configuration>
```

---

## 🏁 5. Start Hadoop Cluster (on Master Node)

Format the Namenode (first time only):

```bash
hdfs namenode -format
```

Start HDFS:

```bash
start-dfs.sh
```

Start YARN:

```bash
start-yarn.sh
```

Verify:

```bash
jps     # should show NameNode, DataNode, ResourceManager, etc.
```

<img src="Screenshots/IMG-20250423-WA0008.jpg" alt="Hadoop Services" width="45%" style="margin-right: 10px;"/>
<img src="Screenshots/IMG-20250423-WA0012.jpg" alt="Hadoop Services" width="45%"/>

---

## 📤 6. Send Java Code and Create JAR

On your local machine:

```bash
scp Src/MarksMapper.java Src/MarksReducer\ copy.java Src/AverageMarksDriver.java username@<master_ip>:/home/username/
```

On master:

```bash
javac -classpath $(hadoop classpath) -d avg_classes Src/*.java
jar -cvf average_marks.jar -C avg_classes/ .
```

---

## 📁 7. Upload Input File to HDFS

Put into HDFS:

```bash
hdfs dfs -mkdir /input
hdfs dfs -put stud_data.csv /input/
```

---

## 🧮 8. Run MapReduce Job

```bash
hadoop jar average_marks.jar AverageMarksDriver /input /output
```

<img src="Screenshots/IMG-20250423-WA0009.jpg" alt="Run MapReduce Job" width="600px" height="400px"/>

---

## 📂 9. Check Output

```bash
hdfs dfs -ls /output
hdfs dfs -cat /output/part-r-00000
```

<img src="Screenshots/IMG-20250423-WA0005.jpg" alt="Output" width="600px" height="400px"/>

---

## 📸 Additional Screenshots

<p>
  <img src="Screenshots/IMG-20250423-WA0007.jpg" alt="Extra Screenshot" width="45%" style="margin-right: 10px;"/>
  <img src="Screenshots/IMG-20250423-WA0010.jpg" alt="Extra Screenshot" width="45%"/>
  <img src="Screenshots/IMG-20250423-WA0011.jpg" alt="Extra