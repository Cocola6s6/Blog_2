---
title: 笔记-elk
categories: 好的学习笔记
tags: 运维
banner_img: https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230705112602006.png
---



# Docker 部署服务

需要部署的服务：

* 部署 Elasticsearch
* 部署Logstash
* 部署 Kibana
* 部署Zookeeper
* 部署Kafka



![image-20230705112602006](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230705112602006.png)



### 一、单个部署

【参考】[晖哥的笔记](https://note.youdao.com/ynoteshare/index.html?id=b3fbb1961a85c7511924961053b10938&type=note&_time=1688452121644)



### 二、整体部署

使用 docker-compose 整体部署：

1. 创建 docker-compose.yml
2. 创建脚本 install.sh
3. 部署



【注意】文件的存放地址是 /home/sys_install，需要提前创建。

~~~bash
mkdir /home/sys_install
~~~

【注意】服务启动需要的配置文件地址是 /home/sys_install/config，需要提前准备





#### 1、创建 docker-compose.yml

``` yaml
version: '3.5'
services:
  server:
    image: server:1.0
    container_name: server
    restart: always
    networks:
      - cocola
    environment:
      - logging=classpath:log4j2.xml
    ports:
      - "9091:9091"  

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.0
    container_name: elasticsearch
    restart: always
    environment:
      cluster.name: elasticsearch
      ES_JAVA_OPTS: "-Xms256m -Xmx512m"
      discovery.type: single-node
    ports:
      - 9200:9200
      - 9300:9300
    volumes:
      - /home/sys_install/config/elasticsearch/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - /home/services_data/elasticsearch/data:/usr/share/elasticsearch/data
      - /home/services_data/elasticsearch/plugins:/usr/share/elasticsearch/plugins
    networks:
      - cocola

  kibana:
    image: kibana:7.17.0
    container_name: kibana
    restart: always
    environment:
      ELASTICSEARCH_URL: http://elasticsearch:9200
    depends_on:
      - elasticsearch
    ports:
      - 5601:5601
    networks:
      - cocola

  logstash:
    image: docker.elastic.co/logstash/logstash:7.17.0
    container_name: logstash
    restart: always
    volumes:
      - /home/sys_install/config/logstash/pipeline/:/usr/share/logstash/pipeline/
      - /home/sys_install/config/logstash/config:/usr/share/logstash/config/
    depends_on:
      - elasticsearch
    links:
      - elasticsearch:es
    ports:
      - 5407:5407
      - 9600:9600
    networks:
      - cocola

  kafka:
    #image: bitnami/kafka:2.8.0
    image: wurstmeister/kafka
    container_name: kafka
    restart: always
    ports:
      - 9092:9092
      - 9093:9093
    networks:
      - cocola
    depends_on:
      - zookeeper
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      ALLOW_PLAINTEXT_LISTENER: yes
      
      #KAFKA_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT://kafka:9093
      #KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://110.41.21.178:9092,PLAINTEXT://110.41.21.178:9093

      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: A:PLAINTEXT,B:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: A
      KAFKA_ADVERTISED_LISTENERS: A://kafka:9092,B://110.41.21.178:9093
      KAFKA_LISTENERS: A://kafka:9092,B://kafka:9093
      
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'false'
        
  zookeeper:
    #image: bitnami/zookeeper:3.7.0
    image: wurstmeister/zookeeper
    container_name: zookeeper
    restart: always
    ports:
      - 2181:2181
    networks: 
      - cocola
    environment:
      ALLOW_ANONYMOUS_LOGIN: yes

  nginx:
    image: nginx:latest
    container_name: nginx
    restart: always
    networks: 
      - cocola
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /etc/localtime:/etc/localtime       
      - /home/sys_install/config/nginx/nginx.conf:/etc/nginx/nginx.conf


  jenkins:
    image: jenkins/jenkins:2.332.3
    container_name: jenkins
    restart: always
    networks:
      - cocola
    ports:
      - "9000:8080"
    volumes:
      - /home/services_data/jenkins/jenkins_home:/var/jenkins_home
      - /home/services_data/jenkins/maven/maven:/home/jenkins/maven
      - /home/services_data/jenkins/jdk/jdk:/home/jenkins/jdk


networks:
  cocola:
    external: false
    name: cocola
    ipam:
      config:
        - subnet: 192.168.20.0/24
```





#### 2、创建脚本 install.sh

【作用】为了统一管理端口开关、文件映射、权限授予、服务安装卸载。

~~~ bash
#!/bin/bash
set -e

## 开放端口
function openPort() {
	#systemctl start firewalld
	firewall-cmd --zone=public --add-port=$1/tcp
	firewall-cmd --list-ports
	#systemctl restart firewalld	
}

## 创建docker容器并启动
SYS_INSTALL_DIR=/home/sys_install
SERVICES_DATA_DIR=/home/services_data
DOCKER_LIBRARY_DIR=/home/docker_library
function createContainerAndRun(){
        docker-compose -f $SYS_INSTALL_DIR/docker-compose.yml up -d $1 
}

## 构建docker镜像
function buildAllImages() {
        for file in ${DOCKER_LIBRARY_DIR}/*;
        do
                echo $file
		if [ ! -d $file ]; then
			continue
		fi

                cd $file
                sh build.sh
        done
	cd "$SYS_INSTALL_DIR" 
}

## 准备docker镜像
function prepareAllImages() {
	#loadAllImages
	buildAllImages
}

## 安装server服务
function installServer() {
	createContainerAndRun server
}

## 安装elasticsearch服务
ELA_DATA_DIR=$SERVICES_DATA_DIR/elasticsearch/data
ELA_PLUG_DIR=$SERVICES_DATA_DIR/elasticsearch/plugins
function installEla() {
	if [ ! -d $ELA_DATA_DIR ]; then
                sudo mkdir $ELA_DATA_DIR -p
        fi
	
	if [ ! -d $ELA_PLUG_DIR ]; then
                sudo mkdir $ELA_PLUG_DIR -p
        fi
	chmod +777 $ELA_DATA_DIR
	
	cd "$SYS_INSTALL_DIR"
	openPort 9200
	openPort 9300
	
	createContainerAndRun elasticsearch
}

## 安装kibbna服务
function installKibana() {
        cd "$SYS_INSTALL_DIR"
        openPort 5601       
 
        createContainerAndRun kibana
}

## 安装logstash服务
function installLogstash() {
        cd "$SYS_INSTALL_DIR" 	
        openPort 5407
        openPort 9600

        createContainerAndRun logstash
}

## 安装kafka服务
function installKafka() {
        cd "$SYS_INSTALL_DIR"
	openPort 9092
	openPort 9093

        createContainerAndRun kafka
	sleep 10
	createKafkaTopics
}

## 创建kafka主题
KAFKA_TOPIC_ARRAY=(
	log-topic
)

function createKafkaTopics() {
	echo "开始创建kafka主题"
	for(( i=0;i<${#KAFKA_TOPIC_ARRAY[@]};i++)) do
		sudo docker exec -it kafka /bin/bash -c "/opt/kafka/bin/kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic ${KAFKA_TOPIC_ARRAY[$i]}"
	done;

	echo "结束创建kafka主题"
}

## 安装nginx服务
function installNginx() {
        cd "$SYS_INSTALL_DIR"
        sh /home/sys_install/openPort.sh 80
        sh /home/sys_install/openPort.sh 443

        createContainerAndRun nginx
}

## 安装jenkins服务
JENKINS_DATA_DIR=$SERVICES_DATA_DIR/jenkins
MAVEN_DATA_DIR=$SERVICES_DATA_DIR/jenkins/maven
JDK_DATA_DIR=$SERVICES_DATA_DIR/jenkins/jdk
function installJenkins() {
	if [ ! -d $JENKINS_DATA_DIR ]; then
		sudo mkdir $JENKINS_DATA_DIR -p
	fi
	cd $JENKINS_DATA_DIR

	if [ ! -d $MAVEN_DATA_DIR ]; then
		sudo mkdir $MAVEN_DATA_DIR

		if [ ! -f "apache-maven-3.6.3-bin.tar.gz" ]; then
                        echo "===========>download maven online"
                        cd $MAVEN_DATA_DIR
			wget https://mirrors.aliyun.com/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
                else 
                        echo "===========>cp maven"
                        cp apache-maven-3.6.3-bin.tar.gz $MAVEN_DATA_DIR
                fi

		cd $MAVEN_DATA_DIR
                tar -zxvf apache-maven-3.6.3-bin.tar.gz
		mv apache-maven-3.6.3 maven
		cd $JENKINS_DATA_DIR
        fi
	
	if [ ! -d $JDK_DATA_DIR ]; then
		sudo mkdir $JDK_DATA_DIR

		if [ ! -f "jdk-8u371-linux-x64.tar.gz" ]; then
			echo "===========>download jdk online"
			cd $JDK_DATA_DIR
			#wget https://download.oracle.com/otn/java/jdk/8u371-b11/ce59cff5c23f4e2eaf4e778a117d4c5b/jdk-8u371-linux-x64.tar.gz
		else
			echo "===========>cp jdk"
			cp jdk-8u371-linux-x64.tar.gz $JDK_DATA_DIR
        	fi

		cd $JDK_DATA_DIR
		tar -zxvf jdk-8u371-linux-x64.tar.gz
		mv jdk1.8.0_371 jdk
		cd $JENKINS_DATA_DIR
	fi

        chown -R 1000:1000 $SERVICES_DATA_DIR/jenkins
	chmod +777 $SERVICES_DATA_DIR/jenkins/*

	sh /home/sys_install/openPort.sh 9000
	createContainerAndRun jenkins
}

## 安装全部服务
function installAll() {
	echo $(date +'%Y-%m-%d %H:%M:%S'):"开始安装系统"
	prepareAllImages
	installEla
	installKibana
	installLogstash
	installKafka
	#installNginx
	installServer
	echo $(date +'%Y-%m-%d %H:%M:%S'):"完成系统安装"
}


## 停止并删除容器
function stopAndRemoveContainer() {
        docker-compose stop $1
        docker-compose rm -f $1
}

## 取消安装server服务 
function uninstallServer() {
	stopAndRemoveContainer server
}

## 取消安装ela服务
function uninstallEla() {
	rm -rf $ELA_DATA_DIR
	rm -rf $ELA_PLUG_DIR
	stopAndRemoveContainer elasticsearch
}

## 取消安装kibana服务
function uninstallKibana() {
	stopAndRemoveContainer kibana
}

## 取消安装logstash服务
function uninstallLogstash() {
        stopAndRemoveContainer logstash
}

## delete kafka主题
function deleteKafkaTopics() {
        echo "开始delete kafka主题"
        for(( i=0;i<${#KAFKA_TOPIC_ARRAY[@]};i++)) do
                sudo docker exec -it kafka /bin/bash -c "/opt/kafka/bin/kafka-topics.sh --delete --zookeeper zookeeper:2181 --topic ${KAFKA_TOPIC_ARRAY[$i]}"
        done;
             
        echo "结束delete kafka主题"
}

## 取消安装kafka服务
function uninstallKafka() {
	#deleteKafkaTopics
	stopAndRemoveContainer zookeeper
        stopAndRemoveContainer kafka
}

## 取消安装nginx服务
function uninstallNginx() {
        stopAndRemoveContainer nginx
}

## 取消安装jenkins服务
function uninstallJenkins() {
	rm -rf $MAVEN_DATA_DIR
	rm -rf $JDK_DATA_DIR
	stopAndRemoveContainer jenkins
}

## 安装单个服务
function installSingle() {
        case $1 in
		elasticsearch)
		        installEla
			;;
		kibana)
			installKibana
			;;
	  	logstash)
      		        installLogstash
      		        ;;
		server)
			installServer
			;;
		kafka)
                        installKafka
                        ;;
		nginx)
                        installNginx
                        ;;
		jenkins)
                        installJenkins
			;;
		topic)
			createKafkaTopics
			;;
                *)
                        ;;
        esac
}

## 取消安装单个服务
function uninstallSingle() {
	case $1 in
		elasticsearch)
			uninstallEla
			;;
		kibana)
			uninstallKibana
			;;
	  	logstash)
      		        uninstallLogstash
      		        ;;
		server)
			uninstallServer
			;;
                kafka)
                        uninstallKafka
                        ;;
                nginx)
                        uninstallNginx
			;;
		jenkins) 
                        uninstallJenkins
                        ;;
		topic)
			deleteKafkaTopics
			;;
                *)
                        ;;
        esac
}

## 重新安装单个服务
function reinstallSingle() {
        uninstallSingle $1
        installSingle $1
}

## 选择功能
case $1 in
	all)
		installAll
		;;
 	i)
    	        installSingle "$2"
		;;
	u)
		uninstallSingle "$2"
		;;
	ri)
		reinstallSingle "$2"
		;;
	port)
		openPort "$2"
		;;
	*)
		echo "Usage: $0 [all|i]"
		;;

esac
~~~



#### 3、部署

执行脚本命令：

```bash
sh install.sh all
```



# Jenkins 自动化构建部署

* 部署 Jenkins
* 配置 Jenkins
* 开始构建部署



### 一、部署 Jenkins

* 下载 jdk 和 maven【可忽略】
* 部署 Jenkins



#### 1、下载 jdk 和 maven

这里是为了 Jenkins 能构建 java 项目，如果没有 java 服务需要构建的话，【忽略】。



#### 2、部署 Jenkins

执行脚本命令：

~~~bash
sh install.sh i jenkins
~~~



如果是在其它服务器上，就将 docker-compose.yml、install.sh 里面对应的 Jenkins，拷贝到其它服务器上执行即可。



### 二、配置 Jenkins

网页输入 http://ip:9000/ ，进入 Jenkins UI



#### 1、进入【系统管理->插件管理】

安装插件 【Publish Over SSH】、【Maven Integration plugin】。



#### 2、进入【系统管理->系统配置->Publish over SSH】

需要配置的字段：

* Name
* Hostname
* Username
* Proxy password



![image-20230705121448661](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230705121448661.png)





#### 3、进入【新建任务】



![image-20230705121137685](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230705121137685.png)



#### 3、进入任务【配置->构建触发器】

1. 选择 Send file....over SSH
2. 选择 Name，要连接的服务器
3. 填入 Exec Command，要在服务器上执行的命令



![image-20230705121805235](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230705121805235.png)



![image-20230705121945767](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230705121945767.png)



### 三、开始构建部署

开始部署



# 遇到问题

#### 1、productor 连接 kafka 时，LEADER_NOT_AVAILABLE

* topic 正在被删除，导致当前分区的领导者（Leader）不可用。
* 客户端连接还在 topic 就服务完成删除。停止、删除然后重新部署 kafka，sh install.sh ri kafka



#### 2、productor 连接 kafka 时，TimeoutException

* kafka listener 配置有问题，参考：https://www.confluent.io/blog/kafka-listeners-explained/
* KAFKA_LISTENERS 配置 Broker 监听连接地址，KAFKA_ADVERTISED_LISTENERS 配置是为了告诉客户端 Broker 监听的地址

~~~bash
##修改前
#KAFKA_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT://kafka:9093
#KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://110.41.21.178:9092,PLAINTEXT://110.41.21.178:9093

##修改后
KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: A:PLAINTEXT,B:PLAINTEXT
KAFKA_INTER_BROKER_LISTENER_NAME: A
KAFKA_ADVERTISED_LISTENERS: A://kafka:9092,B://110.41.21.178:9093
KAFKA_LISTENERS: A://kafka:9092,B://kafka:9093
~~~



【解释】

KAFKA_ADVERTISED_LISTENERS: A://kafka:9092,B://110.41.21.178:9093

* 表示配置了两个监听地址，kafka:9092 是给 docker 网络内的客户端连接的，110.41.21.178:9093 是给宿主机网络内的客户端连接的。
* 【A://kafka:9092,B://kafka:9093】会连接超时，因为现在只有容器内的客户端能访问。
* 为了方便，可以配置成【A://110.41.21.178:9092,B://110.41.21.178:9093】。容器内的客户端访问 110.41.21.178:9092 也是会转发到 kafka:9092 的。



KAFKA_LISTENERS: A://kafka:9092,B://kafka:9093

* 表示配置了两个 Broker 地址，通过监听地址进来后会走到对应的 Broker 地址进行连接。
* 为了方便，完全可以配置成为【 A://:9092,B://:9093】。表示监听所有 ip 下的 9092 和 9093 地址。





# Kafka tools 的使用

下载地址： https://www.kafkatool.com/download.html



需要填写的字段：

* Zookeeper 的地址：ip:port
* kafka broker 的地址：ip:port

【注意】kafka broker 的地址是【KAFKA_ADVERTISED_LISTENERS】配置的地址。这里填写的是【kafka:9092,kafka:9093】，因为我的本地 host 配置了映射地址【110.41.21.178 kafka】。这里完全可以直接填写【110.41.21.178:9092,110.41.21.178:9093】

![image-20230705144911594](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230705144911594.png)



![image-20230705144943633](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230705144943633.png)

