# 目录
* [安装JDK](#1)
* [安装Tomcat](#2)
* [tomcat8以上管理页面提示403 Access Denied问题](#3)

###### 1
# 安装JDK

卸载系统的JDK
先查看 rpm -qa | grep java
显示如下信息：

java-1.4.2-gcj-compat-1.4.2.0-40jpp.115
java-1.6.0-openjdk-1.6.0.0-1.7.b09.el5

卸载：
```
rpm -e --nodeps java-1.4.2-gcj-compat-1.4.2.0-40jpp.115
rpm -e --nodeps java-1.6.0-openjdk-1.6.0.0-1.7.b09.el5
```
安装sun公司的jdk1.7

查看JDK软件包列表
```
yum -y list java*
或者
yum search java | grep -i --color JDK
```
安装
```
yum -y install java-1.7.0-openjdk java-1.7.0-openjdk-devel.x86_64
```
通过yum默认安装的路径为
```
/usr/lib/jvm
```

配置环境变量 

在/etc/profile文件中加入下面内容配置环境变量
```
########## jdk  environment ######################
export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.181-2.6.14.8.el7_5.x86_64
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
########## jdk  environment ######################
```
保存关闭,执行如下命令使设置生效
```
source /etc/profile
```
查看变量是否生效
```
echo $JAVA_HOME && echo $CLASSPATH
```

查看Java版本信息
```
java -version
```
------------------------------------------------------------------------------------------------
###### 2
# 安装多个Tomcat7

下载、安装 tomcat

地址https://tomcat.apache.org/download-80.cgi

把下载的apache-tomcat-8.5.31.tar.gz文件解压  
```
tar -zxvf apache-tomcat-8.5.31.tar.gz
```
修改tomcat文件夹名为tomcat-8080
```
mv apache-tomcat-8.5.31 tomcat-8080
```
复制tomcat-8080：  
```
cp -r tomcat-8080 tomcat-8082
```

修改配置文件

在/etc/profile文件中加入下面内容配置环境变量

第一个tomcat
```
########## tomcat 1###########
CATALINA_BASE=/home/tomcat-8080
CATALINA_HOME=/home/tomcat-8080
TOMCAT_HOME=/home/tomcat-8080
export CATALINA_BASE CATALINA_HOME TOMCAT_HOME
########## tomcat 1############
```

第二个tomcat
```
######### tomcat 2 ##########
CATALINA_2_BASE=/home/tomcat-8082
CATALINA_2_HOME=/home/tomcat-8082
TOMCAT_2_HOME=/home/tomcat-8082
export CATALINA_2_BASE CATALINA_2_HOME TOMCAT_2_HOME
########## tomcat 2##########
```
source命令也称为“点命令”，也就是一个点符号（.）。source命令通常用于重新执行刚修改的初始化文件，使之立即生效
用法： 
```
source /etc/profile 
或者
. /etc/profile
```

修改第二个tomcat文件

进入tomcat-8082的bin目录，修改startup.sh和shutdown.sh 两个文件，都添加如下内容
```
######### tomcat 2 ##########
export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.181-2.6.14.8.el7_5.x86_64
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=$JAVA_HOME/lib
export CATALINA_HOME=$CATALINA_2_HOME
export CATALINA_BASE=$CATALINA_2_BASE
######### tomcat 2 ##########
```

修改第二个tomcat端口,第一个不变

进入/tomcat-8082/conf中修改server.xml
修改后示例如下：
```
<Server port="9005" shutdown="SHUTDOWN">　#关闭端口：8005->9005

<Connector port="8181" maxHttpHeaderSize="8192"　#Web端口：8080->8181
maxThreads="150" minSpareThreads="25" maxSpareThreads="75"
enableLookups="false" redirectPort="8443" acceptCount="100"
connectionTimeout="20000" disableUploadTimeout="true" />

<Connector port="9009"  #监听端口：8009->9009
enableLookups="false" redirectPort="8443" protocol="AJP/1.3" />
```
*************************************************************
###### 3
# tomcat8以上管理页面提示403 Access Denied问题
## 修改conf/tomcat-users.xml
```shell
vi conf/tomcat-users.xml
```
### 按shift+g跳到末尾,在</tomcat-users>前添加
```xml
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <role rolename="admin-gui"/>
  <role rolename="admin-script"/>
  <user username="tomcat" password="密码" roles="manager-gui,manager-script,admin-gui,admin-script"/>
```

### 打开webapps下的host-manager和manager，在META-INF里面都有context.xml
```shell
vi webapps/manager/META-INF/context.xml
vi webapps/host-manager/META-INF/context.xml
```
### 修改<Context antiResourceLocking="false" privileged="true" >节点
#### 这段代码的作用是限制来访IP的，127.d+.d+.d+|::1|0:0:0:0:0:0:0:1，是正则表达式，表示IPv4和IPv6的本机环回地址
```xml
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|\d+\.\d+\.\d+\.\d+" />
```


## Tomcat热部署
```diff
+替换WEB-INF/lib目录中的jar文件或WEB-INF/classes目录中的class文件时，reloadable="true"会让修改生效（但代价不小），该选项适合调试。
<Context docBase="xxx" path="/xxx" reloadable="true"/> 
+在webapps目录中增加新的目录、war文件、修改WEB-INF/web.xml，autoDeploy="true"会新建或重新部署应用，该选项方便部署。
<Context docBase="xxx" path="/xxx" autoDeploy="true"/> 
```


# [返回顶部](#readme)
