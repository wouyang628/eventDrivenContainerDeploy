This is to deploy EventConsumer and Remediation service with Python EventConsumer/Remediation App, Kafka, Elasticsearch and Kibana containers.

This is the instructions for setting it up using JCL with blueprint EDI.  

Login to JCL https://jlabs.juniper.net/jcl/portal/index.page From the reserve the blue print, reserve EDI
![JCL-EDI](uploads/73a43f6832b84fb4293db8fe53a7895e/JCL-EDI.png)

Use the following settings when reserve:  
![reserve-settings](uploads/a7dc362a9add2a83f6db42a6a25cc17e/reserve-settings.png)

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
cat .ssh/id_rsa.pub | ssh jcluser@100.123.34.1 'mkdir -p ~/.ssh && chmod 700 ~/.ssh&& chmod 640 .ssh/authorized_keys && cat >>  ~/.ssh/authorized_keys'
```

## Use ansible playbook to deploy the containers
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

PLAY [deploy kafka, elasticsearch and kibana containers] ***********************************************************************************************************************************

TASK [deploy_zookeeper : deploy zookeeper] *************************************************************************************************************************************************
[DEPRECATION WARNING]: Please note that docker_container handles networks slightly different than docker CLI. If you specify networks, the default network will still be attached as the
first network. (You can specify purge_networks to remove all networks not explicitly listed.) This behavior will change in Ansible 2.12. You can change the behavior now by setting the new
 `networks_cli_compatible` option to `yes`, and remove this warning by setting it to `no`. This feature will be removed in version 2.12. Deprecation warnings can be disabled by setting
deprecation_warnings=False in ansible.cfg.
changed: [127.0.0.1]

TASK [deploy_kafka : deploy kafka] *********************************************************************************************************************************************************
changed: [127.0.0.1]

TASK [deploy_elasticsearch : deploy elasticsearch] *****************************************************************************************************************************************
changed: [127.0.0.1]

TASK [deploy_kibana : deploy kibana] *******************************************************************************************************************************************************
changed: [127.0.0.1]

PLAY [deploy event consumer and remediation server] ****************************************************************************************************************************************

TASK [deploy_event : deploy event] *********************************************************************************************************************************************************
changed: [100.123.34.1]

PLAY RECAP *********************************************************************************************************************************************************************************
100.123.34.1               : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
127.0.0.1                  : ok=4    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
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
From the JCL sandbox GUI, click "Http UserDef1" from the CentOS-A1 menu:
![kibana1](uploads/0a1b6d15e0867153bab514cbddd411ac/kibana1.png)
make sure Kibana GUI can be opened:
![kibana2](uploads/42078bc1918b7e2e049de8f737021856/kibana2.png)


# Other notes
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

## use Helper VM to load the network config
login to the helperVM
```
[root@HelperVM-0114 ~]#git clone git@git.cloudlabs.juniper.net:wouyang/edi.git
[root@HelperVM-0114 ~]# cd edi/
[root@HelperVM-0114 ~]# ansible-playbook install-config-to-device.yml
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
