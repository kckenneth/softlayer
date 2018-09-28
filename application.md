
|Title |  Softlayer Application |
|-----------|----------------------------------|
|Author | Kenneth Chen |
|Utility | IBM, Softlayer, Saltstack |
|Date | 9/10/2018 |

__Synopsis__  

   This is a walk through for how to set up softlayer via CLI. You need to install Docker in your local computer. 

__Softlayer__

Softlayer is a cloud server where you spin up your customized server and launch. It is the same as AWS EC2 provides. However there are a couple of differences between those cloud providers. Here we will explore softlayer in details. First you need to sign up Softlayer account. You need to fill up credit card information so that they will place a temporary charge of $0.25 to your account. Once you
re done creating your account, you can generate API key which you will use to execute your cloud server. 

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

# Week3 
# Mosquitto (MQTT)

```
$ slcli vs create -d hou02 --os UBUNTU_LATEST_64 --cpu 2 --memory 2048 --hostname kenneth --domain someplace.net --billing=hourly
:.........:......................................:
:    name : value                                :
:.........:......................................:
:      id : 62234241                             :
: created : 2018-09-25T18:50:42-04:00            :
:    guid : 8164e05e-e51e-4b95-9220-18236a190d59 :
:.........:......................................:

$ slcli vs list
:..........:..........:................:...............:............:........:
:    id    : hostname :   primary_ip   :   backend_ip  : datacenter : action :
:..........:..........:................:...............:............:........:
: 62234241 : kenneth  : 184.173.26.246 : 10.77.200.159 :   hou02    :   -    :
:..........:..........:................:...............:............:........:

$ slcli vs credentials 62234241
:..........:..........:
: username : password :
:..........:..........:
:   root   :          :
:..........:..........:

$ ssh root@184.173.26.246
# apt-get update
# apt-get install mosquitto-clients
```

### Question: how are block chains relevant for the Internet of Things?

Blockchains are fundamental backbones of transaction ledger. At the same time, without centralized intervention and high fidelity of transaction chain, blockchain offers decentralized interaction among users. Internet of things is basically a legitimate digital interaction between physical products, eg, living room lightbulbs controlled by some physical device which in turn is controlled by human voice. Or remote sensor on home appliances via internet. When many diverse products are digitally connected, or commence interaction, the fidelity of communication becomes paramount. This is where blockchain concept offers a critical step towards maintaining the integrity of internet of things. 

```
# mosquitto_sub -t /applications/in/+/public/# -h 169.44.201.108

{"t":1537917210,"d":[{"PRN":1,"el":8,"az":315,"ss":25,"used":true},{"PRN":8,"el":36,"az":268,"ss":23,"used":true},{"PRN":10,"el":55,"az":51,"ss":32,"used":true},{"PRN":11,"el":20,"az":313,"ss":23,"used":true},{"PRN":14,"el":64,"az":210,"ss":24,"used":false},{"PRN":18,"el":33,"az":313,"ss":26,"used":true},{"PRN":20,"el":32,"az":79,"ss":26,"used":false},{"PRN":21,"el":17,"az":138,"ss":0,"used":false},{"PRN":22,"el":6,"az":278,"ss":0,"used":false},{"PRN":24,"el":11,"az":42,"ss":36,"used":true},{"PRN":27,"el":33,"az":227,"ss":0,"used":false},{"PRN":31,"el":12,"az":167,"ss":23,"used":false},{"PRN":32,"el":88,"az":141,"ss":24,"used":false}]}
{"t":1537917211,"d":[{"PRN":1,"el":8,"az":315,"ss":25,"used":true},{"PRN":8,"el":36,"az":268,"ss":22,"used":true},{"PRN":10,"el":55,"az":51,"ss":32,"used":true},{"PRN":11,"el":20,"az":313,"ss":22,"used":true},{"PRN":14,"el":64,"az":210,"ss":23,"used":false},{"PRN":18,"el":33,"az":313,"ss":26,"used":true},{"PRN":20,"el":32,"az":79,"ss":24,"used":true},{"PRN":21,"el":17,"az":138,"ss":0,"used":false},{"PRN":22,"el":6,"az":278,"ss":0,"used":false},{"PRN":24,"el":11,"az":42,"ss":35,"used":true},{"PRN":27,"el":33,"az":227,"ss":22,"used":false},{"PRN":31,"el":12,"az":167,"ss":23,"used":false},{"PRN":32,"el":88,"az":141,"ss":24,"used":true}]}
```
### Qeustion: Can you recognize some of the messages? What is their meaning?

