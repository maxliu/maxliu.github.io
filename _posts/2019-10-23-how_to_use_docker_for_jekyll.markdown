---
layout: post
title:  "Use docker for jekyll making github log"
date:   2019-10-23 19:58:05 +0000
categories: jekyll docker
---

## The docker file 
```
FROM ubuntu:18.04
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

RUN apt update --fix-missing  && \
    apt-get -y upgrade        && \
    apt install    -y sudo    && \ 
    apt update     -y         && \
	  apt autoclean  -y         && \
    apt autoremove -y	      && \
	  apt install -y ubuntu-desktop  && \
	  apt install -y  \
				git \
	 			curl  && \
	  apt  update -y 

ENV USER=maxliu

RUN useradd  -m -s /bin/bash maxliu 
RUN usermod -aG sudo maxliu
RUN echo "maxliu ALL=(ALL:ALL) NOPASSWD: ALL" > /etc/sudoers.d/maxliu
RUN  echo "America/New_York" > /etc/timezone

USER maxliu 
WORKDIR /home/maxliu

RUN mkdir -p ~/tmp

RUN ["/bin/bash", "-c", "echo -e '12345\12345\n' | sudo passwd root"]
RUN ["/bin/bash", "-c", "echo -e 'maxliu:12345' | sudo chpasswd"]
RUN ["/bin/bash", "-c", "source ~/.bashrc"]

RUN git config --global user.email "xinyulrsm@gmail.com"  && \
	  git config --global user.name "xinyu max liu"

#########################################################
RUN sudo apt-get install ruby-full rails build-essential zlib1g-dev  -y
RUN sudo gem install jekyll -v 3.8.5
RUN sudo apt  -y update 
#########################################################
```

## docker-compose.yml

```
version: '3'

services:
 jekyll:
    container_name: jekyll
    image: .
    command: tail -f /dev/null
    tty: true
    ports:
     - 4000:4000
    volumes:
     - ~/mysite:/home/maxliu/mysite
```

Run the command below to docker bash.

```
$docker exec  -it jekyll  bash
```

in the docker run below.

```
cd mysite
mkdir maxliu.github.io
cd maxliu.github.io
git init
jekyll new . --force
bundler install
jekyll serve --host=0.0.0.0

```
If everthing is OK. push to github

```
git remote add origin https://github.com/maxliu/maxliu.github.io.git
git push -u origin master


```
