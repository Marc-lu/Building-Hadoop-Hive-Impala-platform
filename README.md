# Building-Hadoop-Hive-Impala-Platform
To build a big data platform including hadoop/hive/impala based on server cluster or virtual machines

## 1. fully distributed Hadoop

### virtual machines version

Machine: one namenode, two datanodes

Network:

|Machine name |IP address |
|-----|-----|
|namenode1|192.168.17.10|
|datanode1|192.168.17.11|
|datanode2|192.168.17.12|

**steps: configurate the namenode first, then copy two machines (datanode1 & datanode2), finally reset the datanodes**

I. configurate the CentOS7  
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
