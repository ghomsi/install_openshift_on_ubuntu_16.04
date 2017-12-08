# install_openshift_on_ubuntu_16.04
openshift installation demo guide on ubuntu 16.04 amd64

to know if your OS is **amd64** do:
```
uname -a
```

Get Origin Up and Running From the Comfort of Your Own Laptop
This image is based off of OpenShift Origin and is a fully functioning OpenShift instance with an integrated Docker registry. The
intent of this project is to allow Web developers and other interested parties to run OpenShift on their own computer.


Step 1 - Install and configure docker on your machine
---------------------------------------

* Update the **apt** package index:
```
sudo apt-get update
```
* Install packages to allow **apt to use a repository over HTTPS**:
```  
    sudo apt-get install \
    apt-transport-https 
    ca-certificates \
    curl \
    software-properties-common
 ```   
* Add Docker’s official **GPG key**:
 ```
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
 
 ```
* Use the following command to set up the stable repository. You always need the stable repository, even if you want to install builds from the edge or test repositories as well. To add the edge or test repository, add the word edge or test (or both) after the word stable in the commands below.
#### amd64
```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```
#### install DOCKER CE
* Update the apt package index.

```
sudo apt-get update
```

* Install the latest version of Docker CE, or go to the next step to install a specific version. Any existing installation of Docker is replaced.
```
sudo apt-get install docker-ce
```
*  adding yourself to the user group 'docker'
```
sudo usermod -aG docker ${USER}
```
you can run the following command to find out what groups you belong to
```
groups $USER
```

Supporting Articles
-------------------
- [Get Docker CE for Ubuntu](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)

NEXT
----
#### configure DOCKER deamon with an insecure registry parameter of 172.30.0.0/16

* edit /etc/default/docker, add line
```
DOCKER_OPTS="--insecure-registry 172.30.0.0/16"
```
But for some reason, docker will not take this after restart.
This is fixed by editing systemd unit for docker at
```
sudo nano /etc/systemd/system/multi-user.target.wants/docker.service
```
  Find
    ```
    /usr/bin/dockerd -H fd://
    ```
  Replace with 
    ```
    /usr/bin/dockerd --insecure-registry 172.30.0.0/16 -H fd://
    ``` 
* Now reload systemd
```
systemctl daemon-reload
```
* Restart docker
```
systemctl stop docker
systemctl start docker
```

Supporting Articles
-------------------
- [–insecure-registry argument on the Docker daemon](https://blog.hostonnet.com/did-not-detect-insecure-registry-argument-docker-
daemon)

NEXT
----

#### Ensure that your firewall allows containers access to the OpenShift master API (8443/tcp) and DNS (53/udp) endpoints.
* Determine the Docker bridge network container subnet:
```
docker network inspect -f "{{range .IPAM.Config }}{{ .Subnet }}{{end}}" bridge
```
You should get a subnet like: **172.30.0.0/16**

* Create a new firewalld zone for the subnet and grant it access to the API and DNS ports:
```
firewall-cmd --permanent --new-zone dockerc
firewall-cmd --permanent --zone dockerc --add-source 172.17.0.0/16
firewall-cmd --permanent --zone dockerc --add-port 8443/tcp
firewall-cmd --permanent --zone dockerc --add-port 53/udp
firewall-cmd --permanent --zone dockerc --add-port 8053/udp
firewall-cmd --reload
```

Supporting Articles
-------------------
- [Local Cluster Management](https://github.com/openshift/origin/blob/master/docs/cluster_up_down.md#linux)

Step 2 - Install openshift origin
----------------------------------

* Download the Linux oc binary from:

- [openshift-origin-client-tools-VERSION-linux-64bit.tar.gz](https://github.com/openshift/origin/releases)

* place it in your path
go to the directory using **cd** command then do:
``` 
export PATH="$(pwd)":$PATH
```

* Open a terminal with a user that has permission to run Docker commands and run:
``` 
oc cluster up
```
Enjoy Openshift!!
-----------------

* To stop your cluster, run:
``` 
oc cluster down
```
Supporting Articles
-------------------
- [Local Cluster Management](https://github.com/openshift/origin/blob/master/docs/cluster_up_down.md#linux)

![OpenShiftOrigin local running](https://github.com/ghomsi/install_openshift_on_ubuntu_16.04/raw/master/images/openshift.png)
