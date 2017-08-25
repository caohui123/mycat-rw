����mysql���Ӹ���,���Ľ�����δ`mycat�м��`,����`mycat`����`��д����`.

�����ļ��Լ��ĵ���ַ:

ϵͳ����
=======
* docker 1.12.3
* mysql5.7.17
* mycat1.6

Ҫ��˵��
======
* ����ƪ���µ���ϸ����
* ��¶`mysql` `mycat`�˿ں�,�������
* ����ֱ�Ӵ�`docker-compose.yml`��ʼ

Begin
======

### docker-compose.yml�ļ�

Ϊ�˿���������,�ۻ���һ����������
```
version: '2'
services:
  m1:
    build: ./master
    container_name: m1
    volumes:
      - /home/ssab/config/mysql-master/:/etc/mysql/:ro
      - /etc/localtime:/etc/localtime:ro
      - /home/ssab/config/hosts:/etc/hosts:ro
    ports:
      - "3309:3306" #��¶mysql�Ķ˿�
    networks:
      mysql:
        ipv4_address: 172.18.0.2
    ulimits:
      nproc: 65535
    hostname: m1
    mem_limit: 1024m
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: m1test
  s1:
      build: ./s1
      container_name: s1
      volumes:
        - /home/ssab/config/mysql-s1/:/etc/mysql/:ro
        - /etc/localtime:/etc/localtime:ro
        - /home/ssab/config/hosts:/etc/hosts:ro
      ports:
        - "3307:3306"
      networks:
        mysql:
          ipv4_address: 172.18.0.3
      links:
        - m1
      ulimits:
        nproc: 65535
      hostname: s1
      mem_limit: 1024m
      restart: always
      environment:
        MYSQL_ROOT_PASSWORD: s1test
  s2:
    build: ./s2
    container_name: s2
    volumes:
      - /home/ssab/config/mysql-s2/:/etc/mysql/:ro
      - /etc/localtime:/etc/localtime:ro
      - /home/ssab/config/hosts:/etc/hosts:ro
    ports:
      - "3308:3306"
    links:
      - m1
    networks:
      mysql:
        ipv4_address: 172.18.0.4
    ulimits:
      nproc: 65535
    hostname: s2
    mem_limit: 1024m
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: s2test
  mycat: # ����mycat
    build: ./mycat
    container_name: mycat
    volumes:
      - /home/ssab/config/mycat/:/mycat/conf/:ro # mycat�����ļ�
      - /home/ssab/config/mycat-logs/:/mycat/logs/:rw # mycat��־�ļ�
      - /etc/localtime:/etc/localtime:ro
      - /home/ssab/config/hosts:/etc/hosts:ro
    ports:
      - "8066:8066" # ��¶mycat����˿�
      - "9066:9066" # ��¶mycat����˿�
    links: # mycat��������m1 s1 s2
      - m1
      - s1
      - s2
    networks:
      mysql:
        ipv4_address: 172.18.0.5
    ulimits:
      nproc: 65535
    hostname: mycat
    mem_limit: 1024m
    restart: always
networks:
  mysql:
    driver: bridge
    ipam:
      driver: default
      config:
      - subnet: 172.18.0.0/24
        gateway: 172.18.0.1
```

### mycat ����

����ֻ��˵һ���ɹ����е�����,������ϸ�����ù������Լ��ο�mycatȨ��ָ��.

#### schema.xml����

```
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

	<schema name="mall" checkSQLschema="false" sqlMaxLimit="100" dataNode="mallDN">

	</schema>

	<dataNode name="mallDN" dataHost="mallDH" database="mall">

	</dataNode>

	<dataHost name="mallDH" maxCon="1000" minCon="10" balance="1"
			  writeType="0" dbType="mysql" dbDriver="native" switchType="-1" slaveThreshold="100">
		<heartbeat>select user()</heartbeat>
		<writeHost host="m1" url="172.18.0.2:3306" user="root" password="m1test">
			<readHost host="s1" url="172.18.0.3:3306" user="root" password="s1test" />
			<readHost host="s2" url="172.18.0.4:3306" user="root" password="s2test" />
		</writeHost>

	</dataHost>

</mycat:schema>
```

#### server.xml����

