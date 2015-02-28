# kafka-storm-vm

Assumes VirtualBox and Vagrant are already installed.

###Create the VM image:
```shell
vagrant box add --provider virtualbox centos6-compatible http://puppet-vagrant-boxes.puppetlabs.com/centos-65-x64-virtualbox-puppet.box

vagrant init centos6-compatible

vagrant up
vagrant ssh
```

###Java Install: 
from http://tecadmin.net/steps-to-install-java-on-centos-5-6-or-rhel-5-6/
In your downloads directory

```shell
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/7u75-b13/jdk-7u75-linux-x64.tar.gz"

tar xzf jdk-7u75-linux-x64.tar.gz

sudo mv jdk1.7.0_75/ /opt/jdk1.7.0_75

cd /opt/jdk1.7.0_75/
sudo alternatives --install /usr/bin/java java /opt/jdk1.7.0_75/bin/java 2
sudo alternatives --config java
sudo alternatives --install /usr/bin/jar jar /opt/jdk1.7.0_75/bin/jar 2
sudo alternatives --install /usr/bin/javac javac /opt/jdk1.7.0_75/bin/javac 2
sudo alternatives --set jar /opt/jdk1.7.0_75/bin/jar
sudo alternatives --set javac /opt/jdk1.7.0_75/bin/javac
```

###Maven install: 
from http://www.unixmen.com/install-apache-ant-maven-tomcat-centos-76-5/
In your downloads directory

```shell
wget http://www.eng.lsu.edu/mirrors/apache/maven/maven-3/3.2.5/binaries/apache-maven-3.2.5-bin.zip
unzip apache-maven-3.2.5-bin.zip
sudo mv apache-maven-3.2.5/ /opt/maven
sudo ln -s /opt/maven/bin/mvn /usr/bin/mvn

sudo nano /etc/profile.d/maven.sh
```
Add the following contents:
```shell
#!/bin/bash
MAVEN_HOME=/opt/maven
PATH=$MAVEN_HOME/bin:$PATH
export PATH MAVEN_HOME
export CLASSPATH=.
```

Save and close the file. Make it executable using the following command.
```shell
sudo chmod +x /etc/profile.d/maven.sh
```
Then, set the environment variables permanently by running the following command:
```shell
source /etc/profile.d/maven.sh
```
Log out or reboot your system.
Now, check the ant version using command:
```shell
mvn -version
```

###Install Supervisor
```shell
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
sudo rpm -Uvh epel-release-6*.rpm
sudo yum install supervisor
sudo chkconfig supervisord on
```
###Install ZooKeeper: from http://jansipke.nl/installing-a-storm-cluster-on-centos-hosts/
In your downloads directory

```shell
wget http://www.eng.lsu.edu/mirrors/apache/zookeeper/stable/zookeeper-3.4.6.tar.gz

gunzip -c zookeeper-3.4.6.tar.gz  | tar xvf -

sudo mv zookeeper-3.4.6/ /opt/zookeeper-3.4.6

cp /opt/zookeeper-3.4.6/conf/zoo_sample.cfg /opt/zookeeper-3.4.6/conf/zoo.cfg
```
edit conf/zoo.cfg
```shell
dataDir=/var/zookeeper
```

edit /etc/supervisord.conf, add:
```shell
[program:zookeeper]
command=/opt/zookeeper-3.4.6/bin/zkServer.sh start-foreground
autostart=true
autorestart=true
startsecs=1
startretries=999
redirect_stderr=false
stdout_logfile=/var/log/zookeeper-out
stdout_logfile_maxbytes=10MB
stdout_logfile_backups=10
stdout_events_enabled=true
stderr_logfile=/var/log/zookeeper-err
stderr_logfile_maxbytes=100MB
stderr_logfile_backups=10
stderr_events_enabled=true
```

```shell
sudo chkconfig supervisord on
sudo service supervisord start
```

check it is running
```shell
sudo supervisorctl
```

###Install ZeroMq and JZMQ
```shell
sudo yum install gcc gcc-c++ libuuid-devel
sudo yum install libtool
sudo yum install zeromq zeromq-devel
```

edit file ~/.bashrc and add,
```shell
export JAVA_HOME=/opt/jdk1.7.0_75
export JAVAH=/opt/jdk1.7.0_75/bin/javah
export JAR=/opt/jdk1.7.0_75/bin/jar
```

```shell
git clone https://github.com/zeromq/jzmq.git
cd jzmq/
./autogen.sh 
./configure 
make
sudo make install
```

###Create a storm group and user
```shell
sudo groupadd -g 53001 storm
sudo useradd -u 53001 -g 53001 -d /home/storm/ -s /bin/bash storm -c "Storm service account"
sudo chmod 700 /home/storm/
sudo chage -I -1 -E -1 -m -1 -M -1 -W -1 -E -1 storm
```

###Install Storm:
In your downloads directory

```shell
sudo wget http://apache.cs.utah.edu/storm/apache-storm-0.9.2-incubating/apache-storm-0.9.2-incubating.tar.gz
sudo tar xvzf apache-storm-0.9.2-incubating.tar.gz
sudo mv apache-storm-0.9.2-incubating/ /opt/apache-storm-0.9.2-incubating
sudo chmod +x /opt/apache-storm-0.9.2-incubating/bin/storm
sudo ln -s /opt/apache-storm-0.9.2-incubating/bin/storm /usr/bin/storm

sudo mkdir /var/log/storm
sudo chown storm:storm /var/log/storm

sudo mkdir /etc/storm
sudo chown storm:storm /etc/storm
sudo mv /opt/apache-storm-0.9.2-incubating/conf/storm.yaml /etc/storm/
sudo ln -s /etc/storm/storm.yaml /opt/apache-storm-0.9.2-incubating/conf/storm.yaml
```

edit /etc/supervisord.conf, add:
```shell
[program:storm-nimbus]
command=storm nimbus
directory=/home/storm
autorestart=true
user=storm

[program:storm-supervisor]
command=storm supervisor
directory=/home/storm
autorestart=true
user=storm

[program:storm-ui]
command=storm ui
directory=/home/storm
autorestart=true
user=storm
```

###Install Kafka:

```shell
wget http://apache.petsads.us/kafka/0.8.1.1/kafka_2.9.2-0.8.1.1.tgz
tar -zxf kafka_2.9.2-0.8.1.1.tgz
sudo mv kafka_2.9.2-0.8.1.1/ /opt/kafka_2.9.2-0.8.1.1
```
