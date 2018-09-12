
|Title |  Softlayer Application |
|-----------|----------------------------------|
|Author | Kenneth Chen, Ph.D |
|Utility | |
|Date | 9/10/2018 |

__Synopsis__  

   This is a walk through for how to set up softlayer via CLI. You need to install Docker in your local computer. The procedure will not be covered in this procedure. 

__Softlayer__

Softlayer is a cloud server where you spin up your customized server and launch. It is the same as AWS EC2 provides. However there are a couple of differences between those cloud providers. Here we will explore softlayer in details. First you need to sign up Softlayer account. You need to fill up credit card information so that they will place a temporary charge of $0.25 to your account. Once you created your account, you can generate API key which you will use to execute your cloud server. 

__Dockerfile__  

```
$ cd ~
$ mkdir softlayer
$ cd softlayer
$ vi Dockerfile
```
click i to insert the following text. I changed YOUR_SL_API_ID and YOUR_SL_API_KEY accordingly.

```
FROM ubuntu:16.04

RUN apt-get update 
RUN apt-get install -y \
    python \
    python-pip \
    python-setuptools \
    python-dev \ 
    openssh-client
RUN pip install SoftLayer
RUN echo '[softlayer]' > ~/.softlayer
RUN echo 'username = YOUR_SL_API_ID' >> ~/.softlayer
RUN echo 'api_key = YOUR_SL_API_KEY' >> ~/.softlayer
RUN echo 'endpoint_url = https://api.softlayer.com/xmlrpc/v3.1/' >> ~/.softlayer
RUN ssh-keygen -b 2048 -t rsa -f ~/.ssh/id_rsa -q -N ""

ENTRYPOINT ["/bin/bash"]
```

Since we're using the Ubuntu and we don't need any dependencies (eg. Redis) and web app (eg. Flask), we just created Dockerfile and build the image on it. `-t` or `--tag`. 

```
$ docker build -t slcli --file Dockerfile
```

Running the docker image  
`-it` for interactive and terminal flag 
```
$ docker run -it slcli
```

# Working with Softlayer Virtual Server (vs) 

I changed domain and hostname accordingly. 
`--domain=softlayer.com` 
`--hostname=kenneth`

```
slcli vs create --datacenter=dal09 --domain=<somedomain> --hostname=<some hostname> --os=CENTOS_7_64 --cpu=1 --memory=1024 --billing=hourly
```
