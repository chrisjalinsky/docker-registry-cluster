Docker Registry
===============
This playbook will be the building block for future development with containerization. The idea is to provision a set of hosts to work in Docker environments. The Vagrantfile by default builds 3 Ubuntu 14.04 LTS hosts:

* docker0.lan
* docker1.lan
* registry0.lan



registry0.lan
================
```
start docker-registry
```

On docker0.lan and docker1.lan
==============

To begin using the registry, several steps must be performed. Since we are using self signed SSL certificates, we must share these with our cluster nodes. Docker Registry expects these to exist in a specific location, so the steps below handle that:

```
sudo mkdir -p /etc/docker/certs.d/registry0.lan
sudo mkdir /usr/local/share/ca-certificates/registry0.lan

sudo scp vagrant@registry0.lan:/etc/ssl/registry0.lan/registry0.lan.crt /etc/docker/certs.d/registry0.lan/ca.crt
sudo cp /etc/docker/certs.d/registry0.lan/ca.crt /usr/local/share/ca-certificates/registry0.lan/ca.crt

sudo update-ca-certificates

sudo service docker restart

```
Using the Registry from docker0.lan or docker1.lan
=======================

Here's a simple use case:

```
sudo docker login https://registry0.lan
```

There are 2 test users defined, so sign in with the credentials defined in vars/makevault.yml. This should be converted to an ansible-vault, but for testing purposes, it's left open.

At the login prompt:

```
username: testuser
password: pass
email: me@here.com
```

Once logged in, use the registry:

```
sudo docker pull ubuntu

sudo docker tag ubuntu registry0.lan/ubuntu

sudo docker push registry0.lan/ubuntu

sudo docker pull registry0.lan/ubuntu

```