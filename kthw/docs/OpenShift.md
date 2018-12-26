
https://github.com/rht-labs/enablement-docs
https://rht-labs.github.io/enablement-docs/#/
https://github.com/redhat-cop/openshift-toolkit

https://github.com/openshift/openshift-ansible-contrib/tree/master/reference-architecture
https://github.com/ContainerSolutions/k8s-deployment-strategies/tree/master/blue-green/multiple-services

https://medium.freecodecamp.org/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882
https://learn.hashicorp.com/vault/
https://github.com/ned1313/Terraform-FotD
https://www.awsgeek.com/
https://www.allthingsdistributed.com/2016/03/10-lessons-from-10-years-of-aws.html

vim /etc/origin/master/master-config.yaml

htpasswd_auth
HTPasswd
file: /etc/origin/htpasswd-users

touch /etc/origin/htpasswd-users

htpasswd -b /etc/origin/htpasswd-users duck stacker

cat /etc/origin/htpasswd-userssys

systemctl restart origin-master-controllers origin-master-api

!export
oc login -u system:admin -n default

export KUBECONFIG=/etc/origin/master/admin.kubeconfig

* Controllers
* Self-service platform
* Polyglot, multi-lingual support
* Automation
* User Interfaces
* Scalability
* Container portability
* Choice of platform


* Containers and images


* Pods and services
* Projects and usersBuild and image streams
* Deployments
* Routes
* Templates


OCP Masters
* API Server
* etcd
* HAProxy
* Controllers

Nodes
* Runtime environments
* Docker service
* Kubelet
* Service proxy

OpenShift Architecture
* OCP-Kbernetes Extentions
* Containerized Services
* Runtimes and xPaaS
* Developer Experience

Hostnames for OpenShift routes are managed by wildcard domains configured using DNSmasq, a powerful DNS utility that can be used to configure local DNS resolution.

DNSMasq
* DNS
* TFTP
* PXE
* Router advertisment
* DHCP

# Lecture: Configuring Docker For OpenShift Container Platform

As we learned during lecture, OpenShift heavily uses Docker for cluster management. Before a cluster can be deployed, Docker will need to be configured with a sufficient amount of dedicated LVM storage to function properly. In this video, we will walk through the process of setting up docker v1.12.6 for use with our OpenShift cluster.

# Install Master and Node
subscription-manager register

subscription-manager attach --pool=$(subscription-manager list --available \
| grep -A18 "Red Hat Enterprise Linux Server Entry Level, Self-support \
| grep "Pool ID" | awk '{print $3}')

subscription-manager attach --pool=$(subscription-manager list --available \
| grep -A32 "Red Hat OpenShift Container Platform \
| grep "Pool ID" | awk '{print $3}')

subscription-manager attach --pool=$(subscription-manager list --available \
| grep -A29 "Red Hat OpenStack Platform | grep -B3 "Unlimited" \
| grep "Pool ID" | awk '{print $3}')

subscription-manager repos --disable=*

subscription-manager repos --enable=rhel-7-server-rpms \
 --enable=rhel-7-server-optional-rpms \
 --enable=rhel-7-server-extras-rpms \
 --enable=rhel-7-server-ose-3.5-rpms \
 --enable=rhel-7-server-openstack-10-rpms
 
yum repolist

yum -y update 

ssh-keygen -f /root/.ssh/ids_rsa -t -rsa -N ''
ssh-copy-id root@node1

# Configure DNS Resolution with dnsmaq
# Install packages dnsmaq and bind-utils
yum -y install dnsmaq bind-utils
Add the following /etc/dnsmasq.config
- address=/ocp.master.example.com/<MASTER_IP>
- address=/ocp.node1.example.com/<NODE1_IP>
- resolv-file=/etc/resolv.dnsmasq

Create /etc/resolv.dnsmasq
- nameserver <YOU_GW_IP>

Edit /etc/resolv.conf 
- search example.com
- nameserver 127.0.0.1

Add entries to /etc/hosts for master and node1
- <MASTER_IP> master master.example.com
- <NODE1_IP> node1 node1.example.com

Start and Enable
- systemctl enable dnsmasq
- systemctl start dnsmasq

Disable firewall
systemctl stop firewalld 
systemctl disable firewalld 

Verify DNS resolution
-  host $(hostname)
- [omg -c 1 test.ocp.master.example.com


yum -y install docker-1.12.6
cp /etc/sysconfig/docker-storage-setup /etc/sysconfig/docker-storage-setup.orig

vim /etc/sysconfig/docker-storage-setup 
> DEVS=vdb
> VG=docker-vg

lvmconf --disable-cluster
docker-storage-setup

cat /etc/sysconfig/docker-storage
DOCKER_STORAGE_OPTIONS="--storage-driver devicemapper --storage-opt dm.fs=xfs --storage-opt dm.thinpooldev=/dev/mapper/docker--vg-docker--pool --storage-opt dm.use_deffered_removal=true --storage-opt dm.use_defferred_deletion=true "

Start and Enable Docker
- systemctl enable docker
- systemctl start docker

OpenShift developers provide a vast series of Ansible playbooks to assist in installation of an OpenShift cluster. The process is made even easier using a module called atomic-openshift-installer. In this video, I will demonstrate how to deploy your own OpenShift cluster on KVM using atomic-openshift-installer.

# atomic-openshift
yum -y install atomic-openshift-docker-excluder \
 atomic-openshift-excluder atomic-openshift-utils bridge-utils bind-utils \
 git iptables-services net-tools wget
 
atomic-openshift-excluder unexclude
atomic-openshift-installer install

systemctl status  | grep openshift


   1. Log in to all nodes, including master, in your OpenShift cluster.
   2. On all nodes, install docker-1.12.6. Do not start Docker after installation completes.
   3. Add --insecure-registry 172.30.0.0/16 to /etc/sysconfig/docker.
   4. If enabled, disable the LVM cluster feature.
   5. Edit docker-storage-setup to use storage mounted on /dev/[x]vdb, then complete the installation using the proper command-line tools.
   6. Verify that storage is configured properly with any preferred tool at your disposal.
   7. Start and enable Docker on all OpenShift nodes.

    Log in to the master via the terminal.
    Before installation, verify that both the master and node1 server have proper DNS resolution configured. 
    On the master, install all needed packages for the OCP v3.5 installation.
    On the master and node1, remove OpenShift exclusions from /etc/yum.conf.
    Install the OpenShift Container Platform using tools provided in the atomic-openshift-utils package. Installation will take 20-45 minutes depending on system resources.
    When the installation completes, re-add exclusions to /etc/yum.conf.
    Verify that all nodes and pods are in "Running/Ready" status.
    Verify that atomic-openshift-master and atomic-openshift-node are running on master server and that atomic-openshift-node is running on the node1 server.

