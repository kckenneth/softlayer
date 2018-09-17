
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
slcli vs create --datacenter=dal09 --domain=softlayer.com --hostname=kenneth --os=CENTOS_7_64 --cpu=1 --memory=1024 --billing=hourly
```

After you set up the docker vs, you can check if you have an up and running server. 
```
$ slcli vs list
```

If it asks to setup softlayer cli configuration
```
$ slcli config setup
```

`username[] :` 
`API: `
`end_point URL: https://api.softlayer.com/xmlrpc/v3.1/`
`wait: 40`


You just keyed in the username from softlayer API. It's not the softlayer username. When you generate API key, you'll see the 7 digits, probably prefixed with 2 letters. 

### Softlayer vs provision

Once your softlayer server is provisioned in the cloud, you can now see from the list. This will take a few minutes for softlayer to setup. So you won't see anything showing up from the list. You can check by `ready`. If it's ready, it will show `READY`.  

```
$ slcli vs ready 'kenneth'
READY
```

```
$ slcli vs list

:..........:..........:................:...............:............:........:
:    id    : hostname :   primary_ip   :   backend_ip  : datacenter : action :
:..........:..........:................:...............:............:........:
: 61455959 :  kenneth : 169.46.199.168 : 10.173.184.70 :   dal09    :   -    :
:..........:..........:................:...............:............:........:
```

You can also check your server credentials once you got the `id`, which is shown from the list. I blanked the password below. 
```
$ slcli vs credentials 61455959

:..........:..........:
: username : password :
:..........:..........:
:   root   : <blank>  :
:..........:..........:
```

You can check all the details of your server. Using the `--passwords` flag, it will show everything. So make sure you're in a safe environment. 

```
$ slcli vs detail kenneth --passwords

:....................:......................................:
:               Name : Value                                :
:....................:......................................:
:                 id : 61455959                             :
:               guid : b373875b-681c-4653-b667-c44385ac0c08 :
:           hostname : mediator                             :
:             domain : softlayer.com                        :
:               fqdn : mediator.softlayer.com               :
:             status : Active                               :
:              state : Running                              :
: active_transaction : -                                    :
:         datacenter : dal09                                :
:                 os : CentOS                               :
:         os_version : 7.0-64 Minimal for VSI               :
:              cores : 1                                    :
:             memory : 1G                                   :
:          public_ip : 169.46.199.168                       :
:         private_ip : 10.173.184.70                        :
:       private_only : False                                :
:        private_cpu : False                                :
:            created : 2018-09-12T18:32:16-04:00            :
:           modified : 2018-09-12T18:53:03-04:00            :
:              owner : NULL                                 :
:              vlans : :.........:........:.........:       :
:                    : :   type  : number :    id   :       :
:                    : :.........:........:.........:       :
:                    : :  PUBLIC :  1175  : 2439589 :       :
:                    : : PRIVATE :  1253  : 2439591 :       :
:                    : :.........:........:.........:       :
:              users : :..........:..........:              :
:                    : : username : password :              :
:                    : :..........:..........:              :
:                    : :   root   :          :              :
:                    : :..........:..........:              :
:....................:......................................:
```

Once you know how to set up softlayer server, you can deprovision by `vs cancel`. This will take a while to tear down your server. 

```
$ slcli vs cancel 61455959
```

# Installing Docker CE in Ubuntu
We're going to install docker in virtual server. The idea is, we will create Virtual Server (VS) in Softlayer cloud. Once VS is launched, we will install Docker. Remember, this is installing docker so that we can launch docker container on top of the VS we just created. 

1. Launch Virtual Server which will come with Ubuntu OS 
2. Install Docker on Ubuntu OS 
3. Spin up docker container (with many apps, eg, Python) 

```
$ slcli vs create --datacenter=dal09 --domain=softlayer.com --hostname=kenneth --os=CENTOS_7_64 --cpu=1 --memory=1024 --billing=hourly
:.........:......................................:
:    name : value                                :
:.........:......................................:
:      id : 61712999                             :
: created : 2018-09-16T21:38:52-04:00            :
:    guid : 28ad22ce-5ca5-447c-9264-92c63e625544 :
:.........:......................................:
```

```
$ slcli vs list
:..........:..........:...............:................:............:........:
:    id    : hostname :   primary_ip  :   backend_ip   : datacenter : action :
:..........:..........:...............:................:............:........:
: 61712999 : kenneth  : 169.55.204.86 : 10.143.143.197 :   dal09    :   -    :
:..........:..........:...............:................:............:........:
MBP:softlayer kchen$ slcli vs credentials 61712999
:..........:..........:
: username : password :
:..........:..........:
:   root   :          :
:..........:..........:
```


Created ssh keygen to establish connection between host and softlayer server I just created. 
```
$ ssh-keygen -f ~/.ssh/w251 -b 2048 -t rsa -C 'w251_ssh_keys' 
$ slcli sshkey add -f ~/.ssh/w251.pub --note 'added during HW2' w251key
SSH key added: 6f:f0:dc:83:ad:8c:b9:30:25:9d:f1:85:9f:a9:89:88
```

ssh into softlayer virtual server with the IP generated. The password is obtained from credentials call shown above. Although the last two letters were omitted for security purposes. 

