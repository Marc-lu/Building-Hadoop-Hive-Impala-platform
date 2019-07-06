# Building-Hadoop-Hive-Impala-Platform
To build a big data platform including hadoop/hive/impala based on server cluster or virtual machines

## Fully distributed Hadoop

### On Virtual Machines

Machine: one namenode, two datanodes

Network:

|Machine name |IP address |
|-----|-----|
|namenode1|192.168.17.10|
|datanode1|192.168.17.11|
|datanode2|192.168.17.12|

**steps: configurate the namenode first, then copy two machines (datanode1 & datanode2), finally reset the datanodes**

- Configurate the CentOS7  

1. set the static IP  
`# vim /etc/sysconfig/network-scripts/ifcfg-en* 
//the content of * depends on your own machine, you can use command 'ls' to check, * is ens32 here `  
then write the follow content in it  
`BOOTPROTO=static`  
`ONBOOT=yes`  
`IPADDR=192.168.17.10`  
`NETMASK=255.255.255.0`  
`GATEWAY=192.168.17.2`  
`DNS1=192.168.17.2`  
*if you configurate it on your server, please not to reset your network configuration, or something you nerver expect will happen, don't ask me how to know it*  

2. set the hostname
check the hostname
`# cat /etc/hostname`  
`vim /etc/hostname` // configurate the value to namenode1  

3. bind the IP and hostname to realize the name analysis  
` vim /etc/hosts`  
then append the content as follows  
`192.168.17.10 namenode1`  
`192.168.17.11 datanode1`  
`192.168.17.12 datanode2`  

4. restart the network service, then use `ping` to test
`# systemctl restart network`  
`# ping namenode1`  

5. close the firework (CentOS use the firewall as default FireWall but not iptables)
`systemctl stop firewalld` // stop the firewall service  
`systemctl disable firewalld`  

6. shutdown SELinux
`# setenforce 0` // close it temporarily
`# vim /etc/selinux/config` // close permanently after restart by set `SELINUX=disabled`  

7. create hadoop group and user (if you have created the 'hadoop' user when you install the OS, just skip it)  
`# groupadd hadoop` //create hadoop group  
`# useradd -g hadoop hadoop` //create user 'hadoop' and add it into hadoop group  
`# passwd hadoop` // set the password for user 'hadoop'  

- Install and set the JDK environment  

1. download the corresponding rpm package of JDK from Oracle, here use the jdk-8u131-linux-x64 version  

2. upload the download rpm file to /root directory to install JDK  
`# cd /root`  
`# rpm -ivh jdk-8u131-linux-x64.rpm`  

3. append the JDK environment variables in /etc/profile  
`# vim /etc/profile`  
`export JAVA_HOME=/usr/java/jdk.1.8.0_131`  
`export JRE_HOME=$JAVA_HOME/jre`  
`export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:.`  
`export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH`  
then restart the configuration file to valid the setting  
`# source /etc/profile`  

4. test whether the JDK is successfully installed  
`# java -version`
`java version "1.8.0_131"...`  

- Configurate Hadoop (namenode)

