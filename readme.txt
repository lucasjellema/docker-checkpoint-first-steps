Vagrant 2.0.3
Virtual Box 5.2.8
https://criu.org/Docker
http://www.admin-magazine.com/Archive/2014/22/Save-and-Restore-Linux-Processes-with-CRIU



note: Vagrant Docker provisioning: https://www.vagrantup.com/docs/provisioning/docker.html 

vagrant up
vagrant ssh

sudo apt-get upgrade

downgrade Docker: https://forums.docker.com/t/how-to-downgrade-docker-to-a-specific-version/29523/4 
(down to 17.03 or 17.04 when CRIU was still working)
remove Docker

sudo apt-get autoremove -y docker-ce \
&& sudo apt-get purge docker-ce -y \
&& sudo rm -rf /etc/docker/ \
&& sudo rm -f /etc/systemd/system/multi-user.target.wants/docker.service \
&& sudo rm -rf /var/lib/docker \
&&  sudo systemctl daemon-reload

then:

sudo apt-cache policy docker-ce

sudo apt-get install -y docker-ce=17.04.0~ce-0~ubuntu-xenial


confige Docker for experimental options: https://stackoverflow.com/questions/44346322/how-to-run-docker-with-experimental-functions-on-ubuntu-16-04
sudo nano /etc/docker/daemon.json
add
{ 
    "experimental": true 
} 
sudo service docker restart
docker version


install CRIU: https://criu.org/Packages#Ubuntu
sudo apt-get install criu

https://yipee.io/2017/06/saving-and-restoring-container-state-with-criu/

docker container run --name=AppTestR17491 -rm AppTest:R17.49.1

docker checkpoint create  --leave-running=false AppTestR1749 AppTestR17491CheckPoint 

docker container start -–checkpoint=AppTestR17491CheckPoint AppTestR17491

optional:
 --checkpoint-dir=${chkptdir} 

description: https://github.com/docker/cli/blob/master/experimental/checkpoint-restore.md 


docker run --security-opt=seccomp:unconfined --name cr -d busybox /bin/sh -c 'i=0; while true; do echo $i; i=$(expr $i + 1); sleep 1; done'

docker logs cr 


docker checkpoint create  --leave-running=true cr checkpoint0 

docker logs cr 

docker checkpoint create cr checkpoint1


docker checkpoint ls  cr 

# <later>
docker start --checkpoint checkpoint1 cr

docker logs cr 


On Docker 17.12 and 18.03 I ran into: https://github.com/moby/moby/issues/35691 -  working in docker ce 17.03 then I upgraded to 17.12 and it broke.
Error response from daemon: open /var/lib/docker/containers/f4aec8385fbcb5e32772135d7b731ce9a2f69064dd5fdb6d40fce948c51b4826/checkpoints/checkpoint2x/config.json: no such file or directory

With downgrading to 17.04, it seems to work.




docker run --name reqctr2 -e "GIT_URL=https://github.com/lucasjellema/microservices-choreography-kubernetes-workshop-june2017" -e "APP_PORT=8080" -p 8005:8080 -e "APP_HOME=part1"  -e "APP_STARTUP=requestCounter.js"   lucasjellema/node-app-runner

access requestcounter in browser on host:
http://192.168.188.106:8005/



docker checkpoint create  --leave-running=true reqctr checkpoint1

(at some value of the request counter)

perform some additional requests

then stop the container

docker stop reqctr

docker start --checkpoint checkpoint1 reqctr2
rapid start of the container!
perform a request: back at the value at the time of taking the snapshot

perform some additional requests

docker stop reqctr

docker start --checkpoint checkpoint1 reqctr2

perform a request: back again at the value at the time of taking the snapshot



======

build CRIU (to get latest version)
https://criu.org/Installation


sudo apt-get autoremove -y criu \
&& sudo apt-get purge criu -y \


sudo apt-get install build-essential
sudo apt-get install gcc (is already there)
sudo apt-get install libprotobuf-dev libprotobuf-c0-dev protobuf-c-compiler protobuf-compiler python-protobuf

sudo apt-get install pkg-config python-ipaddr iproute2 libcap-dev  libnl-3-dev libnet-dev --no-install-recommends

git clone https://github.com/checkpoint-restore/criu

cd criu
make

sudo apt-get install asciidoc  xmlto
sudo make install
criu check

sudo ./criu/criu check --ms

sudo apt-get install  libaio-dev python-yaml

test/zdtm.py run -a
=============
