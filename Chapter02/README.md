# Chapter 2  Working with Ceph BlockWorking with Ceph Block Device

## Configuring Ceph client

Any regular Linux host (RHEL or Debian-based) can act as a Ceph client. The client interacts with the Ceph storage cluster over the network to store or retrieve user data. Ceph RBD support has been added to the Linux mainline kernel, starting with 2.6.34 and later versions.

### How to do it...

As we did earlier, we will set up a Ceph client machine using Vagrant and VirtualBox. We will use the same Vagrantfile that we cloned in the last chapter i.e. Chapter 1 , Ceph - Introduction and Beyond. Vagrant will then launch a CentOS 7.3 virtual machine that we will configure as a Ceph client:

1.  From the directory where we cloned the Ceph-Designing-and-Implementing-Scalable-Storage-Systems GitHub repository, launch the client virtual machine using Vagrant:

```bash
$ vagrant status client-node1
$ vagrant up client-node1
```

2. Log in to client-node1 and update the node:

```bash
$ vagrant ssh client-node1
$ sudo yum update -y

```
3. Check OS and kernel release (this is optional):

```bash
[vagrant@client-node1 ~]$ cat /etc/centos-release
CentOS Linux release 7.6.1810 (Core) 
[vagrant@client-node1 ~]$ uname -r
3.10.0-957.21.3.el7.x86_64

```
4.   Check for RBD support in the kernel:

```bash
[vagrant@client-node1 ~]$ sudo modprobe rbd
[vagrant@client-node1 ~]$ echo $?
0
[vagrant@client-node1 ~]$

```

5. Allow ceph-node1 monitor machine to access client-node1 over SSH. To do this, copy root SSH keys from ceph-node1 to client-node1 Vagrant user. Execute the following commands from ceph-node1 machine until otherwise specified:

```bash
## Log in to the ceph-node1 machine
$ vagrant ssh ceph-node1
$ sudo su -
# ssh-copy-id vagrant@client-node1

```
Provide a one-time Vagrant user password, that is, vagrant , for client-node1 . Once the SSH keys are copied from ceph-node1 to client-node1 , you should able to log in to client-node1 without a password.

6.  Using Ansible, we will create the ceph-client role which will copy the Ceph configuration file and administration keyring to the client node. On our Ansible administration node, ceph-node1 , add a new section [clients] to the /etc/ansible/hosts file:

```vi
[mons]
ceph-node1
ceph-node2
ceph-node3


[osds]
ceph-node1
ceph-node2
ceph-node3

[clients]
client-node1

```
7.  Go to the /etc/ansible/group_vars directory on ceph-node1 and create a copy of clients.yml from the clients.yml.sample :

  

```bash
[root@ceph-node1 ~]# cd  /etc/ansible/group_vars 
[root@ceph-node1 group_vars]# cp clients.yml.sample clients.yml

```

8. Run the Ansible playbook from ceph-node1 :

```bash

[root@ceph-node1 group_vars]#  cd /usr/share/ceph-ansible
[root@ceph-node1 ceph-ansible]# ansible-playbook site.yml
PLAY RECAP ********************************************************************************************************************************************************************************************************
ceph-node1                 : ok=199  changed=7    unreachable=0    failed=0    skipped=313  rescued=0    ignored=0   
ceph-node2                 : ok=188  changed=7    unreachable=0    failed=0    skipped=262  rescued=0    ignored=0   
ceph-node3                 : ok=189  changed=7    unreachable=0    failed=0    skipped=261  rescued=0    ignored=0   
client-node1               : ok=96   changed=13   unreachable=0    failed=0    skipped=202  rescued=0    ignored=0   


INSTALLER STATUS **************************************************************************************************************************************************************************************************
Install Ceph Monitor           : Complete (0:01:05)
Install Ceph OSD               : Complete (0:01:01)
Install Ceph Client            : Complete (0:02:48)

``` 
9.  On client-node1 check and validate that the keyring and ceph.conf file were populated into the /etc/ceph directory by Ansible:

