
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

After you set up the docker vs, you can check if you have an up and running server. The primary_ip and backend_ip creation might be a bit slow. 
```
$ slcli vs list
:..........:..........:............:............:............:.............:
:    id    : hostname : primary_ip : backend_ip : datacenter :    action   :
:..........:..........:............:............:............:.............:
: 62139488 : kenneth  :     -      :     -      :   dal09    : Assign Host :
:..........:..........:............:............:............:.............:
```

You can check softlayer configuration. 
```
$ slcli config show
:..............:..................................................................:
:         Name : Value                                                            :
:..............:..................................................................:
:     Username : SL1234567                                                        :
:      API Key : XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX  :
: Endpoint URL : https://api.softlayer.com/xmlrpc/v3.1                            :
:      Timeout : 40.0                                                             :
:..............:..................................................................:
```

If it asks to setup softlayer cli configuration.
```
$ slcli config setup
```

`username[] :`  
`API: `  
`end_point URL: https://api.softlayer.com/xmlrpc/v3.1/`  
`wait: 40`


You just keyed in the username from softlayer API. It's not the softlayer username. When you generate API key, you'll see the 7 digits, probably prefixed with 2 letters; SL. 

### Softlayer vs (Virtual Server) provision

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
:..........:..........:...............:..............:............:........:
:    id    : hostname :   primary_ip  :  backend_ip  : datacenter : action :
:..........:..........:...............:..............:............:........:
: 62139488 : kenneth  : 169.54.221.19 : 10.142.25.90 :   dal09    :   -    :
:..........:..........:...............:..............:............:........:
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
#### ssh parameters
`-f` for file you're creating. Some people name the file as x_rsa to be more descriptive about the protocol used.  
`-b` for bits. The default is 2048.  
`-t` for type declaration. It is rsa here. Other protocols = rsa1, dsa, ecdsa  
`-C` for comment. This is optional tag.  

#### Setting up ssh key pair in SL VS 
##### (1) Logging into SL VS 

ssh into softlayer virtual server with the IP generated. The password is obtained from credentials call shown above. 
```
$ ssh root@169.54.221.19
password: 
[root@kenneth ~]# cd .ssh
# vi authorized_keys 
```
Open another terminal, go to your w251.pub by `$ cd ~/.ssh` and `more w251.pub`. Copy the output and paste in the SL's authorized_keys file. Next time log in will bypass the password requirement
```
$ ssh -i .ssh/w251 root@169.55.221.19
[root@kenneth ~]# 
```
##### (2) Directly pasting rsa pub key into SL VS 
You can also directly paste the rsa pub key into SL VS. You are now at your local computer. However, it might not work for Mac OS because I found some Linux commands do not work in Mac OS, especially piping as well. 
```
$ cat ~/.ssh/w251.pub | ssh root@169.55.221.19 "mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys && chmod -R go= ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### Setting up Salt Cloud on Softlayer VS 
Remember to change the last identifier, I changed to `w251key` that I used when creating ssh key.

```
slcli vs create -d hou02 --os UBUNTU_LATEST_64 --cpu 1 --memory 1024 --hostname saltmaster --domain someplace.net --key w251key