```
$ ssh root@169.55.204.86
password: 
[root@kenneth ~]# cd .ssh
# vi w251.pub
```
copied and pasted the w251.pub from the host. Then place the key in authorized_keys

```
# cat w251.pub >>authorized_keys
```
Next time log in will bypass the password requirement
```
$ ssh -i .ssh/w251 root@169.55.204.86
[root@kenneth ~]# 
```

### Setting up Salt Cloud on Softlayer VS 
Remember to change the last identifier, I changed to `w251key` that I used when creating ssh key.

```
slcli vs create -d hou02 --os UBUNTU_LATEST_64 --cpu 1 --memory 1024 --hostname saltmaster --domain someplace.net --key w251key

This action will incur charges on your account. Continue? [y/N]: y
:.........:......................................:
:    name : value                                :
:.........:......................................:
:      id : 61714451                             :
: created : 2018-09-16T22:20:37-04:00            :
:    guid : 989af8d3-ca95-41a3-bb9e-2ff636b38c5f :
:.........:......................................:
```

```
$ slcli vs list
:..........:............:................:................:............:........:
:    id    :  hostname  :   primary_ip   :   backend_ip   : datacenter : action :
:..........:............:................:................:............:........:
: 61712999 :  kenneth   : 169.55.204.86  : 10.143.143.197 :   dal09    :   -    :
: 61714451 : saltmaster : 184.173.59.133 : 10.77.208.197  :   hou02    :   -    :
:..........:............:................:................:............:........:
$ slcli vs credentials 61714451
:..........:..........:
: username : password :
:..........:..........:
:   root   :          :
:..........:..........:
```

Now ssh into 2nd VS I just created. 
```
$ ssh root@184.173.59.133
password:
root@saltmaster:~#
```

## Installing Docker in Virtual Server

```
# apt-get update
apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
    
# add the docker repo    
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
 
# install it
apt-get update
apt-get install docker-ce
```

## Make Docker image 

```
$ vi Dockerfile

FROM ubuntu:16.04

RUN apt-get update
RUN apt-get install -y wget python-setuptools python-dev build-essential
RUN easy_install pip
RUN wget -O - https://repo.saltstack.com/apt/ubuntu/16.04/amd64/latest/SALTSTACK-GPG-KEY.pub | apt-key add -
RUN echo 'deb http://repo.saltstack.com/apt/ubuntu/16.04/amd64/latest xenial main' > /etc/apt/sources.list.d/saltstack.list
RUN apt-get update
RUN apt-get install -y salt-master salt-minion salt-ssh salt-syndic salt-cloud salt-api
RUN mkdir -p /etc/salt/{cloud.providers.d,cloud.profiles.d}
RUN pip install SoftLayer
RUN echo '[softlayer]' > ~/.softlayer
RUN echo 'username =' >> ~/.softlayer
RUN echo 'api_key =' >> ~/.softlayer
RUN echo 'endpoint_url = https://api.softlayer.com/xmlrpc/v3.1/' >> ~/.softlayer
RUN service salt-master start
RUN service salt-syndic start
RUN service salt-api start

ENTRYPOINT ["/bin/bash"]
```

## Configure Salt Cloud

### Making two folders simultaneously
```
mkdir -p /etc/salt/{cloud.providers.d,cloud.profiles.d}
```
### Making softlayer.conf in cloud.providers.d folder
Change API name and KEY. You can either do with `cat` line by line or vi and copy and paste the following. 
```
cat > /etc/salt/cloud.providers.d/softlayer.conf
sl:
  minion:
    master: YOUR_VM_PUBLIC_IP
  user: YOUR_SL_API_ID
  apikey: YOUR_SL_API_KEY
  driver: softlayer
```

### Making softlayer.conf in cloud.profiles.d folder
```
$ cat > /etc/salt/cloud.profiles.d/softlayer.conf
sl_ubuntu_small:
  provider: sl
  image: UBUNTU_LATEST_64
  cpu_number: 1
  ram: 1024
  disk_size: 25
  local_disk: True
  hourly_billing: True
  domain: somewhere.net
  location: dal06
  ```
  
### Run docker in Salt VS
This will create a docker container, volume mounting from two configuration files I just created. Once the container is created, you will see the hostname changes to the container id. 
```
root@saltmaster:~# docker run -it -v /etc/salt/cloud.providers.d:/etc/salt/cloud.providers.d -v /etc/salt/cloud.profiles.d:/etc/salt/cloud.profiles.d salt
root@e22c2b2fcc3f:/# 
```
  
### Run salt cloud in docker container
This will take a few minutes. I saw a bunch of updates, including python and what not. 
```
salt-cloud -p sl_ubuntu_small mytestvs
```
  
### View minion under salt manager in docker container 
```
# salt-key -L
Accepted Keys:
mytestvs
Denied Keys:
Unaccepted Keys:
Rejected Keys:
```

Here, I need to redo the steps because it didn't work when I call master node and netstate. So I deprovision the salt cloud. 

```
# salt-cloud -dy mytestvs
sl:
    ----------
    softlayer:
        ----------
        mytestvs:
            True
```