The output is from GPS GLONASS Satellite. PRN indicates parameter numbers, el for elevation. 

# Week4
# GPFS Setup

### I. Keygen generation to setup ssh key pairs later on virtual servers

In your local host terminal (laptop),
```
$ ssh-keygen -f ~/.ssh/w251 -b 2048 -t rsa 
```

Add your public key to your softlayer account.  
`--note` flag is for softlayer account.  
`w251key` the identifier after that is for later use when we communicate with the virtual servers we will provision later on.  
You don't need to privision vs to add ssh key to your softlayer account. 
```
$ slcli sshkey add -f ~/.ssh/w251.pub --note 'added during HW2' w251key

SSH key added: 49:aa:25:77:c3:b9:32:f5:30:0a:0a:f0:d3:94:09:d0
```   

### II. Provision four vs 
##### This step is being done in local host, laptop

A. Get three virtual servers provisioned, 2 vCPUs, 4G RAM, UBUNTU_16_64, two local disks 25G each, in San Jose. Make sure you attach a keypair. Pick intuitive names such as gpfs1, gpfs2, gpfs3. Note their internal ip addresses.

You can first test run the virtual server by using the flag `--test`. It will show you how much it will charge for each parameters. If you need help with the flag, you can call `slcli vs create --help`. Since I'm creating two local disks, I used `--disk=25` twice because the flag allows multiple occurence. 

Test running
```
$ slcli vs create -d hou02 --os UBUNTU_LATEST_64 --cpu 1 --memory 1024 --hostname saltmaster --domain someplace.net --disk=25 --disk=25 --test  

:................................................................:......:
:                                                           Item : cost :
:................................................................:......:
:                                     1 x 2.0 GHz or higher Core : 0.02 :
:                                                           1 GB : 0.01 :
: Ubuntu Linux 18.04 LTS Bionic Beaver Minimal Install (64 bit)  : 0.00 :
:                                                  25 GB (LOCAL) : 0.00 :
:                                                  25 GB (LOCAL) : 0.00 :
:                                        Reboot / Remote Console : 0.00 :
:                      100 Mbps Public & Private Network Uplinks : 0.00 :
:                                       0 GB Bandwidth Allotment : 0.00 :
:                                                   1 IP Address : 0.00 :
:                                                      Host Ping : 0.00 :
:                                               Email and Ticket : 0.00 :
:                                         Automated Notification : 0.00 :
:          Unlimited SSL VPN Users & 1 PPTP VPN User per account : 0.00 :
:                    Nessus Vulnerability Assessment & Reporting : 0.00 :
:                                              Total hourly cost : 0.04 :
:................................................................:......:
```

I will provision 4 virtual servers. You can also see the flags in short form and long form. Note the hostname for three vs. Also make sure you'd use the identifer for `--key` flag. The identifier you created when you generated keygen on earlier step. 

