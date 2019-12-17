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
