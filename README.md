# Setting up JCL Lab

This is the instructions for setting it up using JCL with blueprint EDI.  

Login to JCL https://jlabs.juniper.net/jcl/portal/index.page From the reserve the blue print, reserve EDI
![JCL-EDI](uploads/73a43f6832b84fb4293db8fe53a7895e/JCL-EDI.png)

Use the following settings when reserve:  
![reserve-settings](uploads/a7dc362a9add2a83f6db42a6a25cc17e/reserve-settings.png)



## clone the project
```
[jcluser@centos ~]$git clone git@ssd-git.juniper.net:CPO-Solutions-Automation/edi.git
```
For some reason, JCL cannot access the project, you can manually download the project and upload to JCL CentOS-A1 server.

## set up keyless login from CentOS-A1(100.123.34.0) to CentOS-A2(100.123.34.1)
set up keyless login from CentOS-A1(100.123.34.0) to CentOS-A2(100.123.34.1), we will use CentOS-A2 for event consumer/remediation container
on 100.123.34.0, generate keys:
```
[jcluser@centos ~]$ ssh-keygen -t rsa
```
copy the keys to the remote 100.123.34.1 server:
```
cat .ssh/id_rsa.pub | ssh jcluser@100.123.34.1 'mkdir -p ~/.ssh && chmod 700 ~/.ssh && touch .ssh/authorized_keys &&  chmod 640 .ssh/authorized_keys && cat >>  ~/.ssh/authorized_keys'
```

