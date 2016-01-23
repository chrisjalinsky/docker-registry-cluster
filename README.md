Docker Registry
===============
This playbook will be the building block for future development with containerization. The idea is to provision a set of hosts to work in Docker environments. The Vagrantfile by default builds 3 Ubuntu 14.04 LTS hosts: The roles expose many default variables to control output. view those a each roles default/main.yml. You can override in the playbook.

* docker0.lan
* docker1.lan
* registry0.lan

To build the cluster:

```
vagrant up
```
Once provisioning in complete, Ansible with display play recap, if none failed, the hosts are ready:

```
PLAY RECAP ******************************************************************** 
docker0.lan                : ok=15   changed=15   unreachable=0    failed=0   
docker1.lan                : ok=15   changed=15   unreachable=0    failed=0   
registry0.lan              : ok=30   changed=30   unreachable=0    failed=0 
```

If you make changes to the Vagrantfile, like adding another host, or the Ansible playbook, update the cluster:

```
vagrant provision
```


SSH into each box with vagrant like so:

```
vagrant ssh registry0.lan
```


registry0.lan
=============

An upstart script is present in /etc/init/docker-registry.conf which controls a docker-compose script located at /docker-registry/docker-compose.yml.
The playbook deploys a script which generates self signed SSL certificates based on the hostname.

```
sudo start docker-registry
```

On docker0.lan and docker1.lan
==============

Log into each box like so:

```
vagrant ssh docker0.lan
vagrant ssh docker1.lan
```


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