1. download the hadoop package  
(1)[apache hadoop:](http://www-us.apache.org/dist/hadoop/common/ )  
(2)[cloudera hadoop(CDH)](http://archive-primary.cloudera.com/cdh5/cdh/5/) (recommended)  

2. install cloudera hadoop-2.6.0-cdh5.12.1  
`# tar -zxvf hadoop-2.6.0-cdh5.12.1.tar.gz -C /home/hadoop` //decompress  
`# cd /home/hadoop/hadoop-2.6.0-cdh5.12.1`  
`# mkdir hdfs hdfs/name hdfs/data logs tmp` // create 5 directories in hadoop directory  

3.  append the hadoop environment variables into `/home/hadoop/.bash_profile`  
`# vi /home/hadoop/.bash_profile`  
`export HADOOP_HOME=/home/hadoop/hadoop-2.6.0-cdh5.12.1` // hadoop main directory  
`export HADOOP_LOG_DIR=$HADOOP_HOME/logs` // hadoop log directory  
`export YARN_LOG_DIR=$HADOOP_LOG_DIR` //YARN log directory  
`export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH` //add the path of hadoop command  

4. modify nine configuration files (All are in `/home/hadoop/hadoop-2.6.0-cdh5.12.1/etc/hadoop`)  
(1) append JAVA_HOME to hadoop-env.sh、mapred-env.sh、yarn-env.sh
`export JAVA_HOME=/usr/java/jdk1.8.0_131`  
(2) append the followed content into `slaves` file (save all the datanodes' hostname)  
`datanode1`  
`datenode2`  
(3) modify the log4j.properties file, append the content to the last line as follows:
`log4j.logger.org.apache.hadoop.util.NativeCodeLoader=ERROR`  
to avoid the warning : WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform...  
(4) edit the content of `core-site.xml`、`hdfs-site.xml`、`mapred-site.xml`、`yarn-site.xml`  
*Notes: these all 9 files can be upload and decompress into the `/home/hadoop/hadoop-2.6.0-cdh5.12.1/etc/hadoop` to rewrite the files by Winscp, I will give the resoures *
**core-site.xml ：** 
```
<configuration>
    <property>
		<name>fs.defaultFS</name>
		<value>hdfs://namenode1:9000</value>
    </property>
    <property>
		<name>hadoop.tmp.dir</name>
		<value>/home/hadoop/hadoop-2.6.0-cdh5.12.1/tmp</value>
    </property>
</configuration>
```  
the values of **namenode1** and **/home/hadoop/hadoop-2.6.0-cdh5.12.1/** depend on your installment of hadoop 

**hdfs-site.xml ：**  
```
<configuration>
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>/home/hadoop/hadoop-2.6.0-cdh5.12.1/hdfs/name</value>
	</property>
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>/home/hadoop/hadoop-2.6.0-cdh5.12.1/hdfs/data</value>
	</property>
	<property>
		<name>dfs.replication</name>
		<value>2</value>
	</property>
	<property>
		<name>dfs.permissions</name>
		<value>false</value>
	</property>
</configuration>
```  
the value of **/home/hadoop/hadoop-2.6.0-cdh5.12.1/** depends on your installment of hadoop

**mapred-site.xml ：**  
```
# cp mapred-site.xml.template mapred-site.xml		//use the template to create the file
<configuration>
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>
</configuration>
```

**yarn-site.xml ：**  
```
<configuration>
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value></property>
	<property>
		<name>yarn.resourcemanager.resource-tracker.address</name>
		<value>namenode1:8031</value>
	</property>
	<property>
		<name>yarn.resourcemanager.address</name>
		<value>namenode1:8032</value>
	</property>
        <property>
    		<name>yarn.resourcemanager.admin.address</name>
		<value>namenode1:8033</value>
	</property>
	<property>
		<name>yarn.resourcemanager.scheduler.address</name>
		<value>namenode1:8034</value>
	</property>
	<property>
		<name>yarn.resourcemanager.webapp.address</name>
		<value>namenode1:8088</value>
	</property>
	<property>
		<name>yarn.log-aggregation-enable</name>
		<value>true</value>
	</property>
	<property>
  		<name>yarn.log.server.url</name>
		<value>http://namenode1:19888/jobhistory/logs/</value>
	</property>
</configuration>
```  
the value of **namenode1** depends on your installment of hadoop 

5. set the owner of directory hadoop  
`chown -R hadoop:hadoop /home/hadoop/hadoop-2.6.0-cdh5.12.1` //modify the owner
`# ls -ld /home/hadoop/hadoop-2.6.0-cdh5.12.1` //check the result  

- configurate the datanodes  
1. shutdown the namenode machine and make two copies of the namenode machine, then rename them `Hadoop2`、`Hadoop3`  
Hadoop2 is datanode1  
Hadoop3 is datanode2  

2. start the machine and change the machine name and hostname、IP address like the namenode configuration, the restart the service  
```
# vim /etc/sysconfig/network-scripts/ifcfg-en*
# vim /etc/hostname
# systemctl restart network
```  

- Set the SSH login without password  
**Notice: the following steps must login with haoop account**  

1. create key for all nodes
`$ ssh-keygen -t rsa` // execute in all nodes to generate respective rsa keys, just press `enter` while being prompted  
*it will generate public key `id_rsa.pub` and private key `id_rsa` in `/home/hadoop/.ssh` by default*  
2. sent datanodes's public key to `.ssh/authorized_keys` in namenode1  
`$ ssh-copy-id -i ~/.ssh/id_rsa.pub namenode1` // execute it in datanodes  
answer `yes` and input the password of user 'hadoop'  

3. add the public key of namenode1 into `.ssh/authorized_keys` and then distribute it to datanodes  
```
$ cd ~/.ssh
$ cat id_rsa.pub >> authorized_keys				// execute in namenode1 to combine authorized_keys
$ chmod 600 authorized_keys					//set the privilege of authorized_keys with 600
$ scp authorized_keys datanode1:.ssh/				// answer `yes` then input the password of user 'hadoop' in datanode1  
$ scp authorized_keys datanode2:.ssh/				// answer `yes` then input the password of user 'hadoop' in datanode2  
```  

4. test ssh login
execute the following command to login other nodes  
`$ ssh hostname ` // enter `yes`, `hostname` can be datanode1 or namenode1 or others

- Start Hadoop  
1. format the HDFS file system  
`$ hdfs namenode -format` // remember that just execute it when you start hadoop in the first time! Just skip it next time  

2. start the HDFS to verify  
`$ start-dfs.sh` //namenode starts the Namenode process, datanodes start the DataNode process  
`$ jps` // check the process  

3. start YARN  
`$ start-yarn.sh` // namenode starts the ResourceManager process, datanodes start the NodeManager process  

4. start JobHistoryServer (MR)  
`$ mr-jobhistory-daemon.sh start historyserver` // namenode start the JobHistoryServer process  

- Test hadoop program `wordcount`  
1. create two testing files in Local Disk `f1.txt` and `'f2.txt`  
```
$ echo "hello world hello you" > f1.txt
$ echo "hello hadoop hello me" > f2.txt
```  

2. create directory `wcin` in hdfs to save testing files  
`$ hdfs dfs -mkdir /wcin`  

3. put these two files into directory `wcin`  
```
$ hdfs dfs -put f*.txt /wcin  //upload
$ hdfs dfs -ls /wcin	      //check
```  
4. execute `wordcount` to count the testing files, then put the result into `/wcount` (this directory can't exist before, it will be generated automatically)  
`$ hadoop jar ~/hadoop-2.6.0-cdh5.12.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0-cdh5.12.1.jar wordcount /wcin /wcout`  

5. check the result  
`$ hdfs dfs -cat /wcout/part-r-00000`  


### On Server  
**just skip the 1st and 5th step in `configurate the CentOS7  ` and when you bind the IP or connect the cluster, try to use the static IP instead of public IP address**



## Hive  
- Install MySQL  
1. upload the configuration files of MySQL by `WinSCP` (connect CentOS with `root` account)
(1) upload `mysql-community.repo` to `/etc/yum.repos.d/`  
(2) upload `RPM-GPG-KEY-mysql` to `/etc/pki/rpm-gpg/`  

2. update `yum` and install `mysql server` (install the `mysql client` by defalut at the same time)  
```
# yum repolist
# yum install mysql-server
```  

3. check out whether the components of MySQL are installed successfully  
`# rpm -qa | grep mysql`  

- Configurate MySQL  
1. start MySQL Server and check the status  
```
# systemctl start mysqld
# systemctl status mysqld
```  

2. check the version of MySQL  
`# mysql -V`  

3. connect the MySQL, the default password is null  
`# mysql -u root`  
`mysql>`  

4. check the database  
`mysql>show databases;` // be careful that it must be end with `;`, or the input character `>` will continue come up  

5. create hive metastore  
`mysql> create database hive;`  
`mysql>show databases;`  

6. create user `hive`, password `123456`  
`mysql>create user 'hive'@'%' identified by '123456';`  
Notice: `drop user` is the command of delete users  

7. authorize `hadoop` user all the privileges of database 'hive'  
`mysql>grant all privileges on hive.* to 'hive'@'%' with grant option;`  

8. check the new created MySQL user (database name:`mysql`, table name:`user`)  
`mysql> select host,user,password from mysql.user;`  

9. delete the null record of user, or you can't login with user `hive`  
`mysql> delete from mysql.user where user='';`  

10. refresh the system authorized table (no needs to restart mysql service)  
`mysql>flush privileges;`  

11. testing for login with user `hive`  
`$ mysql -u hive -p`  
`Enter password：123456`  

- Install and Config hive  
1. download [hive](http://archive.cloudera.com/cdh5/cdh/5/) CDH version, then upload it to `/home/hadoop` in CentOS with `WinSCP`  

2. decompress `hive-1.1.0-cdh5.12.1.tar.gz` to `/home/hadoop`  
`$ tar zxvf hive-1.1.0-cdh5.12.1.tar.gz`  

3. add the hive environment variables into `.bash_profile`  
```
export HIVE_HOME=/home/hadoop/hive-1.1.0-cdh5.12.1
export PATH=$HIVE_HOME/bin:$PATH
```  

4. valid the settings above  
`$ source .bash_profile`  

5. edit `$HIVE_HOME/conf/hive-env.sh`, append `HADOOP_HOME` in it  
```
$ cd $HIVE_HOME/conf
$ cp hive-env.sh.template hive-env.sh
$ vim hive-env.sh
HADOOP_HOME=/home/hadoop/hadoop-2.6.0-cdh5.12.1
```  

6. create the file `$HIVE_HOME/conf/hive-site.xml`  
```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
        <property>
                <name>javax.jdo.option.ConnectionDriverName</name>
                <value>com.mysql.jdbc.Driver</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionURL</name>
                <value>jdbc:mysql://192.168.17.10:3306/hive</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionUserName</name>
                <value>hive</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionPassword</name>
                <value>123456</value>
        </property>

	<property>
		<name>hive.metastore.warehouse.dir</name>
		<value>/hive/warehouse</value>
	</property>
	<property>
		<name>hive.exec.scratchdir</name>
		<value>/hive/tmp </value>
	</property>
        <property>
                <name>hive.metastore.schema.verification</name>
                <value>false</value>
        </property>
</configuration>
```  
the value of **jdbc:mysql://192.168.17.10:3306/hive** depends on your machine  

7. create the directory of data warehouse(to store hive data files) and temporary dicretory in HDFS  
`$ hdfs dfs -mkdir -p /hive/warehouse /hive/tmp`  

8. download [the driver of mysql connection](https://dev.mysql.com/downloads/connector/j/)  
upload the driver(here is `mysql-connector-java-8.0.13.jar`) to `$HIVE_HOME/lib`  

9. start hive  
`$ hive`  

10. check hive database  
`hive> show databases;`  

11. exit hive  
`hive> quit;`  

- Hive Command  
1. check hive environment  
`hive> set`  
`hive> set param=value;` // the setting just valid this time  

2. check tables in database  
`hive> show tables;` // check tables in current database  
`hive> show tables in default;` // check tables in selected database  

3. create table
`hive> create table user(id int, name string);`  

4. check the structure of table  
`hive> desc user;`  
`hive> desc formatted user;` // check the details  

5. insert data  
``hive> insert into user values (1,’zhao’);``  
``hive> insert into user values (2,’qian’),(3,’sun’),(4,’li’);``  

6. check data  
``hive> select * from user;``  

7. updata and delete data (not supported in hive at pesent)
`hive> update user set name=’wang’ where id=1;`  
`hive> delete from user where id=1;`  

8. create database  
`hive> create database mydb;` create in the default location  
`hive> create database testdb location ‘/hive/testdb’;`	create in selected directory of hdfs  

9. check the structure of database  
`hive> desc database testdb;`  

10. jump to other databases  
`hive> use testdb;`  

11. delete database  
`hive> drop database testdb; ` // can only delete null database
`hive> drop database mydb cascade;` // delete not-null database

12. generate the data file with rand.sh including five pieces of data  
`$ ./rand.sh 5 > d1.txt`  
create datelist  
`hive> create table datelist (date string, no int);`  
load data form local linux file system into hive table (cause error by separation symbol)  
`hive> load data local inpath ‘/home/hadoop/d1.txt’ into table datelist;`  
modify separation symbol to hive default separation symbol `^A` by sed, and then reload  
`$ sed ‘s/ /^A/g’ d1.txt > d2.txt`  
`hive> load data local inpath ‘/home/hadoop/d2.txt’ into table datelist;` //append load  
`hive> load data local inpath ‘/home/hadoop/d2.txt’ overwrite into table datelist;` // overwrite load  
create datelist2 (self-custom separation symbol) and load the data  
```
hive> create table datelist2 (date string, no int) row format delimited  
	> fields terminated by ‘ ‘  lines terminated by ‘\n’;
hive> load data local inpath ‘/home/hadoop/d1.txt’ into table datelist2` 
```  

13. inner join
`hive> select id, name, date from user, datelist where id=no;`  

14. sort
`hive> select id, name, date from user, datelist where id=no order by date desc;`  

15. execute shell comand in hive(add `!` ahead of shell command)  
`hive> ! ls /home/Hadoop`  

16. execute HDFS command in hive(discard prefix `hdfs`)  
`hive> dfs -ls /hive`  

17. execute hive command in shell  
`$ hive -e ‘select * from user ‘| more`  

18. execute the script of hive command  
`$ hive -f hive.hql`  

- Hive API  
1. add hive dependecy in `pom.xml` of maven project  
```
<dependency>
	<groupId>org.apache.hive</groupId>
	<artifactId>hive-exec</artifactId>
	<version>1.1.0</version>
	<exclusions>
		<exclusion>
			<artifactId>
				pentaho-aggdesigner-algorithm
			</artifactId>
			<groupId>org.pentaho</groupId>
		</exclusion>
	</exclusions>
</dependency>

<dependency>
	<groupId>org.apache.hive</groupId>
	<artifactId>hive-jdbc</artifactId>
	<version>1.1.0</version>
</dependency>
```  
*maven will download the packages related with hive automatically *  

2. create `package` and `class` in `src/main/java` (maven project)  

3. code and run, check the result in console

hiveserver2
