# Scalable Microservices with Kubernetes

In this course you will learn how to:

- Provision a complete Kubernetes using Google Container Engine
- Deploy and manage Docker containers using kubectl

Kubernetes Version: 1.2.2

## Course Description
Kubernetes is all about applications and in this course you will utilize the Kubernetes API to deploy, manage, and upgrade applications. In this part of the workshop you will use an example application called "app" to complete the labs.

## Setup GCE and Enable Cloud Shell

### Create a Project
Create a new project or use and existing project 
[Creating and managing projects](https://cloud.google.com/resource-manager/docs/creating-managing-projects?visit_id=638332545978322259-4149601563&rd=1)

### Enable Compute Engine and Container Engine APIs
[Introducing the Google API Console](https://developers.googleblog.com/2016/03/introducing-google-api-console.html)

- Compute Engine API
- Container Engine API

### Enable Cloud Shell
[Cloud Shell documentation](https://cloud.google.com/shell/docs)

#### Configure Your Cloud Shell Environment
Create two Cloud Shell Sessions and run the following commands to avoid setting the compute zone.

List available time zones:

`gcloud compute zones list`

Set a time zone example:

`gcloud config set compute/zone europe-west1-d`

#### Download Go
Cloud Shell comes with an installed Go, but it's not the most recent version, so you should perform the steps below to install the latest Go and set GOPATH.

```bash
wget https://storage.googleapis.com/golang/go1.6.2.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.6.2.linux-amd64.tar.gz
echo "export GOPATH=~/go" >> ~/.bashrc
source ~/.bashrc
```

Get the code:

```bash
mkdir -p $GOPATH/src/github.com/udacity
cd $GOPATH/src/github.com/udacity
git clone https://github.com/udacity/ud615
```

Change to app directory:

`cd ud615/app`

### Build and Interact with Monolith

On shell 1 - build the app
Make sure you are in the app directory and build the app:

```bash
cd $GOPATH/src/github.com/udacity/ud615/app
mkdir bin
go build -o ./bin/monolith ./monolith
```

Optional - if you run into errors building your go binaries, you probably need to install the dependencies first by running:

`go get -u`

On shell 1 - run the monolith server:

`sudo ./bin/monolith -http :10080`

On shell 2 - test the app:

```bash
curl http://127.0.0.1:10080
curl http://127.0.0.1:10080/secure
```
On shell 2 - authenticate (password is `password`):

`curl http://127.0.0.1:10080/login -u user`

It prints out the token.

On shell 2 - login and assign the value of the JWT to a variable

`TOKEN=$(curl http://127.0.0.1:10080/login -u user | jq -r '.token')`

Check that it worked:

`echo $TOKEN`

On shell 2 - access the secure endpoint using the JWT:

`curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure`

On shell 2 - check out dependencies

```bash
cd ~/go/src/github.com/udacity/ud615/app
ls vendor 
cat vendor/vendor.json
```

### Refactor to Microservice Architecture
Shell 1 - build and run the hello service

```bash
go build -o ./bin/hello ./hello
sudo ./bin/hello -http 0.0.0.0:10082
```

Shell 2 - build and run the auth service

```bash
go build -o ./bin/auth ./auth
sudo ./bin/auth -http :10090 -health :10091
```

Shell 3 - interact with the auth and hello microservices

```bash
TOKEN=$(curl 127.0.0.1:10090/login -u user | jq -r '.token')
curl -H "Authorization:  Bearer $TOKEN" http://127.0.0.1:10082/secure
```

## Installing apps with native OS tools

Cloud shell - set compute/zone

Note - Google Cloud shell is an ephemeral instance and will reset if you don't use it for more than 30 minutes. That is why you might have to set some configuration values again

```bash
gcloud compute zones list
gcloud config set compute/zone <zone>
```

Cloud shell - launch a new VM instance
```bash
gcloud compute instances create ubuntu \
--image-project ubuntu-os-cloud \
--image ubuntu-1604-xenial-v20160420c 
```

Cloud shell - log into the VM instance
`gcloud compute ssh ubuntu`

VM instance - update packages and install nginx

```bash
sudo apt-get update
sudo apt-get install nginx
nginx -v
```

VM instance - start nginx

`sudo systemctl start nginx`

Check that it's running

```bash
sudo systemctl status nginx

curl http://127.0.0.1
```

## Problem! How to install two versions

Installing and running two versions of nginx on a native OS can be done, but is not straightforward.  

VM instance
Install nginx again

`sudo apt-get install nginx`

Start nginx again and check nginx status

```bash
sudo systemctl start nginx
sudo systemctl status nginx
sudo ps aux | grep nginx
```

Check nginx config file
`cat /etc/init/nginx.conf`

Stop nginx
`sudo systemctl stop nginx`

## Container Overview

- [What are containers and why do you need them?](https://www.cio.com/article/247005/what-are-containers-and-why-do-you-need-them.html)
- [Containers bring a skinny new world of virtualization to Linux](https://www.computerworld.com/article/2698646/containers-bring-a-skinny-new-world-of-virtualization-to-linux.html)

## Installing Images with Docker

Commands to run (on the VM Instance)

Install Docker

`sudo apt-get install docker.io`

Check Docker images

`sudo docker images`

Pull nginx image

```bash
sudo docker pull nginx:1.10.0
sudo docker images
```

Verify the versions match

`sudo dpkg -l | grep nginx`

If your version of nginx from native package and Docker are different, you need to update the VM instance:

```bash
sudo apt-get update
sudo apt-get install nginx
```

## Running Images with Docker

Commands to run on the VM Instance

Run the first instance

`sudo docker run -d nginx:1.10.0`

Check if it's up

`sudo docker ps`

Run a different version of nginx

`sudo docker run -d nginx:1.9.3`

Run another version of nginx

`sudo docker run -d nginx:1.10.0`

Check how many instances are running

`sudo docker ps`

Output:
```bash
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
10408d8f4e7c        nginx:1.10.0        "nginx -g 'daemon of…"   6 seconds ago        Up 6 seconds        80/tcp, 443/tcp     goofy_wozniak
2804e12fcb42        nginx:1.9.3         "nginx -g 'daemon of…"   About a minute ago   Up About a minute   80/tcp, 443/tcp     nifty_cartwright
68b3539175f9        nginx:1.10.0        "nginx -g 'daemon of…"   2 minutes ago        Up 2 minutes        80/tcp, 443/tcp     loving_pike
```

`sudo ps aux | grep nginx`

Output:
```bash
$ sudo ps aux | grep nginx
root      7499  0.0  0.1  31684  5148 ?        Ss   21:13   0:00 nginx: master process nginx -g daemon off;
syslog    7537  0.0  0.0  32068  2948 ?        S    21:13   0:00 nginx: worker process
root      7781  0.0  0.1  31500  4908 ?        Ss   21:14   0:00 nginx: master process nginx -g daemon off;
syslog    7824  0.0  0.0  31876  2740 ?        S    21:14   0:00 nginx: worker process
root      8002  0.0  0.1  31684  5148 ?        Ss   21:15   0:00 nginx: master process nginx -g daemon off;
syslog    8037  0.0  0.0  32068  2872 ?        S    21:15   0:00 nginx: worker process
cubin_p+  8076  0.0  0.0  12948   976 pts/0    S+   21:16   0:00 grep --color=auto nginx
```

What's with the container names?

If you don't specify a name, Docker gives a container a random name, such as "stoic_williams," "sharp_bartik," "awesome_murdock," or "evil_hawking." (Stephen Hawking got no love on this one.)

These are generated from a list of adjectives and names of famous scientists and hackers. The combination of the names and adjectives is random, except for one case. Want to see what the exception is? 

[Check it out in the Docker source code](https://github.com/moby/moby/blob/master/pkg/namesgenerator/names-generator.go)


## Talking to Docker instances

List all running container processes

`sudo docker ps`

Output:
```bash
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
10408d8f4e7c        nginx:1.10.0        "nginx -g 'daemon of…"   6 seconds ago        Up 6 seconds        80/tcp, 443/tcp     goofy_wozniak
2804e12fcb42        nginx:1.9.3         "nginx -g 'daemon of…"   About a minute ago   Up About a minute   80/tcp, 443/tcp     nifty_cartwright
68b3539175f9        nginx:1.10.0        "nginx -g 'daemon of…"   2 minutes ago        Up 2 minutes        80/tcp, 443/tcp     loving_pike
```

For use in shell scripts you might want to just get a list of container IDs (-a stands for all instances, not just running, and -q is for "quiet" - show just the numeric ID):

`sudo docker ps -aq`

Output:

```bash
$ sudo docker ps -aq
10408d8f4e7c
2804e12fcb42
68b3539175f9
```

Inspect the container

You can use either CONTAINER ID or NAMES field, from the above output:

You can use either of the following commands:

```bash
sudo docker inspect 10408d8f4e7c
or
sudo docker inspect goofy_wozniak
```

Connect to the nginx using the internal IP

Get the internal IP address either copying from the full inspect file or by assigning it to a shell variable:

```bash
CN="goofy_wozniak"
CIP=$(sudo docker inspect --format '{{ .NetworkSettings.IPAddress }}' $CN)
curl  http://$CIP
```

You can also get all instance IDs and their corresponding IP addresses by doing this:

`sudo docker inspect -f '{{.Name}} - {{.NetworkSettings.IPAddress }}' $(sudo docker ps -aq)`

Output:

```bash
$ sudo docker inspect -f '{{.Name}} - {{.NetworkSettings.IPAddress }}' $(sudo docker ps -aq)
/goofy_wozniak - 172.17.0.4
/nifty_cartwright - 172.17.0.3
/loving_pike - 172.17.0.2
```

Stop an instance

`sudo docker stop <cid>`

or 

`sudo docker stop $(sudo docker ps -aq)`

Verify no more instances running

`sudo docker ps`

Remove the docker containers from the system

`sudo docker rm <cid>`

or

`sudo docker rm $(sudo docker ps -aq)`

## Creating your own images overview

Dockerfiles contain step-by-step instructions for creating Docker images.

Commands to run on the VM Instance

Install Go
```bash
wget https://storage.googleapis.com/golang/go1.6.2.linux-amd64.tar.gz
rm -rf /usr/local/bin/go
sudo tar -C /usr/local -xzf go1.6.2.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
export GOPATH=~/go
```

Get the app code

```bash
mkdir -p $GOPATH/src/github.com/udacity
cd $GOPATH/src/github.com/udacity
git clone https://github.com/udacity/ud615.git
```

Build a static binary of the monolith app

```bash
cd ud615/app/monolith
go get -u
go build --tags netgo --ldflags '-extldflags "-lm -lstdc++ -static"'
```

You have to explicitly make the binary static. This is really important in the Docker community right now because alpine has a different implementation of libc. So your go binary wouldn't have had the lib it needed if it wasn't static. You created a static binary so that your application could be self-contained.

Create a container for the app

Look at the Dockerfile

`cat Dockerfile`

Output:

```bash
$ cat Dockerfile
FROM alpine:3.1
MAINTAINER Kelsey Hightower <kelsey.hightower@gmail.com>
ADD monolith /usr/bin/monolith
ENTRYPOINT ["monolith"]
```

Build the app container

`sudo docker build -t monolith:1.0.0 .`

List the monolith image

`sudo docker images monolith:1.0.0`

Run the monolith container and get it's IP

```bash
sudo docker run -d monolith:1.0.0
sudo docker inspect <container name or cid>
```

or 

```bash
CID=$(sudo docker run -d monolith:1.0.0)
CIP=$(sudo docker inspect --format '{{ .NetworkSettings.IPAddress }}' ${CID})
```

Test the container

`curl <the container IP>`
 
 or

`curl $CIP`

Important note on security

If you are tired of typing "sudo" in front of all Docker commands, and confused why a lot of examples don't have that, please read the following article about implications on security 
[Why we don't let non-root users run Docker in CentOS, Fedora, or RHEL](https://projectatomic.io/blog/2015/08/why-we-dont-let-non-root-users-run-docker-in-centos-fedora-or-rhel/)


## Create the Other Containers

Create docker images for the remaining microservices - auth and hello.
Repeat the steps you took for monolith.

Build the auth app

```bash
cd $GOPATH/src/github.com/udacity/ud615/app
cd auth
go build --tags netgo --ldflags '-extldflags "-lm -lstdc++ -static"'
sudo docker build -t auth:1.0.0 .
CID2=$(sudo docker run -d auth:1.0.0)
```

Build the hello app

```bash
cd $GOPATH/src/github.com/udacity/ud615/app
cd hello
go build --tags netgo --ldflags '-extldflags "-lm -lstdc++ -static"'
sudo docker build -t hello:1.0.0 .
CID3=$(sudo docker run -d hello:1.0.0)
```

See the running containers
`sudo docker ps -a`

## Public vs Private Registries
A lot of the power of container images stems from the ability to host and download them from cloud registries. This makes it easy to share containers without having to use complex pipelines for distributing them.

When it comes to registries there are a few options. Below are a few different registries.

### Docker Hub
Docker Hub is the registry we’re using in this class. Go ahead and sign up for docker hub and create a repository so that you can follow along with the remaining lessons: 

[Docker Hub](https://hub.docker.com/)

### Quay
Quay is another popular registry because of it’s rich automated workflow for building containers from github. 

[Quay](https://quay.io/)

### Google Cloud Registry
Finally, Google Cloud Registry (GCR) is a strong option for large enterprises. 

[Google Cloud Registry](https://cloud.google.com/container-registry/docs/)

### Comparison of 4 registries
A write up comparing some of the different registries

[Comparing Four Hosted Docker Registries](https://www.suse.com/c/rancher_blog/comparing-four-hosted-docker-registries/)

## Push Images

See all images

`sudo docker images`

Output:

```bash
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello               1.0.0               27173b19289c        13 minutes ago      13.3MB
auth                1.0.0               32556385393e        14 minutes ago      13.3MB
monolith            1.0.0               42966dd98011        2 hours ago         13.4MB
alpine              3.1                 a1038a41fe2b        4 years ago         5.05MB
nginx               1.10.0              16666ff3a57f        7 years ago         183MB
nginx               1.9.3               ea4b88a656c9        8 years ago         133MB
```

Docker tag command help

`docker tag --help`

Output:

```bash
$ docker tag --help

Usage:  docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
```

Add your own tag

`sudo docker tag monolith:1.0.0 <your username>/monolith:1.0.0`

Check the tag was applied:

`sudo docker images`

Output:

```bash
$ sudo docker images
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
hello                  1.0.0               27173b19289c        19 minutes ago      13.3MB
auth                   1.0.0               32556385393e        20 minutes ago      13.3MB
itphilcloud/monolith   1.0.0               42966dd98011        2 hours ago         13.4MB
monolith               1.0.0               42966dd98011        2 hours ago         13.4MB
alpine                 3.1                 a1038a41fe2b        4 years ago         5.05MB
nginx                  1.10.0              16666ff3a57f        7 years ago         183MB
nginx                  1.9.3               ea4b88a656c9        8 years ago         133MB
```

Login and use the docker push command

```bash
sudo docker login
sudo docker push <your username>/example-monolith:1.0.0
```


## Resources
- [Udacity](https://learn.udacity.com/)
- [UD615: Scalable Microservices with Kubernetes](https://github.com/udacity/ud615/tree/master)
- [The Go Programming Language](https://golang.org/)
- [Go learn](https://go.dev/learn/)
- [Google Cloud Shell](https://cloud.google.com/shell/docs/)
- [Docker](https://www.docker.com/)
- [Kubernetes](http://kubernetes.io/)
- [Google Container Engine (GKE)](https://cloud.google.com/container-engine/)
- [The twelve-factor app methodology](https://12factor.net/)
- [JSON Web Tokens (JWT)](https://jwt.io/)
- [How Docker works and how you can use it](https://www.docker.com/what-docker)
- [Introduction to Docker](https://opensource.com/resources/what-docker)
- [Book - Kubernetes: Up and Running, by Joe Beda, Kelsey Hightower, Brendan Burns, Publisher O'Reilly](https://www.oreilly.com/library/view/kubernetes-up-and/9781491935668/)
- [Book - Building Microservices, by Sam Newman, Publisher O'Reilly](https://www.oreilly.com/library/view/building-microservices/9781491950340/)
