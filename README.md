# docker auditd

Strongly inspired by rcip-docker-openshift-monitoring

Docker build using rcip-openshift-ansible repo in order to build an auditd docker image

## What
The repository provide a Dockerfile in order to build an auditd docker image. For example on Atomic host we can't setup packages and tools needed to run inside a docker container.
This image includes sensu-client, collectd, logstash (setuped by the rcip-openshift-ansible playbook)

  [1] : https://github.com/redhat-cip/rcip-openshift-ansible

## Step 1 : Ansible inventory
Copy your ansible inventory file (used during the openshift setup) or use the ansible_hosts.example

```bash
ansible_hosts
 ```

## Step 2 : Build your image

Edit the dockerfile_start.sh and change the HOSTNAME to match a node in every groups (masters, nodes, etcd ...)
```bash
cat dockerfile_start.sh

docker build \
--no-cache \
--build-arg SUBSCRIPTION_USER=user \
--build-arg SUBSCRIPTION_PASSWORD=password \
--build-arg SUBSCRIPTION_POOL=poolid \
--build-arg HOSTNAME=master1.ose-example.com \
-t ndox/atomic-auditd:v1 \
.
 ```

And run your build
```bash
bash dockerfile_start.sh
 ```

## Step 3 : Export your image

Save in a tar file
```bash
docker ps
docker save ndox/atomic-auditd > /tmp/atomic-auditd.tar
 ```

Or on Docker hub

```bash
docker tag <imageID> docker.io/ndox/atomic-auditd
docker login --username user --email user@email.com docker.io
docker push docker.io/ndox/atomic-auditd
 ```

## Step 4 : Deploy your image

Copy the image on your nodes
```bash
scp /tmp/atomic-auditd.tar master1.ose-example.com:/tmp/
 ```
From you node, import and tag your image
```bash
docker load < atomic-auditd.tar
docker images
docker tag <imageID> ndox/atomic-auditd
 ```

Or from Docker hub

```bash
docker pull docker.io/ndox/atomic-auditd
 ```


## Step 5 : Run your image

```bash
service atomic-auditd start
 ```

The service declaration is push by the rcip-openshift-ansible [2] during the post.yml playbook

  [2] : https://github.com/redhat-cip/rcip-openshift-ansible/blob/master/roles/common/templates/etc/systemd/system/openshift-monitoring-client.service.j2