```
<?xml version="1.0" encoding="UTF-8"?>
<!-- - - Licensed under the Apache License, Version 2.0 (the "License");
	- you may not use this file except in compliance with the License. - You
	may obtain a copy of the License at - - http://www.apache.org/licenses/LICENSE-2.0
	- - Unless required by applicable law or agreed to in writing, software -
	distributed under the License is distributed on an "AS IS" BASIS, - WITHOUT
	WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. - See the
	License for the specific language governing permissions and - limitations
	under the License. -->
<!DOCTYPE mycat:server SYSTEM "server.dtd">
<mycat:server xmlns:mycat="http://io.mycat/">
	<system>
	<property name="useSqlStat">0</property>  <!-- 1Ϊ����ʵʱͳ�ơ�0Ϊ�ر� -->
	<property name="useGlobleTableCheck">0</property>  <!-- 1Ϊ����ȫ�Ӱ�һ���Լ�⡢0Ϊ�ر� -->

		<property name="sequnceHandlerType">2</property>
      <!--  <property name="useCompression">1</property>--> <!--1Ϊ����mysqlѹ��Э��-->
        <!--  <property name="fakeMySQLVersion">5.6.20</property>--> <!--����ģ���MySQL�汾��-->
	<!-- <property name="processorBufferChunk">40960</property> -->
	<!--
	<property name="processors">1</property>
	<property name="processorExecutor">32</property>
	 -->
		<!--Ĭ��Ϊtype 0: DirectByteBufferPool | type 1 ByteBufferArena-->
		<property name="processorBufferPoolType">0</property>
		<!--Ĭ����65535 64K ����sql����ʱ����ı����� -->
		<!--<property name="maxStringLiteralLength">65535</property>-->
		<!--<property name="sequnceHandlerType">0</property>-->
		<!--<property name="backSocketNoDelay">1</property>-->
		<!--<property name="frontSocketNoDelay">1</property>-->
		<!--<property name="processorExecutor">16</property>-->
		<!--
			<property name="serverPort">8066</property> <property name="managerPort">9066</property>
			<property name="idleTimeout">300000</property> <property name="bindIp">0.0.0.0</property>
			<property name="frontWriteQueueSize">4096</property> <property name="processors">32</property> -->
		<!--�ֲ�ʽ���񿪹أ�0Ϊ�����˷ֲ�ʽ����1Ϊ���˷ֲ�ʽ��������ֲ�ʽ������ֻ�漰ȫ�ֱ��򲻹��ˣ���2Ϊ�����˷ֲ�ʽ����,���Ǽ�¼�ֲ�ʽ������־-->
		<property name="handleDistributedTransactions">0</property>

			<!--
			off heap for merge/order/group/limit      1����   0�ر�
		-->
		<property name="useOffHeapForMerge">1</property>

		<!--
			��λΪm
		-->
		<property name="memoryPageSize">1m</property>

		<!--
			��λΪk
		-->
		<property name="spillsFileBufferSize">1k</property>

		<property name="useStreamOutput">0</property>

		<!--
			��λΪm
		-->
		<property name="systemReserveMemorySize">384m</property>


		<!--�Ƿ����zookeeperЭ���л�  -->
		<property name="useZKSwitch">true</property>


	</system>

	<!-- ȫ��SQL����ǽ����
	<firewall>
	   <whitehost>
	      <host host="172.18.0.2" user="root"/>
	      <host host="172.18.0.3" user="root"/>
				<host host="172.18.0.4" user="root"/>
	   </whitehost>
       <blacklist check="false">
       </blacklist>
	</firewall>-->

	<user name="root">
		<property name="password">jiabin</property>
		<property name="schemas">mall</property>

		<!-- �� DML Ȩ������ -->
		<!--
		<privileges check="false">
			<schema name="TESTDB" dml="0110" >
				<table name="tb01" dml="0000"></table>
				<table name="tb02" dml="1111"></table>
			</schema>
		</privileges>
		 -->
	</user>

</mycat:server>
```

#### log4j2.xml����

�������־�������Ϊdebug,�������ǹ۲����.

#### mycat��Dockerfile

```
FROM java:8-jre
MAINTAINER <ssab work_wjj@163.com>
LABEL Description="ʹ��mycat��mysql���ݿ�Ķ�д����"
ENV mycat-version Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz
USER root
COPY ./Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz /
RUN tar -zxf /Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz
ENV MYCAT_HOME=/mycat
ENV PATH=$PATH:$MYCAT_HOME/bin
WORKDIR $MYCAT_HOME/bin
RUN chmod u+x ./mycat
EXPOSE 8066 9066
CMD ["./mycat","console"]
```

### ����

��`docker-compose.yml`�ļ�Ŀ¼������
```shell
  docker-compose up -d
```

���û��������Ӧ�ľ����ļ�,��`docker-compose`���Զ���������.

ʹ��`docker-compose`�ֶ��������������:`docker-compose build mycat`

����ɹ�ִ��,������mycat,m1,s1,s2���Ѿ������ɹ�.

������`docker ps -a`����һ��.![mycat](https://github.com/sunshineasbefore/resource/blob/master/mycat.png?raw=true)

### ����

#### ����mycat�ͻ���

```
mysql -u root -p -P 8066 -h 127.0.0.1
```

#### ִ��select���

��Ϊ����һƪ�������Ѿ��������Ӹ��ƵĲ���,��������ط����ǾͲ����ظ���,����ֱ��ִ��`select`���,���Ƿ��Ѿ�ʵ���˶�д����.

```
mysql> select * from salesman limit 0,10;
```

�����:
![222](https://github.com/sunshineasbefore/resource/blob/master/mycat-salesman.png?raw=true)

Ȼ�����Ǵ�mycat����־mycat.log��һ��
![log](https://github.com/sunshineasbefore/resource/blob/master/mycatlog.png?raw=true)

ע�⿴ͼ�б�ǳ����ĵط�.

�ð�,����־�����ǿ�������ִ�е�`select`������ߴӿ�s1ִ�е�.

#### ִ��insert���

```
mysql> insert into salesman (id,user_num,true_name,address,mobile,disabled) values('30769','33333','ssab','ɽ��ʡ','33333321',0);
```

��mycat����־mycat.log��һ��

![insert](https://github.com/sunshineasbefore/resource/blob/master/mycatinsert.png?raw=true)

������Ƿ���,ִ��`insert`����ߵ�������m1.

�ܽ�
======

������,һ��ʹ��`mycat�м��`�mysql 1��2�� ���Ӹ��� ��д�����ʵ���������.

Ҫ˵Ϊʲôʹ��`mycat���ݿ��м��`,�ܼ򵥰�,������Ϊ���Կ�����Ա����û��Ӱ��,�������뵽������.