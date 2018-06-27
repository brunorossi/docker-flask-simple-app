# Simple containerized Flask app

## CI Repository
The code is available on Github repository: https://github.com/brunorossi/docker-flask-simple-app

## Docker Image
The docker image is available on docker hub via automated build: https://hub.docker.com/r/brossi/docker-flask-simple-app/

## Docker Swarm provided via Vagrant
The Docker Image is deployable on a local machine via a Vagrant provided Docker Swarm Cluster
With Vagrant, Virtual Box and Git installed on you local Linux client you need to accomplish
the following steps to run the Docker Swarm virtual machines cluster:
git clone https://github.com/tdi/vagrant-docker-swarm.git

### Add a fix to resolve DNS
ATTENTION: Add the v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"] in the Vagrantfike
to allow the virtual machines to resolve DNS

config.vm.provider "virtualbox" do |v|
  v.memory = vmmemory
  v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  v.cpus = numcpu
end

### Install the vagrant-proxyconf plugin
vagrant plugin install vagrant-proxyconf

### Build the cluster
cd vagrant-docker-swarm
vagrant up
vagrant status

### Init The Swarm Manager Node
Run the following commands:
vagrant ssh manager
docker swarm init --advertise-addr 192.168.10.2
exit

### Add the first worker node to the cluster:
Run the following commands:
vagrant ssh worker1
docker swarm join \
    --token SWMTKN-1-2bidni8c9vk185iadiehhin2tkju67da55nanyh31gqzc3nuxk-0pix6nlsp4lossiiven3zhi2z \
    192.168.10.2:2377
exit

### Add the second worker node to the cluster:
Run the following commands:
vagrant ssh worker1
docker swarm join \
    --token SWMTKN-1-2bidni8c9vk185iadiehhin2tkju67da55nanyh31gqzc3nuxk-0pix6nlsp4lossiiven3zhi2z \
    192.168.10.2:2377
exit

### Check it out the Docker Swarm status
vagran ssh manager
docker node ls

### Create the docker-compose.yml file to deploy the Docker Swarm stack
cat <<EOF >>docker-compose.yml
version: '3'
services:
  flask-app:
    image: brossi/docker-flask-simple-app:latest
    deploy:
      replicas: 2
    ports:
      - "80:80"
    depends_on:
      - redis
    environment:
      ENV: production
      REDIS_HOST: redis
  redis:
    image: redis:alpine
    deploy:
      replicas: 1
EOF

### Deploy the stack
docker stack deploy --compose-file docker-compose.yml flask-app

### Test it
curl -X GET 127.0.0.1/get
{"status": "ok", "data": null}

curl -X POST 127.0.0.1/set
{"status": "ok"}

curl -X GET 127.0.0.1/get
{"status": "ok", "data": 1}
