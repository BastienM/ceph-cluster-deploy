## Ceph cluster installation
---

* [Prepare the nodes](#1-prepare-the-nodes)
* [Prepare the master node](#2-prepare-the-master-node)
* [Install the Ceph deployment package](#3-install-the-ceph-deployment-package)
* [Deploy Ceph on our cluster](#4-deploy-ceph-on-our-cluster)

---

###1 - Prepare the nodes
</br>

All the nodes needs a **new user** for Ceph to be created.

```bash
$ sudo useradd -d /home/ceph -m ceph
$ sudo passwd ceph
```

Next, our **ceph** user must be add to the sudoers.

```bash
$ echo "ceph ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph
$ chmod 0440 /etc/sudoers.d/ceph
```

###2 - Prepare the master node
</br>

First we register our nodes's ip in the **hosts file**.

```bash
$ sudo vim /etc/hosts

10.75.35.234    ceph-master
10.75.35.236    ceph-node01
10.75.35.239    ceph-node02
```

Then we generate a **rsa key** for the ceph user.

```bash
$ sudo su - ceph
$ ssh-keygen -t rsa
```

A password-less ssh login is need by Ceph, so we send our new public key to our nodes.

```bash
$ ssh-copy-id ceph@cept-master
$ ssh-copy-id ceph@cept-node01
$ ssh-copy-id ceph@cept-node02
```

For more commidity we will create a **config** file for the SSH client.

```bash
$ vim ~/.ssh/config

Host ceph-master
    Hostname ceph-master
    User ceph

Host ceph-node01
    Hostname ceph-node01
    User ceph

Host ceph-node02
    Hostname ceph-node02
    User ceph
```

###3 - Install the Ceph deployment package
</br>

The deployment can be done from the master node.

To use Ceph's repository we must first import their **GPG key**.

```bash
$ wget -q -O- 'https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc' | sudo apt-key add -
```

Then we add the repository to our sources list.

```bash
$ echo deb http://ceph.com/debian-firefly/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
```

And we ask aptitude to take the changes.

```bash
$ sudo apt-get update
```

Which allow us to now install ceph's package.

```bash
$ sudo apt-get install ceph-deploy ntp
```

You can check if the package has been successfully installed.

```bash
$ sudo dpkg -l | grep ceph-*
ii  ceph-deploy                         1.5.28trusty                     all          Ceph deploy is an easy to use configuration tool
```

###4 - Deploy Ceph on our cluster
</br>

The procedure to deploy Ceph is fairly easy with the package **ceph-deploy** we previously installed on the master node.

```bash
$ sudo su - ceph

# install the needed packages on the nodes
$ ceph-deploy install ceph-master ceph-node01 ceph-node02

# create a new configuration
$ ceph-deploy new ceph-master ceph-node01 ceph-node02

# add the monitor role to the nodes
$ ceph-deploy mon create ceph-master ceph-node01 ceph-node02

# get the auth keys of the nodes
$ ceph-deploy gatherkeys ceph-master ceph-node01 ceph-node02

# create the OSD (storage service) on our nodes
$ ceph-deploy osd create ceph-master:/dev/sdb ceph-node01:/dev/sdb ceph-node02:/dev/sda ceph-node02:/dev/sdb

# then we distribute our configuration  as well as our admin key to the nodes
$ ceph-deploy admin ceph-master ceph-node01 ceph-node02

# the keyring need the proper permissions to be used
$ sudo chmod +r /etc/ceph/ceph.client.admin.keyring

# and we add the MDS role (metadatas) to our master node
$ ceph-deploy mds create ceph-master

# let's check our cluster health
$ ceph health
```