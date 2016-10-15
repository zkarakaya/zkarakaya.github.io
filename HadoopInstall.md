# Installing Hadoop 2.7.3 on Ubuntu 14.04 or Ubuntu 16.04
> Following commansd are to be executed within a terminal session. If you have installed Ubuntu Desktop, then you can create 
a terminal session by ``Ctrl+Alt+T``. If you have installed Ubuntu Server, then by default you will be using a black shell screen. 
Each line in the box is a seperate shell command, and you must execute them one by one.

### Java is needed
If not installed yet, I advise you use Oracle Java 8. To install Oracle Java on your computer execute the following codes;

```
apt-add-repository ppa:webupd8team/java
apt-get update
apt-get install oracle-java8-installer
apt-get install oracle-java8-set-default
``` 
Check if your ``JAVA_HOME`` environment variable is set;
```
env | grep JAVA
```
If nothing is displayed, close and open your shell session. As a result of above command ``JAVA_HOME`` should be displayed with a path where the Java is installed. If nothing is displayed, there is something wrong with Java Installation.

### Enable Password authentication for ``ssh``

For this, simply execute the following commands;
<pre>
sudo sed -i -e 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
service ssh restart
</pre>

### Hadoop ``group`` and ``user`` creation
adduser command will prompt you for ``hadoop`` password. You may simply use ``hadoop`` as a password. 
You will be asked to enetr same password twice. Don't forget the password because you will need it later.
<pre>
sudo addgroup hadoop
adduser --ingroup hadoop hadoop
</pre>

###Download and extract Hadoop 

Currenct latest stable Hadoop version is 2.7.3. Download and extract it by the following commands;
<pre>
wget http://ftp.itu.edu.tr/Mirror/Apache/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz
tar -xzvf hadoop-2.7.3.tar.gz -C /opt
mv /opt/hadoop-2.7.3 /opt/hadoop
chown -R hadoop:hadoop /opt/hadoop
mkdir -p /var/lib/hadoop/hdfs/namenode
mkdir -p /var/lib/hadoop/hdfs/datanode
chown -R hadoop /var/lib/hadoop
</pre>

### Login to hadoop user
<pre>
su - hadoop
</pre>
This command will not ask for password.

### Create RSA keys
Your hadoop would need to open session during code execution. To enable this feature, hadoop needs to access
without providing password. To allow hadoop, we must create and authorize this key for ``hadoop`` user.

First step is to create keys. Following command will ask you for file name by providing the default value. 
Default is good enough, so when you asked just push enter to accept the default filename.
<pre>
ssh-keygen -t rsa -P ""
</pre>
Then execute the following command to put the into ``authorized_keys`` file. 
Command will give you a fingerprint, accept it. Then you would be asked for password of ``hadoop` user, enter the password you choosed
when creating ``hadoop`` account.
<pre>
ssh-copy-id -i ~/.ssh/id_rsa localhost
</pre>

We have installed the Hadoop but we need to configure it. Is is not yet ready to use.

To put enviroment variables into .bashrc file, copy the following lines and put it at the end of ``.bashrc`` file. You 
can edit the file by using ``nano`` editor. To edit you can use ``nano .bashrc`` command.

<pre>
export HADOOP_INSTALL=/opt/hadoop
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_HOME=$HADOOP_INSTALL
export HADOOP_HDFS_HOME=$HADOOP_INSTALL
export YARN_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_INSTALL/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_INSTALL/lib/native"
</pre>

Now we need to know ``JAVA_HOME`` path once again. We need to copy and write it in ``hadoop-env.sh`` file.

Get the environment value by;
<pre>
env | grep JAVA_HOME
</pre>
Copy the path, edit the `hadoop-env.sh` with ``nano /opt/hadoop/etc/hadoop/hadoop-env.sh`` command and update JAVA_HOME variable with the path you have copied. The file
contains 
```
export JAVA_HOME=${JAVA_HOME}
```
and shoud look like;
```
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
```
The path above may differ in your computer. Be carefull to write correct path of ``JAVA_HOME``.

Now edit `core-site.xml` file with ``nano /opt/hadoop/etc/hadoop/core-site.xml`` command and put the following properties
between `<configuration>` and `</configuration>`

It should look like;
```
 <property>
    <name>fs.default.name</name>
    <value>hdfs://localhost:9000</value>
  </property>
```

Edit `yarn-site.xml` with ``nano /opt/hadoop/etc/hadoop/yarn-site.xml`` command and put the following properties 
into ``configuration`` section.

```
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
```

Now we are to copy a template XML file. Execute the command;
```
cp /opt/hadoop/etc/hadoop/mapred-site.xml.template /opt/hadoop/etc/hadoop/mapred-site.xml
```

And then edit the file with ``nano /opt/hadoop/etc/hadoop/mapred-site.xml`` command. Put the following properties 
into `configuration` section.
```
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
```

The last file we are to edit is `hdfs-site.xml` file. Use ``nano /opt/hadoop/etc/hadoop/hdfs-site.xml`` command to edit
and put the followings into `configuration` section.

```
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/var/lib/hadoop/hdfs/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/var/lib/hadoop/hdfs/datanode</value>
  </property>
```
We are almost finished.

###Format HDFS and start services.

Execute the following commands one by one.
```
source .bashrc
hdfs namenode -format
start-dfs.sh
start-yarn.sh
```

If nothing goes wrong, you wouldn't get error and YOU HAVE FINISHED. 
Let us check if it is working. Executing the command ``jps`` will give you an output like;
```
9280 Jps
1939 SecondaryNameNode
2452 NodeManager
1509 NameNode
1702 DataNode
2106 ResourceManager
```
The numbers in front of the service names may differ. If they are all listed, THAT'S ALL. Your Hadoop is ready to use.
Let us execute an example mapreduce code by executing the command;
```
hadoop jar /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar pi 10 100
```

Congratulations. You have done. Enjoy your Hadoop.
