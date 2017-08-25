
ϵͳ����
=======
* docker 1.12.3
* mysql5.7.17
* deepin 15.3�����(���ûɶӰ��,��Ϊ������docker)

Ҫ��˵��
======
* ʹ��`docker bridge`����,���þ�̬IP
* ʹ��`volumes`����,��ʹ�����ݾ�����(��Ϊ��ʹ��`docker compose`û��ɹ� - -!)
* ����ʹ��`build`����(������չ��),��ʹ��`image`
* ĿǰΪֹ,û�б�¶�˿ں�,ֻ������`slave`link��`master`.���������о�ʹ��[mycat](http://www.mycat.org.cn/)���mysql�����Ӹ���+��д����,�����ڴ�.
* ����`hosts`�ļ�,�Ա���ʹ��`hostname`����ip��ַ

Begin
=====

### `docker-engine`��װ

  ���ֱ�Ӳο��ٷ��ĵ���.[debian�°�װdocker-engine](https://docs.docker.com/engine/installation/linux/debian/)

### `docker-compose`��װ

  [debian�°�װdocker-compose](https://docs.docker.com/compose/install/)

### ��ȡ`mysql:5.7.17`����

  ```
    docker pull mysql:5.7.17
  ```
### ��Ҫ���ص������ļ�

#### Ŀ¼�ṹ

ֱ����ͼ:![Ŀ¼�ṹ](https://github.com/sunshineasbefore/resource/blob/master/mysql-replaction-%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.png?raw=true)

##### ��Ҫ˵��

* mysql-master: ���master�����ļ�
* mysql-s1: ��ŵ�һ��slave�����ļ�
* mysql-s2: ��ŵڶ���slave�����ļ�
* hosts: ����·��

#### mysql-master������

û���ٶ���,ֻ��һ��`mysqld.cnf`��Ҫ��ĩβ׷��:

```
  #���������ִ�Сд
  lower_case_table_names=1
  #�����ݿ�����Ψһ��ʶ��һ��Ϊ������÷�����Ip��ĩβ��
  server-id=2
  log-bin=master-bin
  log-bin-index=master-bin.index
```
#### mysql-s1/mysql-s2������

��masterһ��,Ҳֻ��һ��`mysqld.cnf`��Ҫ��ĩβ׷��:

```
  server-id=3 #��һ����3,�ڶ�����4
  log-bin=s1-bin.log #��һ����s1-bin.log,�ڶ�����s2-bin.log
  sync_binlog=1
  lower_case_table_names=1
```

#### hosts�ļ�����

```
127.0.0.1	localhost
172.18.0.2	m1
172.18.0.3	s1
172.18.0.4	s2
```

### docker-compose�����ļ���Dockerfile

##### Ŀ¼�ṹ

ֱ����ͼ:![Ŀ¼�ṹ](https://github.com/sunshineasbefore/resource/blob/master/docker-compose-mysql-%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.png?raw=true)

##### Dockerfile

��ʵ`master s1 s2`��Dockerfile����һ�µ�,�۾���Ϊ�˱���һ������չ�Բ���ôд��.
������ȫ������`docker-compose`��`image`����.

**��docker-compose.yml��image��build����һ��ʹ�õ�**

�ð�,��һ�����`Dockerfile`

```
FROM mysql:5.7.17
MAINTAINER <ssab work_wjj@163.com>
EXPOSE 3306
CMD ["mysqld"]
```

##### docker-compose.yml

�ð�,�ص�����.

```
version: '2' # ���version��ָdockerfile����ʱ�õİ汾,���Ǹ������Լ�����汾���õ�.
services:
  m1: # master
    build: ./master # ./master�ļ�����Ҫ��Dockerfile�ļ�,����build���Ժ�image���Բ���һ��ʹ��
    container_name: m1 # ������
    volumes: # ���� �±�ÿ��ǰ�ߵ�`-`������������������һ��Ԫ��.����˵volumes���Ե�ֵ��һ������
      - /home/ssab/config/mysql-master/:/etc/mysql/:ro # ע�����ӳ���ϵ
      - /etc/localtime:/etc/localtime:ro
      - /home/ssab/config/hosts:/etc/hosts:ro # ע�����ӳ���ϵ
    networks: # ����
      mysql: # ����servicesƽ����networks,�����±�
        ipv4_address: 172.18.0.2 # ���þ�̬ipv4�ĵ�ַ
    ulimits: # ����ϵͳ����
      nproc: 65535
    hostname: m1 # hostname
    mem_limit: 1024m # ����ڴ�ʹ�ò�����1024m,���ڱ��ػ����ϲ���,��ֻд��1024m,��������Ҫ�����Լ��ķ���������,�Լ�docker���������е���.
    restart: always # ������������
    environment: # ���û�������
      MYSQL_ROOT_PASSWORD: m1test
  s1: # slave1
      build: ./s1
      container_name: s1
      volumes:
        - /home/ssab/config/mysql-s1/:/etc/mysql/:ro
        - /etc/localtime:/etc/localtime:ro
        - /home/ssab/config/hosts:/etc/hosts:ro
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
  s2:# slave2
    build: ./s2
    container_name: s2
    volumes:
      - /home/ssab/config/mysql-s2/:/etc/mysql/:ro
      - /etc/localtime:/etc/localtime:ro
      - /home/ssab/config/hosts:/etc/hosts:ro
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
networks: # docker��������
  mysql: # �Զ�����������
    driver: bridge # �Ž�
    ipam: # Ҫʹ�þ�̬ip����ʹ��ipam���
      driver: default
      config:
      - subnet: 172.18.0.0/24
        gateway: 172.18.0.1

```

### run

��`docker-compose.yml`�ļ���Ŀ¼������
```
  docker-compose up -d
```

�𼤶�,�������ڲ�ֻ�������һ��....

### ����mysql���Ӹ���.

#### ����master

* ����master��mysql������
```
docker exec -it m1 /bin/bash
```
```
  mysql -u root -p
```

  ����`MYSQL_ROOT_PASSWORD:`��ֵm1test,����mysql������ģʽ.

* �����������Ӹ��Ƶ��û�`repl`

  ```
  mysql> create user repl;
  ```

* ��`repl`�û�����slave��Ȩ��

  ```
  #repl�û��������REPLICATION SLAVEȨ�ޣ�����֮��û�б�Ҫ��Ӳ���Ҫ��Ȩ�ޣ�����Ϊrepl��˵��һ��172.18.0.%�����������ָ��repl�û����ڷ�����������%��ͨ�������ʾ172.18.0.0-172.18.0.255��Server��������repl�û���½������������Ȼ��Ҳ����ָ���̶�Ip��
  mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'172.18.0.%' IDENTIFIED BY 'repl';
  ```

* ����,���������ٽ���д�붯��,��������ڽ����ն˻Ự��ʱ����Զ�����

  ```
  FLUSH TABLES WITH READ LOCK;
  ```

* �鿴master״̬

  ```
  mysql> show master status;
  ```

  ��ʾ����:
  ```
  +-------------------+----------+--------------+------------------+-------------------+
  | File              | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
  +-------------------+----------+--------------+------------------+-------------------+
  | master-bin.000003 |      636 |              |                  |                   |
  +-------------------+----------+--------------+------------------+-------------------+
  1 row in set (0.00 sec)
  ```

  ����`master-bin.000003`��`636`һ����slave��.

#### ����slave1

* ����s1��mysql������
```
docker exec -it s1 /bin/bash
```
```
  mysql -u root -p
```

����`MYSQL_ROOT_PASSWORD:`��ֵs1test,����mysql������ģʽ.

* ����master

  ```
  mysql> change master to master_host='m1',master_port=3306,master_user='repl',master_password='repl',master_log_file='master-bin.000003',master_log_pos=636;
  ```

* ����slave

  ```
  mysql> start salve;
  ```

#### ����slave2

  ������slaveһ��....�۾Ͳ�д��...

### ʵ��

  ����,����λ��,�����Ѿ����,���Ƿ�ɹ�����...
  ��������һ��.

#### ����masterд����Ƿ��ܹ�ͬ����slave

* ��master��mysql�������´������ݿ�:`ms-test`

  ```
  mysql> create database mstest;
  ```

* ȥ��̨slave�ϲ鿴�Ƿ�Ҳ����mstest���ݿ�.

  ```
  mysql> show databases;
  ```

  �����ʾ����:

  ```
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | mstest             |
  | mysql              |
  | performance_schema |
  | sys                |
  +--------------------+
  5 rows in set (0.00 sec)
  ```

  ��֤���ɹ�.

#### �ۻ����Դ���һ����,����һ����������

  �Լ�����,������.

�ܽ�
==========
ͨ�����ϲ���,���Ǵ��һ����`docker-compose`�����mysql `master-slave`ģʽ�����Ӹ���.
��һ��,������Ҫ���б�¶�˿�,����ʹ��`links`��������Ӧ�û��������ͻ��˹����ܹ��������ǵ�mysql.
����һ��,������Ҫʹ��`mycat`�м����������ǵĶ�д����.

��ͬŬ��,һ�����!