
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
`-it` for interactive and terminal flag. Here I'm spinning docker without any volume mounted. 
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

After you set up the docker vs, you can check if you have an up and running server. 
```
$ slcli vs list
```

If it asks to setup softlayer cli configuration
```
$ slcli config setup
```

username[] : 
API: 
end_point URL: https://api.softlayer.com/xmlrpc/v3.1/
wait: 40


You just keyed in the username from softlayer API. It's not the softlayer username. When you generate API key, you'll see the 7 digits, probably prefixed with 2 letters. 

### Softlayer vs provision

Once your softlayer server is provisioned in the cloud, you can now see from the list

```
$ slcli vs list
```

:..........:..........:................:...............:............:........:
:    id    : hostname :   primary_ip   :   backend_ip  : datacenter : action :
:..........:..........:................:...............:............:........:
: 61455959 :  kenneth : 169.46.199.168 : 10.173.184.70 :   dal09    :   -    :
:..........:..........:................:...............:............:........:
