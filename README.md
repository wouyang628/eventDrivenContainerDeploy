# kafkacontainer

# install docker for centos

```
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
  
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
  
$ sudo yum install docker-ce docker-ce-cli containerd.io

$ sudo systemctl start docker

$ sudo docker run hello-world

$ sudo groupadd docker

$ sudo usermod -aG docker $USER

Log out and log back in so that your group membership is re-evaluated.

$ docker run hello-world

if error "Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.26/containers/create: dial unix /var/run/docker.sock: connect: permission denied.
See '/usr/bin/docker-current run --help'."
then: sudo chmod 666 /var/run/docker.sock

Configure Docker to start on boot:
$ sudo systemctl enable docker

```

# install docker compose

```
sudo yum install python-pip python-devel gcc gcc-c++ make openssl-devel libffi-devel
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

# create docker-compose file
```
[jcluser@centos ~]$ cat docker-compose.yml.bak1
version: '2'

services:

  zookeeper:
    image: wurstmeister/zookeeper:3.4.6
    expose:
    - "2181"

  kafka:
    image: wurstmeister/kafka:2.11-2.0.0
    depends_on:
    - zookeeper
    ports:
    - "9092:9092"
    expose:
    - "9093"
    environment:
      KAFKA_ADVERTISED_LISTENERS: INSIDE://kafka:9093,OUTSIDE://100.123.34.0:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
      KAFKA_LISTENERS: INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
 ```
 bootup:
 ```
 docker-compose up -d
 ```
 test producer:
 ```
 login to the container:
 [jcluser@centos ~]$ docker ps
CONTAINER ID        IMAGE                           COMMAND                  CREATED             STATUS              PORTS                                  NAMES
51080e6d24af        wurstmeister/kafka:2.11-2.0.0   "start-kafka.sh"         47 seconds ago      Up 46 seconds       0.0.0.0:9092->9092/tcp                 jcluser_kafka_1
833c39f1d2bb        wurstmeister/zookeeper:3.4.6    "/bin/sh -c '/usr/..."   48 seconds ago      Up 47 seconds       22/tcp, 2181/tcp, 2888/tcp, 3888/tcp   jcluser_zookeeper_1
[jcluser@centos ~]$ docker exec -it 51080e6d24af /bin/sh
 
 start a producer:
 $KAFKA_HOME/bin/kafka-console-producer.sh --broker-list localhost:9093 --topic test
 ```
 
 test consumer:
 ```
 [jcluser@centos ~]$ docker exec -it 51080e6d24af /bin/sh
/ #
/ # $KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
 ```
 
 ```
 [jcluser@centos kafka_2.12-2.4.0]$ docker exec -it 65583b630c15 /bin/sh
 # $KAFKA_HOME/bin/kafka-console-producer.sh --broker-list kafka:9093 --topic test
 ```

 from another host:
```
sudo yum install python36 python-pip
sudo yum install python36-setuptools
sudo easy_install-3.6 pip
sudo pip3 install kafka
```
 ```
 [jcluser@centos ~]$ python3
Python 3.6.8 (default, Apr 25 2019, 21:02:35)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-36)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from kafka import KafkaProducer
>>> producer = KafkaProducer(bootstrap_servers=['100.123.34.0:9092'])
>>>
>>> data = b"hi"
>>> producer.send('test', value=data)
<kafka.producer.future.FutureRecordMetadata object at 0x7f7792697160>
>>> producer.send('test', value=b'mimimi mamama gugugugu')
<kafka.producer.future.FutureRecordMetadata object at 0x7f7792697e48>

 [jcluser@centos ~]$ python3 consumer.py
 b'hi'
b'mimimi mamama gugugugu'
 ```
 ```
 [jcluser@centos ~]$ cat consumer.py
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'test',
     auto_offset_reset='earliest',
     bootstrap_servers=['100.123.34.0:9092']
     )

for message in consumer:
    message = message.value
    print(message)
 ```
 



works only on local host machine:
```
version: '2'

services:

  zookeeper:
    image: wurstmeister/zookeeper:3.4.6
    expose:
    - "2181"

  kafka:
    image: wurstmeister/kafka:2.11-2.0.0
    depends_on:
    - zookeeper
    ports:
    - "9092:9092"
    expose:
    - "9093"
    environment:
      KAFKA_ADVERTISED_LISTENERS: INSIDE://kafka:9093,OUTSIDE://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
      KAFKA_LISTENERS: INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
 ```