## Use ansible playbook to deploy the containers
install ansible and python docker
```
[jcluser@centos ~]$ sudo yum install ansible
[jcluser@centos ansible]$ sudo python -m pip install --upgrade pip
[jcluser@centos ansible]$ pip install docker
```
in order to execute the sudo commands without typing password, we need to set up the following on both local 100.123.34.0 and the remote server 100.123.34.1:
```
[jcluser@centos ~]$ sudo cp /etc/sudoers /root/sudoers.bak
[jcluser@centos ~]$ sudo vi /etc/sudoers
```
append this line to the /etc/sudoers file:
```
jcluser ALL=(ALL) NOPASSWD:ALL
```
copy the Dockfile to the same level as edi directory
```
[jcluser@centos ~]$ cp edi/Dockerfile ./
[jcluser@centos ~]$ ls
Dockerfile  edi
```
check your inventory file, we use current server CentOS-A1(100.123.34.0) to host Kafka, Elastic and Kibana. Use server CentOS-A2(100.123.34.1) to host event-consumer-remediation container.
```
[jcluser@centos ~]$ cd edi/ansible/
[jcluser@centos ansible]$ cat inventory
[server_kek]
127.0.0.1

[server_event]
100.123.34.1
```
run the ansible playbook
```
[jcluser@centos ansible]$ ansible-playbook -i inventory deploy_all.yml
[WARNING]: While constructing a mapping from /home/jcluser/edi/ansible/deploy_all.yml, line 16, column 3, found a duplicate dict key (gather_facts). Using last defined
value only.

PLAY [deploy kafka, elasticsearch and kibana containers] ***************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************
ok: [127.0.0.1]

TASK [install_docker : install a list of packages] *********************************************************************************************************************
ok: [127.0.0.1]

TASK [install_docker : upgrade pip] ************************************************************************************************************************************
changed: [127.0.0.1]

TASK [install_docker : install pip docker] *****************************************************************************************************************************
changed: [127.0.0.1]

TASK [install_docker : uninstall old versions] *************************************************************************************************************************
ok: [127.0.0.1]

TASK [install_docker : set up the stable repository] *******************************************************************************************************************
changed: [127.0.0.1]

TASK [install_docker : Install the latest version of Docker Engine] ****************************************************************************************************
ok: [127.0.0.1]

TASK [install_docker : add user mod] ***********************************************************************************************************************************
changed: [127.0.0.1]

TASK [install_docker : start docker] ***********************************************************************************************************************************
changed: [127.0.0.1]

TASK [install_docker : Configure Docker to start on boot] **************************************************************************************************************
changed: [127.0.0.1]

TASK [create_network : Create  network] ********************************************************************************************************************************
ok: [127.0.0.1]

TASK [deploy_zookeeper : deploy zookeeper] *****************************************************************************************************************************
[DEPRECATION WARNING]: Please note that docker_container handles networks slightly different than docker CLI. If you specify networks, the default network will still
be attached as the first network. (You can specify purge_networks to remove all networks not explicitly listed.) This behavior will change in Ansible 2.12. You can
change the behavior now by setting the new `networks_cli_compatible` option to `yes`, and remove this warning by setting it to `no`. This feature will be removed in
version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
changed: [127.0.0.1]

TASK [deploy_kafka : deploy kafka] *************************************************************************************************************************************
changed: [127.0.0.1]

TASK [deploy_elasticsearch : deploy elasticsearch] *********************************************************************************************************************
changed: [127.0.0.1]

TASK [deploy_kibana : deploy kibana] ***********************************************************************************************************************************
changed: [127.0.0.1]

TASK [build_edi_image : build the docker image for event consume/remediation] ******************************************************************************************
[WARNING]: The default for build.pull is currently 'yes', but will be changed to 'no' in Ansible 2.12. Please set build.pull explicitly to the value you need.
ok: [127.0.0.1]

TASK [build_edi_image : archive the edi image locally] *****************************************************************************************************************
changed: [127.0.0.1]

PLAY [deploy event consumer and remediation server] ********************************************************************************************************************

TASK [install_docker : install a list of packages] *********************************************************************************************************************
ok: [100.123.34.1]

TASK [install_docker : upgrade pip] ************************************************************************************************************************************
changed: [100.123.34.1]

TASK [install_docker : install pip docker] *****************************************************************************************************************************
changed: [100.123.34.1]

TASK [install_docker : uninstall old versions] *************************************************************************************************************************
ok: [100.123.34.1]

TASK [install_docker : set up the stable repository] *******************************************************************************************************************
changed: [100.123.34.1]

TASK [install_docker : Install the latest version of Docker Engine] ****************************************************************************************************
ok: [100.123.34.1]

TASK [install_docker : add user mod] ***********************************************************************************************************************************
changed: [100.123.34.1]

TASK [install_docker : start docker] ***********************************************************************************************************************************
changed: [100.123.34.1]

TASK [install_docker : Configure Docker to start on boot] **************************************************************************************************************
changed: [100.123.34.1]

TASK [upload_edi_image : copy edi_image to remote event consume/remediation server] ************************************************************************************
changed: [100.123.34.1]

TASK [deploy_edi : load the edi image] *********************************************************************************************************************************
changed: [100.123.34.1]

TASK [deploy_edi : deploy edi docker] **********************************************************************************************************************************
changed: [100.123.34.1]

PLAY RECAP *************************************************************************************************************************************************************
100.123.34.1               : ok=12   changed=9    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
127.0.0.1                  : ok=17   changed=11   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
make sure Kafka, Elasticsearch and Kibana are all up and running on CentOS-A1:
```
[jcluser@centos ansible]$ docker ps
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                                            NAMES
fef1343b0ee5        docker.elastic.co/kibana/kibana:7.5.1                 "/usr/local/bin/du..."   22 hours ago        Up 7 minutes        0.0.0.0:5601->5601/tcp                           kibana
cc5ece1f2069        docker.elastic.co/elasticsearch/elasticsearch:7.5.1   "/usr/local/bin/do..."   22 hours ago        Up 7 minutes        0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   elasticsearch
e278ea4d5036        wurstmeister/kafka:2.11-2.0.0                         "start-kafka.sh"         22 hours ago        Up 7 minutes        0.0.0.0:9092->9092/tcp, 9093/tcp                 kafka
22c2e83b943e        wurstmeister/zookeeper:3.4.6                          "/bin/sh -c '/usr/..."   22 hours ago        Up 7 minutes        22/tcp, 2181/tcp, 2888/tcp, 3888/tcp             zookeeper
```
login to CentOS-A2, make sure event-consumer-remediation container is up and running:
```
[jcluser@centos ansible]$ ssh jcluser@100.123.34.1
Last login: Wed Jan 22 15:43:31 2020 from 100.123.34.0
[jcluser@centos ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
99875998a672        edi                 "/bin/sh -c 'pytho..."   17 hours ago        Up 7 minutes                            edi
[jcluser@centos ~]$
```

## use Helper VM to load the network config
login to the helperVM
```
[root@HelperVM-0114 ~]#git clone git@git.cloudlabs.juniper.net:wouyang/edi.git
[root@HelperVM-0114 ~]# cd edi/
[root@HelperVM-0114 ~]# ansible-playbook install-config-to-device.yml
```

## Configure Kibana
From the JCL sandbox GUI, click "Http UserDef1" from the CentOS-A1 menu:
![kibana1](uploads/0a1b6d15e0867153bab514cbddd411ac/kibana1.png)

create Index:
![kibana4](uploads/1d4ae1de3637c4ba6b69884507672ccd/kibana4.png)

if there are events, then it can be displaed:
![kibana2](uploads/42078bc1918b7e2e049de8f737021856/kibana2.png)


# Other notes

## Validate the Kafka container  
To test the producer, login to the container:
```
[jcluser@centos ~]$ docker exec -it <container_id> /bin/sh
```
To start a producer:
```
 $KAFKA_HOME/bin/kafka-console-producer.sh --broker-list kafka:9093 --topic test
```
 
To test the consumer:
 ```
 [jcluser@centos ~]$ docker exec -it 51080e6d24af /bin/sh
 $KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server kafka:9093 --topic test --from-beginning
 ```

To test from another host machine:
```
[jcluser@centos ~]$ python3
Python 3.6.8 (default, Apr 25 2019, 21:02:35)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-36)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from kafka import KafkaProducer
>>> producer = KafkaProducer(bootstrap_servers=['<kafka_broker_server_ip>:9092'])
>>>
>>> data = b"hi"
>>> producer.send('test', value=data)
<kafka.producer.future.FutureRecordMetadata object at 0x7f7792697160>
>>> producer.send('test', value=b'mimimi mamama gugugugu')
<kafka.producer.future.FutureRecordMetadata object at 0x7f7792697e48>