This action will incur charges on your account. Continue? [y/N]: y
:.........:......................................:
:    name : value                                :
:.........:......................................:
:      id : 62141128                             :
: created : 2018-09-23T10:17:12-04:00            :
:    guid : 00ba177c-57c4-4ffb-b355-c1a0d17132a2 :
:.........:......................................:
```

```
$ slcli vs list
:..........:............:................:..............:............:........:
:    id    :  hostname  :   primary_ip   :  backend_ip  : datacenter : action :
:..........:............:................:..............:............:........:
: 62139488 :  kenneth   : 169.54.221.19  : 10.142.25.90 :   dal09    :   -    :
: 62141128 : saltmaster : 184.173.51.156 : 10.77.243.48 :   hou02    :   -    :
:..........:............:................:..............:............:........:
$ slcli vs credentials 62141128
:..........:..........:
: username : password :
:..........:..........:
:   root   :          :
:..........:..........:
```

Now ssh into 2nd VS I just created. The first VS was CENTOS and the second VS is Ubuntu. Since we're going to install Docker into Ubuntu, we ssh into the 2nd VS. 
```
$ ssh root@184.173.59.133
password:
root@saltmaster:~#
```
## Installing Softlayer API in Softlayer VS 

This is tricky and confusing as hell. But I will detail here as much as I understand. You created a Virtual Server (VS) in Softlayer. Now what's important is your VS. This is where all your work will be processed, your job, your data processing. Softlayer is just a trademark that provides cloud services, including the virtual server we just created. So after you created a VS in Softlayer, you can think of your VS as your own personal computer. You can do anything, install the apps, upgrade the apps as necessary. 

Now in this VS, you'd definitely need python, the most versatile program. You also need some work scheduler such as salt master (I assume). You also like to install Softlayer api so that your VS can connect to your Softlayer account. It sounds confusing at first. Yes, you're right. You're communicating from the VS that you created in Softlayer, to your Softlayer account. 

### I. Option 1: Docker 
How to install those necessary apps and the apps that you like to have in your VS? You could build a Docker image where you can specify what applications you want to install. So by defining the apps to install in Docker image, once you run the Docker, it will automatically retrieve all those apps and would install in your VS. Of course, the first step is to install the base Docker program in your VS. So I detailed how to install Docker in VS below, titled `Installing Docker in Virtual Server`. Sounds neat, right? The problem is, sometimes, even if you specify in Docker image to install some applications, they might not retrieve and install as expected. However, it's good to install Docker in your VS so that you can run any containers down the road. 

### II. Option 2: Install Softlayer in VS

We are now in one of the VS we created in Softlayer. So denoted by `#`. To install `pip` and necessary upgrade, 
```
# sudo apt-get update && sudo apt-get -y upgrade
# sudo apt-get install python-pip
# pip install --upgrade pip
# sudo pip install Softlayer
```
#### Install Salt Stack in VS manually

Run the following command to import the SaltStack repository key:
```
# wget -O - https://repo.saltstack.com/apt/ubuntu/16.04/amd64/latest/SALTSTACK-GPG-KEY.pub | sudo apt-key add -
```

Save the following file to /etc/apt/sources.list.d/saltstack.list:
```
# cat > /etc/apt/sources.list.d/saltstack.list
deb http://repo.saltstack.com/apt/ubuntu/16.04/amd64/latest xenial main
```
Use CTRL-D to complete changes to the file and return to the # prompt.

Run 
```
# sudo apt-get update
```

Install the salt-minion, salt-master, or other Salt components:

```
# sudo apt-get install -y salt-master salt-minion salt-ssh salt-syndic salt-cloud salt-api
```

Start all services:
```
# service salt-master start
# service salt-syndic start
# service salt-api start
```

## Installing Docker in Virtual Server

Since we're not at VS terminal, I presented with a pound or hash sign `#`, instead of a dollar sign `$`.  
Double pound signs `##` is for personal comments. 
```
# apt-get update
# apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
    
## add the docker repo    
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
 
## install it
apt-get update
apt-get install docker-ce
```

## Make Docker image 
#### 1. Dockerfile 
```
# vi Dockerfile

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
#### 2. Build Docker image 
```
# docker build --tag salt --file Dockerfile .
```

## Configure Salt Cloud

### Making two folders simultaneously
```
# mkdir -p /etc/salt/{cloud.providers.d,cloud.profiles.d} 
``` 

### Making softlayer.conf in cloud.providers.d folder
Change API name and KEY. You can either do with `cat` line by line or vi and copy and paste the following. 
```
# cat > /etc/salt/cloud.providers.d/softlayer.conf
sl:
  minion:
    master: YOUR_VM_PUBLIC_IP
  user: YOUR_SL_API_ID
  apikey: YOUR_SL_API_KEY
  driver: softlayer
```

### Making softlayer.conf in cloud.profiles.d folder
```
# cat > /etc/salt/cloud.profiles.d/softlayer.conf
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

# Privision of Minion 

You must have installed all following components already.  
1. Softlayer  
2. saltstack  
3. Docker  

### 1. Softlayer 
Your base account from which any charges of the minion servers provision will be deducted. This is also where you need to create salt configuraion where any minion you'd create, what specs you want to create for your minion server, like CPU, RAM, so on. 

