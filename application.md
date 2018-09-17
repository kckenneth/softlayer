
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