```bash
[root@ceph-node1 ceph-ansible]# cd /etc/ceph/
[root@ceph-node1 ceph]# 
[root@ceph-node1 ceph]# ls
ceph.client.admin.keyring  ceph.conf  rbdmap
[root@ceph-node1 ceph]# 

```

10.  On client-node1 you can validate that the Ceph client packages were installed by Ansible:


```bash
[root@ceph-node1 ceph]# rpm  -qa |grep ceph
ceph-common-14.2.1-0.el7.x86_64
ceph-mon-14.2.1-0.el7.x86_64
ceph-mgr-14.2.1-0.el7.x86_64
libcephfs2-14.2.1-0.el7.x86_64
ceph-base-14.2.1-0.el7.x86_64
python-ceph-argparse-14.2.1-0.el7.x86_64
ceph-osd-14.2.1-0.el7.x86_64
ceph-mgr-dashboard-14.2.1-0.el7.noarch
python-cephfs-14.2.1-0.el7.x86_64
ceph-selinux-14.2.1-0.el7.x86_64
ceph-mgr-diskprediction-local-14.2.1-0.el7.noarch
[root@ceph-node1 ceph]# 


```

11. The client machine will require Ceph keys to access the Ceph cluster. Ceph creates a default user, client.admin , which has full access to the Ceph cluster and Ansible copies the client.admin key to client nodes. It's not recommended to share client.admin keys with client nodes. A better approach is to create a
new Ceph user with separate keys and allow access to specific Ceph pools. In our case, we will create a Ceph user, client.rbd , with access to the RBD pool . By default, Ceph Block Devices are created on the RBD pool :

```bash
[root@ceph-node1 ceph]# cd ~
[root@ceph-node1 ~]# ceph auth get-or-create client.rbd mon 'allow r' osd 'allow class-read object_prefix  rbd_children, allow rwx pool=rbd'
[client.rbd]
	key = AQB1AChdiDxnFRAA7T/evreQz3r75ZrfKSL0pQ==
[root@ceph-node1 ~]#
```

12.  Add the key to client-node1 machine for client.rbd user:

```bash
[root@ceph-node1 ~]# ceph  auth  get-or-create  client.rbd  |  ssh vagrant@client-node1 sudo tee /etc/ceph/ceph.client.rbd.keyring
vagrant@client-node1's password: 
[client.rbd]
	key = AQB1AChdiDxnFRAA7T/evreQz3r75ZrfKSL0pQ==
[root@ceph-node1 ~]# 

```

13. By this step, client-node1 should be ready to act as a Ceph client. Check the cluster status from the client-node1 machine by providing the username and secret key:

```bash
[root@ceph-node1 ~]# ssh client-node1
[root@client-node1 ~]# cat /etc/ceph/ceph.client.rbd.keyring >> /etc/ceph/keyring
[root@client-node1 ~]# 
```

Since we are not using the default user client.admin we need to supply username that will connect to the Ceph cluster

```bash
[root@client-node1 ~]# cat /etc/ceph/ceph.client.rbd.keyring
[client.rbd]
	key = AQB1AChdiDxnFRAA7T/evreQz3r75ZrfKSL0pQ==
[root@client-node1 ~]# cat /etc/ceph/ceph.client.rbd.keyring >> /etc/ceph/keyring
[root@client-node1 ~]# ceph -s --name client.rbd
  cluster:
    id:     73ce9919-a43f-4881-b5fe-920973b31334
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-node1,ceph-node3,ceph-node2 (age 70m)
    mgr: ceph-node1(active, since 92m), standbys: ceph-node3, ceph-node2
    osd: 9 osds: 9 up (since 67m), 9 in (since 67m)
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   9.0 GiB used, 162 GiB / 171 GiB avail
    pgs:     
 
[root@client-node1 ~]# 
```
## Creating Ceph Block Device 

Up to now, we have configured Ceph client, and now we will demonstrate creating a Ceph Block Device from the client-node1 machine.

### How to do it...

1. Create a RADOS Block Device named rbd1 of size 10240 MB:
```bash
[root@client-node1 ~]# rbd create rbd1 --size 10240 --name client.rbd
rbd: error opening default pool 'rbd'
Ensure that the default pool has been created or specify an alternate pool name.

```
2. There are multiple options that you can use to list RBD images:
```
## The default pool to store block device images is "rbd",

you can also specify the pool name with the rbd
command using -p option:
# rbd ls --name client.rbd
# rbd ls -p rbd --name client.rbd
# rbd list --name client.rbd
```

