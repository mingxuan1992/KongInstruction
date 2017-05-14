# KongInstruction
//Instruction and details of Kong Implementation


//install the package from getkong.org 
//1.download to local
//2.push to git
//3.in ec2, git clone http://your/github/this/repository

//install kong command
sudo yum install kong-0.10.1.aws.rpm --nogpgcheck

//due to kong need cassandra3.0+, and cassandra3.0+ need java1.8 
//we need to change java version fisrt
//java -version 
//java version "1.7.0_131" 
//OpenJDK Runtime Environment (amzn-2.6.9.0.71.amzn1-x86_64 u131-b00)
//OpenJDK 64-Bit Server VM (build 24.131-b00, mixed mode)

//Install Java 8
sudo yum install java-1.8.0-openjdk-devel
//** Select Java 8 Option for: **
sudo /usr/sbin/alternatives --config java
//choose 1.8
sudo /usr/sbin/alternatives --config javac
//NOTE:  
//JAVA_HOME = /usr/lib/jvm/java-1.8.0-openjdk.x86_64

export JAVA_HOME=/opt/jdk1.8.0_121
export JRE_HOME=/opt/jdk1.8.0_121/jre
export PATH=$PATH:/opt/jdk1.8.0_121/bin:/opt/jdk1.8.0_121/jre/bin

mkdir ~/cassandra
cd ~/cassandra

//choose Option2
curl -OL http://apache.claz.org/cassandra/redhat/30x/cassandra-3.0.13-1.noarch.rpm 

rpm -i cassandra-3.0.13-1.noarch.rpm
sudo service cassandra start 
//Starting Cassandra: OK
service cassandra status 
//cassandra (pid  3707) is running... 


cd /usr/bin
cqlsh
//check cassandra is working, quit with "quit"

//configure the kong 
cd /etc/kong
sudo chmod 777 kong.conf.default 
vi kong.conf.default
// change database to cassandra

kong start --conf /etc/kong/kong.conf.default
kong stop --conf /etc/kong/kong.conf.default   


//Kong started
//kong works well now
kong check /etc/kong/kong.conf 
curl 127.0.0.1:8001


//example for 
curl -i -X POST \
  --url http://localhost:8001/apis/ \
  --data 'name=jintengtest' \
  --data 'hosts=jinteng' \
  --data 'upstream_url=http://ec2-54-219-163-27.us-west-1.compute.amazonaws.com:3000'  
curl -i -X DELETE \
  --url http://localhost:8001/apis/{nodename}
 

-------------------------------------------------------
!!!!!!Above is my first time attempt and 1 node cassandra database
//for 3 node cassandra database
I followed the 3 node cassandra toturial with ubuntu environment

1. run a ubuntu ec2
2. upload your kong install package
3. install kong
4. change your java version
5. install cassandra 
6. configure your canssandra with your 3 nodes' privite ip address.
7. start 3 cassandra
8. start kong 
9. post kong rules
 this is the link for 3 node cassandra tutorial
https://www.digitalocean.com/community/tutorials/how-to-run-a-multi-node-cluster-database-with-cassandra-on-ubuntu-14-04

