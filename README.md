This is to deploy EventConsumer and Remediation service with Python EventConsumer/Remediation App, Kafka, Elasticsearch and Kibana containers.

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
 

# docker-compose.yml file for elasticsearch and kibana
```
[jcluser@centos ~]$ cat docker-compose-ek.yml
version: '2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.5.1
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      - discovery.type=single-node
  kibana:
    image: docker.elastic.co/kibana/kibana:6.3.2
    ports:
      - "5601:5601"
```
make sure to set the max map count
```
sudo sysctl -w vm.max_map_count=262144
```

to trouble shoot the containers:
```
docker logs <container_id>
```

# install python packages
install pip3
```
sudo yum install python36 python-pip
sudo yum install python36-setuptools
sudo easy_install-3.6 pip
sudo pip3 install kafka
sudo pip3 install elasticsearch
```



# Other
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

# Test Cases
 The follow test cases are simplified version of different scenarios. Because the user action part is highly customizable, more complicated use cases can be built considering more conditions.  

# 1. Automatically bring up an interface shutdown by mistake

When an interface is shutdown, the system can detected and re-act to bring it up back.  

## Enable OpenConfig on the network device so that they can be monitored by Healthbot
If you use the HelperVM in JCL to load the router configs, the Openconfig should already be enabled. Here is an example of the Openconfig:
```
set system services extension-service request-response grpc clear-text address 100.123.1.1
set system services extension-service request-response grpc clear-text port 32767
set system services extension-service request-response grpc max-connections 10
set system services extension-service notification allow-clients address 0.0.0.0/0
```

## Configure Healthbot to monitor interface status

We can use Healthbot mgd cli to easily load the Healthbot configurations:  

Healthbot rule config:
```
jcluser@ubuntu:~$ healthbot mgd cli
root@43a788959893> request healthbot load
root@43a788959893> edit
set healthbot topic external rule interface_status keys element_name
set healthbot topic external rule interface_status sensor interface open-config sensor-name /interfaces
set healthbot topic external rule interface_status sensor interface open-config frequency 30s
set healthbot topic external rule interface_status field admin-status sensor interface path /interfaces/interface/state/admin-status
set healthbot topic external rule interface_status field admin-status type string
set healthbot topic external rule interface_status field element_name sensor interface where "/interfaces/interface/@name =~ /{{interface-name-variable}}/"
set healthbot topic external rule interface_status field element_name sensor interface path "/interfaces/interface/@name"
set healthbot topic external rule interface_status field element_name type string
set healthbot topic external rule interface_status field op-status sensor interface path /interfaces/interface/state/oper-status
set healthbot topic external rule interface_status field op-status type string
set healthbot topic external rule interface_status trigger interface_down frequency 5s
set healthbot topic external rule interface_status trigger interface_down term Term_1 when matches-with "$admin-status" DOWN
set healthbot topic external rule interface_status trigger interface_down term Term_1 then status color red
set healthbot topic external rule interface_status trigger interface_down term Term_1 then status message "$element_name admin state DOWN"
set healthbot topic external rule interface_status trigger interface_down term Term_2 when matches-with "$admin-status" UP
set healthbot topic external rule interface_status trigger interface_down term Term_2 then status color green
set healthbot topic external rule interface_status trigger interface_down term Term_2 then status message "$element_name admin state DOWN"
set healthbot topic external rule interface_status variable interface-name-variable value ge.*
set healthbot topic external rule interface_status variable interface-name-variable type string
```
Healthbot playbook configuration:
```
set healthbot playbook interface_status rules external/interface_status
```
Healthbot device and group configuration:
```
set healthbot device vMX-1 host 100.123.1.1
set healthbot device vMX-1 open-config port 32767
set healthbot device vMX-1 iAgent port 830
set healthbot device vMX-1 authentication password username jcluser
set healthbot device vMX-1 authentication password password "$9$CsS6pOIylMNdsEcds24DjCtuOBIEcy"
set healthbot device vMX-1 vendor juniper operating-system junos

set healthbot device-group all devices vMX-1
set healthbot device-group all playbooks interface_status
set healthbot device-group all notification normal kafka
set healthbot device-group all notification minor kafka
set healthbot device-group all notification major kafka
set healthbot device-group all notification enable
set healthbot device-group all variable interface_status interface_status external/interface_status running-state running
```
Healthbot Kafka notification setting:
```
set healthbot notification kafka kafka-publish bootstrap-servers 100.123.34.0
set healthbot notification kafka kafka-publish topic event
```
Deploy the config:
```
[edit]
root@43a788959893# commit and-quit
root@43a788959893> request healthbot deploy
```  
Validate that Healthbot is monitoring the device correctly:  
![healthbot_device_health](uploads/6d9caf4cf5637adfe8ef42758999b3cf/healthbot_device_health.png)


# 2. Automatically restart a server
When a server is running in trouble(e.g. high cpu, memory usage) , sometimes the solution is as simple as restarting the server. In this case, Appformix monitors the server CPU usage. When Appformix sees the server CPU 100% utilized, it can sent an alarm and the reaction system can trigger an action to restart the server.

set up keyless login from 100.123.34.0 to 100.123.34.1
on 100.123.34.0, generate keys:
```
[jcluser@centos ~]$ ssh-keygen -t rsa
```
copy the keys to the remote 100.123.34.1 server:
```
cat .ssh/id_rsa.pub | ssh jcluser@100.123.34.1 'mkdir -p ~/.ssh && chmod 700 ~/.ssh&& chmod 640 .ssh/authorized_keys && cat >>  ~/.ssh/authorized_keys'
```
in order to execute the sudo commands without typing password, we need to set up the following on the remote server 100.123.34.1:
```
[jcluser@centos ~]$ sudo cp /etc/sudoers /root/sudoers.bak
[jcluser@centos ~]$ sudo vi /etc/sudoers
```
append this line to the /etc/sudoers file:
```
jcluser ALL=(ALL) NOPASSWD:ALL
```

# 3. Send out emails for not handled events


# 4. Trigger Northstar controller to automatically put a problematicv link under maintenance