[jcluser@centos ~]$ vi consumer.py
from kafka import KafkaConsumer
consumer = KafkaConsumer(
    'test',
     auto_offset_reset='earliest',
     bootstrap_servers=['100.123.34.0:9092']
     )
for message in consumer:
    message = message.value
    print(message)
[jcluser@centos ~]$ python3 consumer.py
b'hi'
 ```

## If you plan to use docker compose to start up the containers

## install docker compose
for details please refer to https://docs.docker.com/compose/install/
```
sudo yum install python-pip python-devel gcc gcc-c++ make openssl-devel libffi-devel
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

```
[jcluser@centos ~]$ cd edi/
[jcluser@centos edi]$ docker-compose up -d
docker-compose up -d
```
check all containers are running:
```
[jcluser@centos ~]$ docker ps
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                                            NAMES
fb8b6263fa83        wurstmeister/kafka:2.11-2.0.0                         "start-kafka.sh"         5 hours ago         Up 5 hours          0.0.0.0:9092->9092/tcp, 9093/tcp                 jcluser_kafka_1
213cca63f70b        docker.elastic.co/kibana/kibana:6.3.2                 "/usr/local/bin/ki..."   5 hours ago         Up 5 hours          0.0.0.0:5601->5601/tcp                           jcluser_kibana_1
3551be48b1f2        docker.elastic.co/elasticsearch/elasticsearch:7.5.1   "/usr/local/bin/do..."   5 hours ago         Up 5 hours          0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   jcluser_elasticsearch_1
9a05b62d8cda        wurstmeister/zookeeper:3.4.6                          "/bin/sh -c '/usr/..."   5 hours ago         Up 5 hours          22/tcp, 2181/tcp, 2888/tcp, 3888/tcp             jcluser_zookeeper_1
```

# notes manualy deploy

# deploy Kafka, Elasticsearch and Kibana 
We are using docker and ansible to deploy all these services as containers. Please make sure you have docker and ansible installed.

Login to the Centos A1(100.123.34.0) box:

## install docker for Centos
for details please refer to https://docs.docker.com/install/linux/docker-ce/centos/
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

## change settings to run Elasticsearch
```
sudo sysctl -w vm.max_map_count=262144
```

##
```
sudo yum install python-pip
sudo pip install docker-py

```
## install python3 packages for kafka and elasticsearch testing
JCL Centos box has Python3.6 by default (as of Jan 05, 2020)
```
sudo yum install python36 python-pip
sudo yum install python36-setuptools
sudo easy_install-3.6 pip
sudo pip3 install kafka
sudo pip3 install elasticsearch
```

## install Junos PyEZ packages for junos automation
```
sudo pip3 install junos-eznc
```

## install other packages needed
```
sudo yum install python-pip python-devel gcc gcc-c++ make openssl-devel libffi-devel
sudo pip3 install requests
```
