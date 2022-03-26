## JDK安装

- 去官网下载JDK的压缩包，上传到服务器上面

```sh
[root@ls-usCoaSsG java]# pwd
/home/java
[root@ls-usCoaSsG java]# ll
total 45344
-rw-r--r-- 1 root root 46432256 Feb 12 17:00 jdk-8u321-linux-x64.tar.gz
```

- 解压压缩包

```sh
tar -zxvf jdk-8u321-linux-x64.tar.gz
```

- 编辑配置文件配置环境变量：`vim /etc/profile`

新增以下内容，注意将文件路径改为自己对应的Java路径

```sh
[root@ls-usCoaSsG jdk1.8.0_321]# pwd
/home/java/jdk1.8.0_321
```

```properties
set java environment
JAVA_HOME=/home/java/jdk1.8.0_321   
JRE_HOME=/home/java/jdk1.8.0_321/jre     
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```

使配置生效：`source /etc/profile`

- JDK验证：`java -version`

```sh
[root@ls-usCoaSsG jdk1.8.0_321]# java -version
java version "1.8.0_321"
Java(TM) SE Runtime Environment (build 1.8.0_321-b07)
Java HotSpot(TM) 64-Bit Server VM (build 25.321-b07, mixed mode)
```

