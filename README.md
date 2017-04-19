
Private comments about some Docker course
```
EXAMPLE DOCKER FILE
==================
ubuntu@ip-172-31-19-178:~$ cat Dockerfile
###########################################
# Dockerfile to build Predictive-reclaim
###########################################
# Base image is Ubuntu
FROM amazonlinux
# Author: Dr. Peter
MAINTAINER Ing. M.V. Nossin <nossin_m@schiphol.nl>
# Install os tools
RUN yum install wget  <<< $'y\n'
RUN yum install vim  <<< $'y\n'
RUN yum install -y bzip2 

#Install Anaconda3
RUN wget https://repo.continuum.io/archive/Anaconda3-4.3.0-Linux-x86_64.sh
RUN bash Anaconda3-4.3.0-Linux-x86_64.sh <<< $'\nyes\n\nyes\n'

#Set env, TODO LANG has to be set in .bashrc root
ENV PATH /root/anaconda3/bin:$PATH
ENV LANG en_US.UTF-8

#Install Api server
RUN mkdir /root/predictive-reclaim
ADD pd.tar /root/predictive-reclaim
EXPOSE 80/tcp



THE README FOR DOCKER:
===============
ubuntu@ip-172-31-19-178:~$ cat README
INSTALL DOCKER:
sudo sh -c "echo deb https://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9
sudo apt-get update
sudo apt-get install -y lxc-docker

In case you have to start manually:
sudo service docker start

sudo docker version
sudo docker -D info

sudo docker pull busybox    #download latest version
sudo docker images
sudo docker run busybox echo "Hello World!"
sudo service docker status
tail -f /var/log/upstart/docker.log

sudo docker pull -a busybox #download all versions
sudo docker images

sudo docker run -t -i busybox:ubuntu-14.04 #run specific version 
sudo docker pull busybox:ubuntu-14.04     #download specific version

#docker will pull by default from index.docker.io
sudo docker pull thedockerbook/helloworld   #download any 3rd party image from another location by providing the username, but still same repo

#to use another repository just provide it: 
sudo docker pull registry.example.com/myapp

sudo docker search mysql
sudo docker run -i -t ubuntu:14.04 /bin/bash   #-i stands for interactive mode
<CTRL Q + CTRL P : DETACH BUT KEEP RUNNING>
docker status ps
sudo docker attach  <NAME generated and seen in ps statement> 

<GENERATE FILE IN A CONTAINER USING TOUCH>
<DETACH>
sudo docker ps
<check hostname>
sudo docker diff <hostname> 
<SEE RESULT>

sudo docker stop <hostname>
sudo docker ps
<its gone>
sudo docker ps -a
<shows it again>
sudo docker attach <container>
<and your'in> 

<note start, stop and restart of containers can be done>

sudo docker pause (and unpause) <container> 
<This will freeze execution but keep state>

HOUSEKEEPING:
To avoid all docker containers on the filesystem (and making the ps -a outlong longer and longer) start like this:
sudo docker run -i -t --rm ubuntu:14.04 /bin/bash 

To remove containers already in the list
sudo docker rm <id>
or to remove them all (use -f option to force if some are running):
sudo docker rm `sudo docker ps -aq --no-trunc`

CREATE IMAGE (test only, use dockerfile for prod)
sudo docker run -i -t ubuntu:14.04 /bin/bash
which wget
apt-get update
apt-get install -y wget
which wget
exit
sudo docker diff <id>
sudo docker commit <id> <user/repo like learningdocker/ubuntu_wget>
sudo docker images

RUNNING AS DETACHED/DEAMON
sudo docker run -d ubuntu \
/bin/bash -c "while true; do date; sleep 5; done"
<it will keep on running detached>
sudo docker logs

CREATE IMAGE DOCKERFILE (PRODUCTION)
Create Dockerfile:
FROM busybox:latest
CMD echo Hello World!!

sudo docker build .
<or using tag>
sudo docker tag <id> michel
sudo docker build -t michel
sudo docker run michel
sudo docker images
<michel is there>


SYNTAX DOCKERFILE
FROM <image>[:<tag>]
FROM ubuntu:14.04 or FROM centos (latest)

MAINTAINER Ing, M.V. Nossin <nossin_m@schiphol.nl>

COPY <src> ... <dst>
COPY html /var/www/html
COPY httpd.conf magic /etc/httpd/conf/

ADD <src> ... <dst>
$ tar tf web-page-config.tar
etc/httpd/conf/httpd.conf
var/www/html/index.html
var/www/html/aboutus.html
var/www/html/images/welcome.gif
var/www/html/images/banner.gif
ADD web-page-config.tar /    (will copy adn untar all files to right place)

ENV <key> <value>   (set env var)
ENV DEBUG_LVL 3
ENV APACHE_LOG_DIR /var/log/apache

USER <UID>|<UName> (to prevent the default user, root, to be used, must be in /etc/passwd)
USER 73
USER nossin_m

WORKDIR <dirpath>  (like cs statement)
WORKDIR /home/ec2-user

VOLUME ["<mountpoint>"] or VOLUME <mountpoint>   (creates volume)

EXPOSE <port>[/<proto>] [<port>[/<proto>]...] (open port to world)
EXPOSE 7373/udp 8080

RUN <command> (run build time)
RUN ["<exec>", "<arg-1>", ..., "<arg-n>"]
RUN ["bash", "-c", "rm", "-rf", "/tmp/abc"]
RUN echo "hallo"


FULL EXAMPLE RUN CMD

###########################################
# Dockerfile to build an Apache2 image
###########################################
# Base image is Ubuntu
FROM ubuntu:14.04

# Author: Dr. Peter
MAINTAINER Dr. Peter <peterindia@gmail.com># Author: Dr. Peter

# Install apache2 package
RUN apt-get update && \
apt-get install -y apache2 && \
apt-get clean

CMD <command> (run after launch, not during build)
CMD ["<exec>", "<arg-1>", ..., "<arg-n>"]

FULL EXAMPLE CMD
########################################################
# Dockerfile to demonstrate the behaviour of CMD
########################################################
# Build from base image busybox:latest
FROM busybox:latest
# Author: Dr. Peter
MAINTAINER Dr. Peter <peterindia@gmail.com>
# Set command for CMD
CMD ["echo", "Dockerfile CMD demo"]

To build and run make sure the commandos are in Dockerfile file in current dir and copy these statement:
sudo docker build -t cmd-demo .
sudo docker run cmd-demo
CMD can be overwritten : sudo docker run cmd-demo echo Override CMD demo

ENTRYPOINT <command>
ENTRYPOINT ["<exec>", "<arg-1>", ..., "<arg-n>"]   (starting point of application)
ENTRYPOINT ["echo", "Dockerfile ENTRYPOINT demo"]

sudo docker run entrypoint-demo with additional arguments
<Would echo these words>
OR sudo docker run --entrypoint="/bin/sh" entrypoint-demo
<THIS WOULD start shell>

ONBUILD <INSTRUCTION>
ONBUILD ADD config /etc/appconfig (triggered when a build is triggered)

.dockerignore
<like git ignore files not needed in build>


*************
example Dockerfile:
###########################################
# Dockerfile to build an Apache2 image
###########################################
# Base image is Ubuntu
FROM ubuntu:14.04
# Author: Dr. Peter
MAINTAINER Dr. Peter <peterindia@gmail.com>
# Install apache2 package
RUN apt-get update && \
apt-get install -y apache2 && \
apt-get clean

sudo docker build -t apache2 .
sudo docker history apache2
sudo docker history

BEST PRACTISE: https://docs.docker.com/articles/dockerfile_best-practices/.

TO RUN AND EXPOSE AN IMAGE:
sudo docker run -d -p 80:80 p0bailey/docker-flask


START OF PREDICTIVE-RECLAIM:
###########################################
# Dockerfile to build Predictive-reclaim
###########################################
# Base image is Ubuntu
FROM amazonlinux
# Author: Dr. Peter
MAINTAINER Ing. M.V. Nossin <nossin_m@schiphol.nl>
# Install wget and zip2
RUN yum install wget  <<< $'y\n'
RUN yum install -y bzip2

#Install Anaconda3
RUN wget https://repo.continuum.io/archive/Anaconda3-4.3.0-Linux-x86_64.sh
RUN bash Anaconda3-4.3.0-Linux-x86_64.sh <<< $'\nyes\n\nyes\n'
#RUN bash Anaconda3-4.3.0-Linux-x86_64.sh <<< $'yes\n'

#RUN mkdir /root/predictive-reclaim
#EXPOSE 80/tcp

#ENTRYPOINT service apache2 start
#your_program <<< $'yes\nno\nmaybe\n'


SOME alias'es to clean:
alias docker_clean_images='docker rmi $(docker images -a --filter=dangling=true -q)'
alias docker_clean_ps='docker rm $(docker ps --filter=status=exited --filter=status=created -q)'

BEFORE WE CONTINUE MAKE SURE DOCKER HOST CAN GET GIT FILES:
<add key because we use private repo>
mkdir git
cd git
git init
git remote add -f origin https://github.com/Schiphol-Hub/projects.git
ls
git config core.sparseCheckout true
echo "luggage/flask/" >> .git/info/sparse-checkout
git pull origin master
#tar -cvf /home/ubuntu/pd.tar /home/ubuntu/git/luggage/flask
TAR WITHOUT PATHS!!:
tar -czvf /home/ubuntu/pd.tar -C /home/ubuntu/git/luggage/flask .

REALLY BUILD:
MAKE SURE SPACE IS EMPTIED AS IT FILLS UP, 
or sudo docker rm <image> OR remopve it all:
sudo rm -rf /var/lib/docker
TO REMOVE IMAGES (TAKES LOTS OF SPACE!)
sudo docker images
sudo docker rmi <image id>

sudo docker build -t predictive-reclaim .
sudo docker images
sudo docker start <image>
sudo run -t -i predictive-reclaim /bin/bash

TO TEST NEW BUILDS WITHOUT REBUILDING ALL:
Change Dockerfile (make sure tar is in $HOME):
FROM predictive-reclaim
ADD pd.tar /root/predictive-reclaim

sudo docker build -t piet .
sudo docker images
sudo docker run -i -t piet /bin/bash
<TO DO: EXPAND WITHOUT PATHS>

FINAL DOCKER FILE
###########################################
# Dockerfile to build Predictive-reclaim
###########################################
# Base image is Ubuntu
FROM amazonlinux
# Author: Dr. Peter
MAINTAINER Ing. M.V. Nossin <nossin_m@schiphol.nl>
# Install os tools
RUN yum install wget  <<< $'y\n'
RUN yum install vim  <<< $'y\n'
RUN yum install -y bzip2

#Install Anaconda3
RUN wget https://repo.continuum.io/archive/Anaconda3-4.3.0-Linux-x86_64.sh
RUN bash Anaconda3-4.3.0-Linux-x86_64.sh <<< $'\nyes\n\nyes\n'

#Set env, TODO LANG has to be set in .bashrc root
ENV PATH /root/anaconda3/bin:$PATH
ENV LANG en_US.UTF-8

#Install Api server
RUN mkdir /root/predictive-reclaim
ADD pd.tar /root/predictive-reclaim
EXPOSE 80/tcp




NOTES:
Install this on Aws if you want ps and ifconfig to work:
yum install -y procps
yum install net-tools
To start flask on internal ip, instead of localhost. However, not seen on docker host!
In case your git files are compatible with the docker image, make the build and run this :
sudo docker run -i -p 172.31.19.178:80:80 -t predictive-reclaim /bin/bash
<start run.sh, make sure host is set to the internal private ip and not localhost>
Ctrlq+ctr p + ctrtl q


EXAMPLE DOCKER-COMPOSE.YML
web: 
  build: .
  command: apachectl -DFOREGROUND
  links:
   - mongo
  ports:
   - 80:80
mongo:
  image: mongo:latest

To build:
docker-compose build
docker-compose up (-d)

docker ps
docker exec -it <container id pred-reclaim> bash
<test python code below>

It works TESTED:
============
```