```
$ slcli vs create -d hou02 --os UBUNTU_LATEST_64 --cpu 1 --memory 1024 --hostname saltmaster --domain someplace.net --key w251key
$ slcli vs create --datacenter=sjc01 --hostname=gpfs1 --domain=softlayer.com --billing=hourly --cpu=2 --memory=4096 --os=UBUNTU_16_64 --disk=25 --disk=25 --key w251key
$ slcli vs create --datacenter=sjc01 --hostname=gpfs2 --domain=softlayer.com --billing=hourly --cpu=2 --memory=4096 --os=UBUNTU_16_64 --disk=25 --disk=25 --key w251key
$ slcli vs create --datacenter=sjc01 --hostname=gpfs3 --domain=softlayer.com --billing=hourly --cpu=2 --memory=4096 --os=UBUNTU_16_64 --disk=25 --disk=25  --key w251key
```
Checking all virtual servers provisioned
```
$ slcli vs list 
:..........:............:................:...............:............:........:
:    id    :  hostname  :   primary_ip   :   backend_ip  : datacenter : action :
:..........:............:................:...............:............:........:
: 62391253 :   gpfs1    : 198.23.88.163  :  10.91.105.3  :   sjc01    :   -    :
: 62391273 :   gpfs2    : 198.23.88.166  :  10.91.105.14 :   sjc01    :   -    :
: 62391283 :   gpfs3    : 198.23.88.162  :  10.91.105.16 :   sjc01    :   -    :
: 62391225 : saltmaster : 184.173.26.246 : 10.77.200.159 :   hou02    :   -    :
:..........:............:................:...............:............:........:
```
### III. Setting up keygen in 3 nodes 
[This step I found is optional, the keygen in the node is much more useful. However if you generate with id_rsa, it might work. Since I already have id_rsa (private key) in my laptop, I don't want to mess up with the existing ones.]  
Since we already provisioned 3 virtual servers or nodes with `--key w251` key, the w251.pub public key is automatically added to each node ~/.ssh/authorized_keys file during privisioning. We also like them to communicate each other without requiring any passwords. So we need to add the private key (i.e., the key in our local host, laptop) to each of the three ndoes. The saltmaster node I created just in case, I need to communicate those 3 nodes from other servers.  
`-i` flag is to specify the private key directory. Since we're connecting to the server, we need password. Instead of typing the password, just providing the key will bypass the password step. We're also directing the private key to be copied into virtual server .ssh directory.

```
$ scp -i ~/.ssh/w251 ~/.ssh/w251 root@198.23.88.166:/root/.ssh/
```
If you come across a warning message, go to 
```
$ cd ~/.ssh
$ vi known_hosts
```
delete ECDSA key  

Logging into each node separately, while in the node following three commands will be executed. It's better to open three separate terminals. There is no default .bash_profile and nodefile. So calling vi will automatically create those file. Every time when you call `vi`, type in the script after the respective commands. In nodefile, depending on the node you're in, you change the `:quorum:` position accordingly. 

```
$ ssh -i ~/.ssh/w251 root@198.23.88.166
```
In the node
```
# vi /root/.bash_profile
export PATH=$PATH:/usr/lpp/mmfs/bin
```
export path is for `mmcrcluster` call in later step. You can also use `PATH=/usr/lpp/mmfs/bin:$PATH`. Since bash_profile is readonly file, you need to source it so that it can be activated. 
```
$ source .bash_profile
```

```
# vi /root/nodefile
gpfs1:quorum:
gpfs2::
gpfs3::

# vi /etc/hosts
10.91.105.14   gpfs1
10.91.105.3    gpfs2
10.91.105.16   gpfs3
```

#### Checking nodes communication 

Now we're going to check all nodes communication without password. So the idea is create the ssh-keygen in one node, which will generate private and public key. Make sure you'd use id_rsa. Not other customized names which doesn't work as expected. 

```
# ssh-keygen -f ~/.ssh/id_rsa -b 2048 -t rsa 
# cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys 
# chmod 600 ~/.ssh/authorized_keys
```

You now copy all files in the `.ssh` directory:  
- authorized_keys  
- id_rsa  
- id_rsa.pub  
to all nodes in the cluster. We have 3 nodes set up in our cluster. So we need to copy all files into the other 2 nodes. 

```
# scp ~/.ssh/* root@198.23.88.166:/root/.ssh/
# scp ~/.ssh/* root@198.23.88.162:/root/.ssh/
```
You open 3 individual terminals. In each terminal, ssh into individual nodes and check if those scp works from gpfs1 node. After all done, try this from each node (each terminal). You'll be sshing from gpfs1 to gpfs1. It doesn't matter. We want to add the information into known_hosts. You'll do those in other two terminals as well. 
```
# ssh gpfs1
yes
exit
# ssh gpfs2
yes
exit
# ssh gpfs3
yes 
exit
```
After you're done, make sure they work. Now in GPFS1 node (terminal), 
```
# vi test.sh
```
Copy the following script
```
#!/bin/bash

# Edit node list
nodes="gpfs1 gpfs2 gpfs3"

# Test ssh configuration
for i in $nodes
do for j in $nodes
 do echo -n "Testing ${i} to ${j}: "
 ssh  ${i} "ssh ${j} date"
 done
done
```
run the script. If all nodes communicate without password (In order for GPFS to work, each node must communicate passwordless), you'd see your script works. If nodes communication fails, you'd see some errors here. 
```
# ./test.sh
root@gpfs1:~# ./test.sh
Testing gpfs1 to gpfs1: Thu Sep 27 23:49:24 UTC 2018
Testing gpfs1 to gpfs2: Thu Sep 27 23:49:24 UTC 2018
Testing gpfs1 to gpfs3: Thu Sep 27 23:49:24 UTC 2018
Testing gpfs2 to gpfs1: Thu Sep 27 23:49:25 UTC 2018
Testing gpfs2 to gpfs2: Thu Sep 27 23:49:25 UTC 2018
Testing gpfs2 to gpfs3: Thu Sep 27 23:49:26 UTC 2018
Testing gpfs3 to gpfs1: Thu Sep 27 23:49:26 UTC 2018
Testing gpfs3 to gpfs2: Thu Sep 27 23:49:27 UTC 2018
Testing gpfs3 to gpfs3: Thu Sep 27 23:49:27 UTC 2018
```

### IV. Installing GPFS in each node

Download GPFS tar
```
# apt-get update
# apt-get install ksh binutils libaio1 g++ make m4
# curl -O http://1D7C9.http.dal05.cdn.softlayer.net/icp-artifacts/Spectrum_Scale_ADV_501_x86_64_LNX.tar
# tar -xvf Spectrum_Scale_ADV_501_x86_64_LNX.tar
```

Then install GPFS with:

```
# ./Spectrum_Scale_Advanced-5.0.1.0-x86_64-Linux-install --silent
# cd /usr/lpp/mmfs/5.0.1.0/gpfs_debs
# dpkg -i *.deb
# /usr/lpp/mmfs/bin/mmbuildgpl
```
The first command is agreeing the license agreement. The `--silent` option to accept the license agreement automatically. If installing all .deb doesn't work, 
```
# dpkg -i gpfs.base*deb gpfs.gpl*deb gpfs.license*.deb gpfs.gskit*deb gpfs.msg*deb gpfs.ext*deb gpfs.compression*deb gpfs.adv*deb gpfs.crypto*deb
# /usr/lpp/mmfs/bin/mmbuildgpl
```
### V. Creating Cluster in One Node (GPFS1)

```
# mmcrcluster -C kenneth -p gpfs1 -s gpfs2 -R /usr/bin/scp -r /usr/bin/ssh -N /root/nodefile
```
You will see this message
```
mmcrcluster: Performing preliminary node verification ...
mmcrcluster: Processing quorum and other critical nodes ...
mmcrcluster: Processing the rest of the nodes ...
mmcrcluster: Finalizing the cluster data structures ...
mmcrcluster: Command successfully completed
mmcrcluster: Warning: Not all nodes have proper GPFS license designations.
    Use the mmchlicense command to designate licenses as needed.
mmcrcluster: Propagating the cluster configuration data to all
  affected nodes.  This is an asynchronous process.
```

To accept the license in all nodes
```
# mmchlicense server -N all
```

To start GPFS 
```
# mmstartup -a
Fri Sep 28 00:36:46 UTC 2018: mmstartup: Starting GPFS ...
```

To check the status of GPFS
```
# mmgetstate -a

 Node number  Node name        GPFS state  
-------------------------------------------
       1      gpfs1            arbitrating
       2      gpfs2            arbitrating
       3      gpfs3            arbitrating
```
Wait a few seconds, and try again, you'd see all nodes are active in GPFS
```
 Node number  Node name        GPFS state  
-------------------------------------------
       1      gpfs1            active
       2      gpfs2            active
       3      gpfs3            active
```
NOTE  
Although you launched GPFS from one node, you can check in other nodes after launch. GPFS launch and installation is different. You need to install GPFS in every node individually. However you can launch GPFS from one node. If there's an error, you can check at `/var/adm/ras/mmfs.log.latest`.

### To lookup GPFS details
```
# mmlscluster

GPFS cluster information
========================
  GPFS cluster name:         kenneth.gpfs1
  GPFS cluster id:           10445230950671284009
  GPFS UID domain:           kenneth.gpfs1
  Remote shell command:      /usr/bin/ssh
  Remote file copy command:  /usr/bin/scp
  Repository type:           CCR

 Node  Daemon node name  IP address    Admin node name  Designation
--------------------------------------------------------------------
   1   gpfs1             10.91.105.3   gpfs1            quorum
   2   gpfs2             10.91.105.14  gpfs2            
   3   gpfs3             10.91.105.16  gpfs3  
```

### Check the mounted disk

```
# fdisk -l | grep Disk | grep bytes

Disk /dev/xvdc: 25 GiB, 26843545600 bytes, 52428800 sectors
Disk /dev/xvda: 25 GiB, 26843701248 bytes, 52429104 sectors
Disk /dev/xvdh: 64 MiB, 67125248 bytes, 131104 sectors
Disk /dev/xvdb: 2 GiB, 2147483648 bytes, 4194304 sectors
```
### To check the mounted directory

```
# mount | grep ' \/ '

/dev/xvda2 on / type ext4 (rw,relatime,data=ordered)
```
Note  
This could mean (I'm still learning), the OS is installed in `xvda` partition 2. So we rather not touch on that disk. We will use the other disk. If you remember, we provisioned two local disks in the beginning. So the other disk would be `xvdc`. So we will use the other disk.  

```
# vi /root/diskfile.fpo
```
Copy the following lines
```
%pool:
pool=system
allowWriteAffinity=yes
writeAffinityDepth=1

%nsd:
device=/dev/xvdc
servers=gpfs1
usage=dataAndMetadata
pool=system
failureGroup=1

%nsd:
device=/dev/xvdc
servers=gpfs2
usage=dataAndMetadata
pool=system
failureGroup=2

%nsd:
device=/dev/xvdc
servers=gpfs3
usage=dataAndMetadata
pool=system
failureGroup=3
```

Setup the disk in all nodes
```
mmcrnsd -F /root/diskfile.fpo
```

Check if it's already setup
```
mmcrnsd -m

 Disk name    NSD volume ID      Device         Node name                Remarks       
---------------------------------------------------------------------------------------
 gpfs1nsd     0A5B69035BAD7D63   /dev/xvdc      gpfs1                    server node
 gpfs2nsd     7F0001015BAD7D64   /dev/xvdc      gpfs2                    server node
 gpfs3nsd     7F0001015BAD7D67   /dev/xvdc      gpfs3                    server node
```

### Create a file system 
This is what I think happening. First we launched 3 virtual servers. We then installed GPFS in each of those nodes. We made sure they can communicate without passwords. Since we want to communicate with those nodes at ease. Also the nodes can also communicate with each other at ease. Basically you're combining all nodes so that you increase your capacity. Conceptually every node is like state in the US. But by combining all states (nodes), you have more capacity (federal), and more resources. Of course you'd need an elected leader, which is a quorum manager. Here GPFS is our quorum manager. 

Now since we have established all disks from every node, we want them to communicate efficiently. Basically if you have an address on your disk, you don't want them to be the same as in other nodes. Otherwise, it would be a disaster. So you'll need to streamline all address across all the nodes you created. So when you create a file system, this will create `inode` system that will become a backbone of GPFS across all nodes in your cluster. 

Replication 1 means the data won't be replicated. If 2, the data will be replicated twice. 

```
mmcrfs gpfsfpo -F /root/diskfile.fpo -A yes -Q no -r 1 -R 1

...
Clearing Inode Allocation Map
Clearing Block Allocation Map
Formatting Allocation Map for storage pool system
Completed creation of file system /dev/gpfsfpo.
```

Check the filesystem
```
mmlsfs all
```

Mount the filesystem in all nodes. Before we mount, you can check before and after filesystem. So first check how the filesystem looks like. 
```
# df -h

Filesystem      Size  Used Avail Use% Mounted on
udev            2.0G     0  2.0G   0% /dev
tmpfs           395M  5.6M  389M   2% /run
/dev/xvda2       24G  3.9G   21G  17% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/xvda1      240M   36M  192M  16% /boot
tmpfs           396M     0  396M   0% /run/user/0
```

Mount the filesystem in all nodes 
```
# mmmount all -a
```
Check the filesystem
```
# df -h 

Filesystem      Size  Used Avail Use% Mounted on
udev            2.0G     0  2.0G   0% /dev
tmpfs           395M  5.6M  389M   2% /run
/dev/xvda2       24G  3.9G   21G  17% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/xvda1      240M   36M  192M  16% /boot
tmpfs           396M     0  396M   0% /run/user/0
gpfsfpo          75G  1.6G   74G   3% /gpfs/gpfsfpo
```

You'd see the `gpfsfpo 75G` after mounting the file system in our cluster. You can also check by 
```
# cd /gpfs/gpfsfpo
# df -h .

Filesystem      Size  Used Avail Use% Mounted on
gpfsfpo          75G  1.6G   74G   3% /gpfs/gpfsfpo
```

Check you create a file and check across all nodes in the cluster
```
# touch aa
# ls -l
total 0
# ssh gpfs2 'ls -l /gpfs/gpfsfpo'
total 0
# ssh gpfs3 'ls -l /gpfs/gpfsfpo'
total 0
```

# Mumbler 

Download google two-gram 27G data set - English version 20090715 from this repository  
http://storage.googleapis.com/books/ngrams/books/datasetsv2.html

```
# mkdir mumbler
# cd mumbler
# wget http://storage.googleapis.com/books/ngrams/books/googlebooks-eng-all-2gram-20090715-{0..99}.csv.zip
```




