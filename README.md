On registry0.lan
================
```
start docker-registry
```

On docker0.lan
==============
```
sudo mkdir -p /etc/docker/certs.d/registry0.lan
sudo mkdir /usr/local/share/ca-certificates/registry0.lan

sudo scp vagrant@registry0.lan:/etc/ssl/registry0.lan/registry0.lan.crt /etc/docker/certs.d/registry0.lan/ca.crt
sudo cp /etc/docker/certs.d/registry0.lan/ca.crt /usr/local/share/ca-certificates/registry0.lan/ca.crt

sudo update-ca-certificates

sudo service docker restart

sudo docker login https://registry0.lan

sudo docker pull ubuntu

sudo docker tag ubuntu registry0.lan/ubuntu

sudo docker push registry0.lan/ubuntu
sudo docker pull registry0.lan/ubuntu

```