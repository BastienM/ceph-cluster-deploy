## Ceph client installation
---

* [Prepare the node](#1-prepare-the-node)
* [Install the client on the selected node](#2-install-the-client-on-the-selected-node)

---

###1 - Prepare the node
</br>

On the node who have to use Ceph, add an user **ceph**.

```bash
$ sudo useradd -d /home/ceph -m ceph
$ sudo passwd ceph
```

And add him to the sudoers.

```bash
$ echo "ceph ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph
$ chmod 0440 /etc/sudoers.d/ceph
```

###2 - Install the client on the selected node
</br>

We can install ceph's client remotely, by using our **master node**.
But first, we have to put our new client node's ip in **/etc/hosts**.

```bash
$ sudo vim /etc/hosts

192.168.68.21    cinder
```

Then, while logged as **ceph**, we send our public key to it.

```bash
$ ssh-copy-id cinder
```

Adding it to our ssh config file can be convenient.

```bash
$ vim ~/.ssh/config

Host cinder
    Hostname cinder
    User ceph
```

We can now execute :

```bash
$ ceph-deploy install cinder
```

At the end of the script, Ceph and all the needed binaries to interact with the cluster will be available (like ceph or rados).

```bash
$ ceph health

HEALTH_OK
```