3. Check the details of the RBD image:
```
# rbd --image rbd1 info --name client.rbd
```

## Mapping Ceph Block Device

Now that we have created a block device on a Ceph cluster, in order to use this block device, we need to map it to the client machine. To do this, execute the following commands from the client-node1 machine.

### How to do it...

1.  Map the block device to the client-node1 :

```
# rbd map --image rbd1 --name client.rbd

```
2. With Ceph Jewel the new default format for RBD images is 2 and Ceph Jewel default configuration includes the following default Ceph Block Device features:

* layering : layering support
* exclusive-lock : exclusive locking support
* object-map : object map support (requires exclusive-lock )
* deep-flatten : snapshot flatten support
* fast-diff : fast diff calculations (requires object-map ) Using the krbd (kernel rbd) client on client-node1 we will be unable to map the block device image on CentOS kernel 3.10 as this
kernel does not support object-map , deep-flatten and fast-diff (support was introduced in kernel 4.9). In order to work around this we will disable the unsupported features, there are several options to do this:

    * Disable the unsupported features dynamically (this is
the option we will be using):

```bash
rbd feature disable rbd1
exclusive-lock object-map
deep-flatten fast-diff
```
    * When creating the RBD image initially utilize the --image-feature layering option with the rbd create command which will only enable the layering feature:

```bash
  rbd create rbd1 --size 10240
--image-feature layering
--name client.rbd
```

    * Disable the feature in the Ceph configuration file:
    ```bash
    rbd_default_features = 1
    ```

3. Retry mapping the block device with the unsupported features now disabled:

```
# rbd map --image rbd1 --name client.rbd
```
4. Check the mapped block device:

```
rbd showmapped --name client.rbd

```
5. To make use of this block device, we should create a filesystem on this and mount it:

```
#
#
#
#
#
fdisk -l /dev/rbd0
mkfs.xfs /dev/rbd0
mkdir /mnt/ceph-disk1
mount /dev/rbd0 /mnt/ceph-disk1
df -h /mnt/ceph-disk1
```

6. Test the block device by writing data to it:

```
dd if=/dev/zero of=/mnt/ceph-disk1/file1 count=100 bs=1M
```

7. To map the block device across reboots, we will need to create and configure a
services file:
    1.  Create a new file in the /usr/local/bin directory for mounting and unmounting and include the  following:

    ```bash
# cd /usr/local/bin
# vim rbd-mount

    ```

    2.  Save the file and make it executable:
    ```
      # sudo chmod +x rbd-mount
    ```
    This can be done automatically by grabbing the rbd-mount script from the Ceph-Designing-and-Implementing-Scalable-Storage-Systems repository and making it executable:
    ```
    # wget https://raw.githubusercontent.com/PacktPublishing/Ceph-Designing-and-Implementing-Scalable-Storage-Systems/Module_1/master/
    rbdmap -O /usr/local/bin/rbd-mount
    # chmod +x /usr/local/bin/rbd-mount

    ```
    3. Go to the systemd directory and create the service file, include the following in the file rbd-mount.service :

    ```bash
    # cd /etc/systemd/system/
    # vim rbd-mount.service
    ```
    This can be done automatically by grabbing the service file from the Ceph-Designing-and-Implementing-Scalable-Storage-Systems/Chapter02 repository:

    ```bash
    # wget https://raw.githubusercontent.com/PacktPublishing/Ceph-Designing-and-Implementing-Scalable-Storage-Systems/Chapter02/rbd-mount.service

    ```
    4. After saving the file and exiting Vim, reload the systemd files and enable the rbd-mount.service to start at boot time:

    ```bash
    # systemctl daemon-reload
    # systemctl enable rbd-mount.service
    ```

8. Reboot client-node1 and verify that block device rbd0 is mounted to /mnt/ceph-disk1 after the reboot:

```bash
root@client-node1 # reboot -f
# df -h

```