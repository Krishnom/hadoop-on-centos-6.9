# hadoop-on-centos-6.9


> Reference taken from [Techadmin Guide]( https://tecadmin.net/setup-hadoop-single-node-cluster-on-centos-redhat/).

## Install hadoop 3.1 on centos 6.9


- Install JAVA
```
$ cd /opt/
$ wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u171-b11/512cd62ec5174c3487ac17c61aaa89e8/jdk-8u171-linux-x64.tar.gz"
$ tar xzf jdk-8u171-linux-x64.tar.gz
$ cd /opt/jdk1.8.0_171/
$ alternatives --install /usr/bin/java java /opt/jdk1.8.0_171/bin/java 2
$ alternatives --config java
$ alternatives --install /usr/bin/jar jar /opt/jdk1.8.0_171/bin/jar 2
$ alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_171/bin/javac 2
$ alternatives --install /usr/bin/jps javac /opt/jdk1.8.0_171/bin/jps 2
$ alternatives --set jar /opt/jdk1.8.0_171/bin/jar
$ alternatives --set javac /opt/jdk1.8.0_171/bin/javac
$ alternatives --set jps /opt/jdk1.8.0_171/bin/jps
$ java -version

$ cat <<EOF >> /etc/bashrc
export JAVA_HOME=/opt/jdk1.8.0_171
export JRE_HOME=/opt/jdk1.8.0_171/jre
export PATH=$PATH:/opt/jdk1.8.0_171/bin:/opt/jdk1.8.0_171/jre/bin
EOF
```
- Create Dedicated Hadoop user
```
$ adduser hadoop
$ passwd hadoop
```
- Enable passwordless login from dedicated user to self
```
$ su - hadoop
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
```
- Verify keybased login 
```
$ ssh localhost
$ exit
```
- Hadoop 3.1 Installation
```
$ cd ~
$ wget http://www-eu.apache.org/dist/hadoop/common/hadoop-3.1.0/hadoop-3.1.0.tar.gz
$ tar xzf hadoop-3.1.0.tar.gz
$ mv hadoop-3.1.0 hadoop
```
- Setup Hadoop Environment Variables
```
$ cat <<EOF >> ~/bashrc
export HADOOP_HOME=/home/hadoop/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_PID_DIR=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
EOF

$ source ~/.bashrc
$ sed -i -e 's/#\ export\ JAVA_HOME=/export\ JAVA_HOME=\/opt\/jdk1.8.0_171/g' $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```


- Setup Hadoop Configuration Files
```
$ cat <<EOF > $HADOOP_HOME/etc/hadoop/core-site.xml
<configuration>
<property>
  <name>fs.default.name</name>
    <value>hdfs://localhost:9000</value>
</property>
</configuration>
EOF

$ cat <<EOF > $HADOOP_HOME/etc/hadoop/hdfs-site.xml
<configuration>
<property>
 <name>dfs.replication</name>
 <value>1</value>
</property>

<property>
  <name>dfs.name.dir</name>
    <value>file:///home/hadoop/hadoopdata/hdfs/namenode</value>
</property>

<property>
  <name>dfs.data.dir</name>
    <value>file:///home/hadoop/hadoopdata/hdfs/datanode</value>
</property>
</configuration>
EOF
```
- Format Namenode
```
$ hdfs namenode -format
```

- Start Hadoop Cluster
```
$ start-dfs.sh
```

> Note : Hadoop NameNode started on port 9870 default. Access your server on port 9870 in your favorite web browser.
> Note : Access port 9864 to get details about your Hadoop node.


- Setting up Hadoop to run at boot on SystemV (init.d) systems(requires root access)
1. Copy hadoop-dfs-init_d file in /etc/init.d/
2. rename the file to hadoop-dfs
```	$ mv /etc/init.d/hadoop-dfs-init_d /etc/init.d/hadoop-dfs ```
3. Add executable permission to script
```     $ chmod +x /etc/init.d/ ```
4. Add the script to run at boot
```	$ chkconfig hadoop-dfs on ```
5. Vertify that the hadoop script is installed fine.
```     $ service hadoop-dfs status 
    $ service hadoop-dfs stop
    $ service hadoop-dfs start 
```
