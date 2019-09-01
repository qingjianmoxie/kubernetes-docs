# 使用Docker运行JAVA应用



## 安装JDK
```shell
yum -y install java 
[root@dockerserver ~]# java -version
openjdk version "1.8.0_222"
OpenJDK Runtime Environment (build 1.8.0_222-b10)
OpenJDK 64-Bit Server VM (build 25.222-b10, mixed mode)
```

## 安装maven
```shell
wget http://mirror.bit.edu.cn/apache/maven/maven-3/3.6.1/binaries/apache-maven-3.6.1-bin.tar.gz

tar zxf apache-maven-3.6.1-bin.tar.gz  -C /usr/local/
echo "export M2_HOME=/usr/local/apache-maven-3.6.1" >> /etc/profile
echo "export PATH=\$PATH:\$M2_HOME/bin" >> /etc/profile
source /etc/profile

mvn -version
Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-04T15:00:29-04:00)
Maven home: /usr/local/apache-maven-3.6.1
Java version: 1.8.0_222, vendor: Oracle Corporation, runtime: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-514.el7.x86_64", arch: "amd64", family: "unix"
```


## 创建一个Springboot项目

项目地址：https://github.com/gazgeek/springboot-helloworld.git

```shell
yum -y install git wget 
git clone https://github.com/gazgeek/springboot-helloworld.git
cd springboot-helloworld
mvn clean install -DskipTests
ls target/

systemctl stop firewalld
java -jar helloworld-0.0.1-SNAPSHOT.jar   #启动服务
nohup java -jar helloworld-0.0.1-SNAPSHOT.jar &
```


## 安装Docker

```shell
yum -y install docker 
vim /etc/sysconfig/docker
OPTIONS='--selinux-enabled=false --log-driver=journald --signature-verification=false'

systemctl start docker
systemctl status docker
systemctl enable docker

yum -y update
reboot
```

## 创建Dockerfile

```
FROM openjdk:8
MAINTAINER zy

COPY ./helloworld-0.0.1-SNAPSHOT.jar helloworld-0.0.1-SNAPSHOT.jar
EXPOSE 8080
CMD ["java","-jar","helloworld-0.0.1-SNAPSHOT.jar"]
```
## 构建容器镜像

```shell
docker build -t mytest/docker/springbootdemo:latest .

[root@dockerserver ~]# docker build -t mytest/docker/springbootdemo:latest .
Sending build context to Docker daemon 35.98 MB
Step 1/5 : FROM openjdk:8
Trying to pull repository docker.io/library/openjdk ...
8: Pulling from docker.io/library/openjdk
9cc2ad81d40d: Pull complete
e6cb98e32a52: Pull complete
ae1b8d879bad: Pull complete
42cfa3699b05: Pull complete
8d27062ef0ea: Pull complete
9b91647396e3: Pull complete
7498c1055ea3: Pull complete
Digest: sha256:0a16427b68dfaea1ba1d6e462de940580d72549a694dd6cab82ce16ef9edfe66
Status: Downloaded newer image for docker.io/openjdk:8
 ---> 08ded5f856cc
Step 2/5 : MAINTAINER demo
 ---> Running in 546d342383a5
 ---> 90eea0bb5aaf
Removing intermediate container 546d342383a5
Step 3/5 : COPY ./helloworld-0.0.1-SNAPSHOT.jar helloworld-0.0.1-SNAPSHOT.jar
 ---> dcefbeb1107f
Removing intermediate container 51d150a3a8b5
Step 4/5 : EXPOSE 8080
 ---> Running in 5a1c6103b615
 ---> c5c27261fb9d
Removing intermediate container 5a1c6103b615
Step 5/5 : CMD java -jar helloworld-0.0.1-SNAPSHOT.jar
 ---> Running in d0df0ced4030
 ---> 8189ee4c7f86
Removing intermediate container d0df0ced4030
Successfully built 8189ee4c7f86

#查看镜像

[root@dockerserver ~]# docker images
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
mytest/docker/springbootdemo   latest              8189ee4c7f86        34 seconds ago      501 MB
docker.io/openjdk              8                   08ded5f856cc        2 weeks ago         488 MB

#删除镜像

docker rmi 8189ee4c7f86
Untagged: mytest/docker/springbootdemo:latest
Deleted: sha256:8189ee4c7f86766100db7a1df412c797d460eb18861913f352befdbbc25df859
Deleted: sha256:c5c27261fb9ded086e6be8e6782079459f38301e62203336ad0b18f982e74484
Deleted: sha256:dcefbeb1107f20b706668a36931237e1540de5eb3aca7067cf69751a31569e8e
Deleted: sha256:217140c9719bd7120916fc97a5e171d33a64458ab8bfe106accc793f51562a16
Deleted: sha256:90eea0bb5aafdc2462e925dd0a8f9615f03cf6c2898c48b04ff4dad93a80c4ac
```
## 运行容器镜像

```shell
docker run -itd -p 8080:8080  mytest/docker/springbootdemo:latest

docker run -itd mytest/docker/springbootdemo:latest

8c2b9a3b545c021a3f8d06d96e36d55f607e75afdfcffa251e68ed3975c70553
docker ps

CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS              PORTS               NAMES
8c2b9a3b545c        mytest/docker/springbootdemo:latest   "java -jar hellowo..."   7 seconds ago       Up 6 seconds        8080/tcp            wizardly_brattain

docker ps -a
CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS              PORTS               NAMES
8c2b9a3b545c        mytest/docker/springbootdemo:latest   "java -jar hellowo..."   17 seconds ago      Up 16 seconds       8080/tcp            wizardly_brattain

#链接容器
docker exec -it 8c2b9a3b545c /bin/bash
root@8c2b9a3b545c:/# ps
   PID TTY          TIME CMD
    23 ?        00:00:00 bash
    28 ?        00:00:00 ps
root@8c2b9a3b545c:/# ps axu
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1 19.8  3.8 2943196 149588 ?      Ssl+ 03:10   0:16 java -jar helloworld-0.0.1-SNAPSHOT.jar
root         23  0.5  0.0  19936  2220 ?        Ss   03:11   0:00 /bin/bash
root         29  0.0  0.0  38372  1644 ?        R+   03:11   0:00 ps axu
root@8c2b9a3b545c:/# curl 127.0.0.1:8080
Hello from GazGeek!root@8c2b9a3b545c:/#
```
## 停止和删除容器

```
docker stop 
docker rm

[root@dockerserver ~]# docker rm $(docker ps -a -q)
e976498656af
2330b6a8875e
7ace91b113ae
f1adee2c6f37
d5c37fb8ecf6
ebaf4fc4c263
cd2b979d25aa
3d5b0616634e
Error response from daemon: You cannot remove a running container 8c2b9a3b545c021a3f8d06d96e36d55f607e75afdfcffa251e68ed3975c70553. Stop the container before attempting removal or use -f
[root@dockerserver ~]# docker ps
CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS              PORTS               NAMES
8c2b9a3b545c        mytest/docker/springbootdemo:latest   "java -jar hellowo..."   5 minutes ago       Up 5 minutes        8080/tcp            wizardly_brattain
[root@dockerserver ~]# docker ps  -a
CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS              PORTS               NAMES
8c2b9a3b545c        mytest/docker/springbootdemo:latest   "java -jar hellowo..."   5 minutes ago       Up 5 minutes        8080/tcp            wizardly_brattain
[root@dockerserver ~]# docker rm $(docker ps -a -q)
[root@dockerserver ~]# docker rm $(docker ps -a -q) -f
8c2b9a3b545c