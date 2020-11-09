# Installation Guide of OF20 by Kelsey

## Table of Contents

+ [1. Pre settings](#1-pre-settings)
+ [2. JAVA installation](#2-java-installation)
+ [3. Tibero installation](#3-tibero-installation)
+ [4. JEUS installation](#4-jeus-installation)
+ [5. OF20 installation](#4-jeus-installation)
+ [5. UnixODBC installation](#5-unixodbc-installation)
+ [6. PROSORT installation](#6-prosort-installation)

+ [12. OFGW installation](#1312-ofgw-installation)
+ [13. OFManager installation](#1313-ofmanager-installation)
+ [14. OFMiner installation](#1314-ofminer-installation)

### 1. Pre settings

__a.__ Required Package Installation
```
yum install -y wget
yum install -y  dos2unix
yum install -y  glibc*
yum install -y  glibc.i686 glibc.x86_64
yum install -y *libtermcap*
yum install -y  gcc
yum install -y  gcc-c++
yum install -y libncurses*
yum install ncurses*
yum update
```

- Packages for running tibero
```
yum install libaio
yum install libnsl
```

- Extra Packages if needed
```
yum install strace
yum install ltrace
yum install gdb 
yum install nano 
yum install vim-enhanced 
yum install git 
yum install htop
```

__b.__ Kernel Parameters Modification 

vi /etc/sysctl.conf  

```bash
kernel.shmall = 2097152
kernel.shmmax = 4294967295
kernel.shmmni = 4096
kernel.sem = 100000 32000 10000 10000
fs.file-max = 65536
net.ipv4.ip_local_port_range = 1024 65000  
net.core.rmem_default=262144
net.core.wmem_default=262144
net.core.rmem_max=262144
net.core.wmem_max=262144
```

* Refresh the kernel parameters.
```bash
/sbin/sysctl -p 
```

__c.__ Firewall setting

```
systemctl disable firewalld

systemctl stop firewalld  
```

__d.__ Prepare licenses from Technet
* Use the correct hostname for downloading license files from Technet website.
* You need to check hostname and the number of cores.


__e.__ Add user as hostname
``` 
groupadd mqm -g 10000
useradd -d /home/of7azure -g mqm -s /bin/bash -m of7azure -u 10001
```

```
groupadd dba -g 10005
useradd -d /home/oftibr -g dba -s /bin/bash -m oftibr -u 10002
```

__f.__ Add information in bash_profile
```
# clear screen
clear

echo ""
echo "**********************************************"
echo "***          ##OF20 DEMO ENV ##            ***"
echo "**********************************************"
echo "***              OF-CS TEAM                  *"
echo "**********************************************"
echo "***  account : of20                        ***"
echo "***  Download binary : NO IMS#             ***"
echo "***  Installed Product :                   ***"
echo "***    - java version  1.8.0_262           ***"
echo "***                                        ***"
echo "***                            2020.11.03  ***"
echo "**********************************************"
echo ""
```

### 2. JAVA installation
```
[of20@of20 ~]$ java -version
openjdk version "1.8.0_262"
OpenJDK Runtime Environment (build 1.8.0_262-b10)
OpenJDK 64-Bit Server VM (build 25.262-b10, mixed mode)
```

### 3. Tibero installation
```bash
tar -xzvf [tibero tar file]
mv license.xml tibero6/license/
```

```bash
vi .bash_profile

##### Tibero #####
export TB_HOME=/home/of20/tibero6
export TB_SID=of20
export PATH=$TB_HOME/bin:$TB_HOME/client/bin:$PATH
export LD_LIBRARY_PATH=$TB_HOME/lib:$TB_HOME/client/lib:$LD_LIBRARY_PATH
export TBCLI_GET_COREDUMP=Y

source ~/.bash_profile
```

```
sh $TB_HOME/config/gen_tip.sh

vi $TB_HOME/config/$TB_SID.tip
```
```bash
DB_NAME=oframe
LISTENER_PORT=8629
CONTROL_FILES="/home/oframe7/tbdata/c1.ctl"
DB_CREATE_FILE_DEST="/home/oframe7/tbdata" # match the directory CONTROL_FILES
#CERTIFICATE_FILE="/home/oframe7/tibero6/config/svr_wallet/oframe.crt"
#PRIVKEY_FILE="/home/oframe7/tibero6/config/svr_wallet/oframe.key"
#WALLET_FILE="/home/oframe7/tibero6/config/svr_wallet/WALLET"
#EVENT_TRACE_MAP="/home/oframe7/tibero6/config/event.map"
MAX_SESSION_COUNT=100
TOTAL_SHM_SIZE=2G
MEMORY_TARGET=3G 
```
```
tbboot nomount 
    
tbsql sys/tibero

SQL> CREATE DATABASE
USER SYS IDENTIFIED BY TIBERO
MAXINSTANCES 8                                            
MAXDATAFILES 4096                                         
CHARACTER SET MSWIN949
LOGFILE   GROUP 1 ('redo001.redo') SIZE 512M,            
          GROUP 2 ('redo002.redo') SIZE 512M,            
          GROUP 3 ('redo003.redo') SIZE 512M,            
          GROUP 4 ('redo004.redo') SIZE 512M,            
          GROUP 5 ('redo005.redo') SIZE 512M             
MAXLOGGROUPS 255                                          
MAXLOGMEMBERS 8                                           
NOARCHIVELOG                                              
DATAFILE 'system001.dtf' SIZE 200M autoextend on maxsize 1G
DEFAULT TABLESPACE USR                                    
DATAFILE 'usr001.dtf' SIZE 200M  autoextend on maxsize 1G 
DEFAULT TEMPORARY TABLESPACE TEMP                         
TEMPFILE 'temp001.dtf' SIZE 200M autoextend on maxsize 1G 
UNDO TABLESPACE UNDO0                                     
DATAFILE 'undo001.dtf' SIZE 200M autoextend on maxsize 1G; 
```
```
tbboot
    
sh $TB_HOME/scripts/system.sh 

SYS password : tibero
SYSCAT password : syscat
```

### 4. JEUS installation
```
vi ~/.bash_profile

######JAVA ####
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64

#### JEUS ####
export JEUS_HOME=$HOME/jeus8
export PATH="/home/of20/jeus8/bin:/home/of20/jeus8/lib/system:/home/of20/jeus8/webserver/bin:${PATH}"

source ~/.bash_profile
```

```
mkdir jeus8

cd jeus8

unzip jeus8-b162106.zip 

export JEUS_HOME=$HOME/jeus8

chmod 775 ${JEUS_HOME}/lib/etc/ant/bin/ant

${JEUS_HOME}/lib/etc/ant/bin/ant -f ${JEUS_HOME}/setup/build.xml install
```

```
cp license.bin ../jeus8/license/license
```

- Add jeus server

```
1. Start up DAS
$ startDomainAdminServer -u <user-name> -p <password>
ex) startDomainAdminServer -u jeus -p jeus

2. Use jps command to check DAS
$jps
62936 DomainAdminServerBootstrapper
48116 Jps

3. Connect to jeusadmin
$jeusadmin -u <user-name> -p <password> -p <DAS base port>
ex)jeusadmin -u jeus -p jeus -port 9736

4. Add server2 
[DAS]jeus_domain.adminServer> add-server <SERVER_NAME> -addr <JEUS_IP> -baseport <Server_BasePort> -node <DAS_Nodename> -jvm "-Xmx512m -XX:MaxPermSize=128m"
ex) add-server RteServer -addr 192.168.55.33 -baseport 9636 -node node1 -jvm "-Xmx512m -XX:MaxPermSize=128m"

5. Add listener to server2
[DAS]jeus_domain.adminServer> add-listener -server <SERVER_NAME> -name <LISTENER_NAME> -port <LISTENER_PORT>
ex) add-listener -server RteServer -name http-rtesvr -port 8087

6. Add http listener to server2
[DAS]jeus_domain.adminServer> add-web-listener -name <HTTP_NAME> -server <SERVER_NAME> -slref <LISTENER_NAME> -tmin 10
ex) add-web-listener -name http1 -server RteServer -slref http-rtesvr -tmin 10 -tmax 100

7. Restart JEUS
Check domain.xml if RteServer is successfully added.


[Example]
$ jeusadmin -u administrator -p 1111111 -port 9736
Attempting to connect to 127.0.0.1:9736.
The connection has been established to Domain Administration Server adminServer in the domain jeus_domain.
JEUS7 Administration Tool
To view help, use the 'help' command.

[DAS]jeus_domain.adminServer>add-server server2 -addr 192.168.105.196 -baseport 9636 -node ofLinux64 -jvm "-Xmx512m -XX:MaxPermSize=128m"
Successfully performed the ADD operation for server (server2).
Check the results using "list-servers or add-server"

[DAS]qa_domain.adminServer>add-listener -server server2 -name http-server2 -port 8087
Executed successfully, but some configurations were not applied dynamically. It might be necessary to restart the server.
Check the result using 'list-server-listeners -server server2 -name http-server2.

[DAS]qa_domain.adminServer>add-web-listener -name http2 -server server2 -slref http-server2 -tmin 10
Successfully changed only the XML.
Restart the server to apply the changes.
For detailed web connection information, use the 'show-web-engine-configuration -cn' command.

stopServer -host 192.168.55.33:9736 -u jeus -p jeus

startDomainAdminServer -u jeus -p jeus

startManagedServer -domain domain1 -server RteServer -u jeus -p jeus -dasurl ###.###.##.##:9736
```

```
# JEUS alias
alias dsboot='startDomainAdminServer -domain domain1 -u jeus -p jeus'
alias msboot='startManagedServer -domain domain1 -server RteServer -u jeus -p jeus'
alias msdown='stopServer -u jeus -p jeus -host 192.168.96.195:9636' -> check port number
alias dsdown='stopServer -u jeus -p jeus -host 192.168.96.195:9736' -> check port number
```

### 5. OF20 installation

- **DAS should be up and the server should be down before installing OF20.**

```
vi of20.install.properties
```

```
################################################################################
#           Configuration Sample for OpenFrame 20 Installation                 #
#                                                                              #
#   File : of20.install.properties.sample                                      #
#   Date : 20200929                                                            #
#   Version : OF20_INITIAL                                                     #
#                                                                              #
################################################################################

########### for env ##############
PROOBJECT_HOME=/home/of20/OpenFrame20

######## for deply #########
JEUS_HOME=/home/of20/jeus8
JEUS_DAS_IP=192.168.55.33
JEUS_DAS_PORT=9736
JEUS_USER=jeus
JEUS_PW=jeus
SERVER_NAME=RteServer
DOMAIN_NAME=domain1

####### for db  #######
TIBERO_USERNAME=tibero
TIBERO_PASSWORD=tmax
TIBERO_DBNAME=of20
TIBERO_IP=192.168.55.33
TIBERO_PORT=8629

###### for po config ######
HOST_NAME=of20
PO_LISTENER=http-rtesvr
PO_PORT=6677
FILE_PORT=6666
```

```
./OpenFrame_20_Generic.bin -f of20.install.properties
```

Check bash_profile after installing OF20.

The contents below should be automatically added.

```
# New environment setting added by OpenFrame_20 on Fri Nov 06 11:19:05 KST 2020 1.
# The unmodified version of this file is saved in /home/of20/.bash_profile1940443623.
# Do NOT modify these lines; they are used to uninstall.
PROOBJECT_HOME=/home/of20/OpenFrame20
export PROOBJECT_HOME
# End comments by InstallAnywhere on Fri Nov 06 11:19:05 KST 2020 1.
```


### 5. Uninstall OF20

- **DAS should be up before uninstalling OF20.**

***NAT 설정 꺼줘야 함 ***

*** 설치후 msdown , msboot, 재기동!***


cd