### 2. saltstack
Basically, I found saltstack is kind of like a work scheduler or manager, that you can control any of the minion servers you create. Remember, the server you will create is considered 'minion' because you're creating from your top-level server 'saltmaster' which is also you created earlier on. When you provision your minion server, you need to use `salt-cloud`, not `slcli vs create`. If you use `slcli vs create`, it will only create another standalone server. 

### 3. Docker
Docker is if you want to create a container for any of the downstream jobs where you'd need python, redis, kafka, etc. 

### Run salt cloud in docker container
This will take a few minutes. I saw a bunch of updates, including python and what not. Be patient. 
```
# salt-cloud -p sl_ubuntu_small mytestvs
```
Note
I create 'mytestvs' (a new VS) in docker container. It takes a while. After done, I checked 
```
# salt 'mytestvs' network.interface_ip eth1  
# salt '*' status.netstats
```
It didn't work. So I exited the container, (removed the unused docker container by `docker rm -f $(docker ps -aq)`). In the saltmaster VS, I created a new VS named 'mytestvs1' 

```
# salt-cloud -p sl_ubuntu_small mytestvs1
```

This is the output if your minion privision is successful. 
```
mytestvs1:
    ----------
    accountId:
        1729689
    createDate:
        2018-09-23T13:01:39-04:00
    deployed:
        True
    domain:
        somewhere.net
    fullyQualifiedDomainName:
        mytestvs1.somewhere.net
    globalIdentifier:
        7340b51f-5d40-4c11-9b6b-cb2c109ac2ef
    hostname:
        mytestvs1
    id:
        62146822
    lastPowerStateId:
    lastVerifiedDate:
    maxCpu:
        1
    maxCpuUnits:
        CORE
    maxMemory:
        1024
    metricPollDate:
    modifyDate:
    password:
        XXXXXXX
    provisionDate:
    public_ip:
        184.173.51.155
    startCpus:
        1
    statusId:
        1001
    username:
        root
    uuid:
        3cb319a5-27cc-4c8e-aa7c-e471dff29861
```
 
I checked how many VS I created. 
```
root@saltmaster:~# slcli vs list
:..........:............:................:..............:............:........:
:    id    :  hostname  :   primary_ip   :  backend_ip  : datacenter : action :
:..........:............:................:..............:............:........:
: 62139488 :  kenneth   : 169.54.221.19  : 10.142.25.90 :   dal09    :   -    :
: 62146204 :  mytestvs  : 184.173.51.157 : 10.77.243.55 :   hou02    :   -    :
: 62146822 : mytestvs1  : 184.173.51.155 : 10.77.243.42 :   hou02    :   -    :
: 62141128 : saltmaster : 184.173.51.156 : 10.77.243.48 :   hou02    :   -    :
:..........:............:................:..............:............:........:
```

### View minion (a new VS) under salt manager 
```
# salt-key -L
Accepted Keys:
mytestvs
Denied Keys:
Unaccepted Keys:
Rejected Keys:
```
#### Option
If you have created more minions and they are not in your Accepted Keys, you can add them by
```
# salt-key --accept=<key>
# salt-key --accept-all
```
https://docs.saltstack.com/en/getstarted/fundamentals/install.html

I'm in one of the VS I initially created 'saltmaster'. 
```
root@saltmaster:~# salt 'mytestvs1' network.interface_ip eth1
mytestvs1:
    184.173.51.155
```
Checking network status 
```
root@saltmaster:~# salt '*' status.netstats
mytestvs1:
    ----------
    IpExt:
        ----------
        InBcastOctets:
            0
        InBcastPkts:
            0
        InCsumErrors:
            0
        InECT0Pkts:
            0
        InECT1Pkts:
            0
```

### Deprovisioning the minion in salt-cloud

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
Check if there's any Accepted keys after deprovisioning. 
```
# salt-key -L
Accepted Keys:
Denied Keys:
Unaccepted Keys:
Rejected Keys:
```

# Miscellaneous Notes

If you're connecting to VS, you use `$ ssh root@169.55.204.86`. You cannot use the hostname `kenneth` although you use `kenneth` as your hostname when you provision VS in the first place. It's because of the DNS system. If you want to use the hostname, you can do so by updating in `hosts` folder. 
```
$ vi /etc/hosts/
169.55.204.86 kenneth
```

After you updated the hosts, you can ssh next time by using 
```
$ ssh root@kenneth
```
