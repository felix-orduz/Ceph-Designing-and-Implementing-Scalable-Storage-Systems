# Setting up a virtual infrastructure

1. we will launch three VMs using Vagrant; they are required throughout this chapter:

```bash

$ vagrant up ceph-node1 ceph-node2 ceph-node3

```
2. Run vagrant up ceph-node1 ceph-node2 ceph-node3 .
3. Check the status of your virtual machines:

```bash
$ vagrant status ceph-node1 ceph-node2 ceph-node3
```

4. Vagrant will, by default, set up hostnames as ceph-node<node_number> and IP address subnet as 192.16.1.X and will create three additional disks that will be used as OSDs by the Ceph cluster. Log in to each of these machines one by one and check whether the hostname, networking, and additional disks have been set up correctly by Vagrant:

```bash 

$ vagrant ssh ceph-node1
$ ip addr show
$ sudo fdisk -l
$ exit

``` 

5.  Vagrant is configured to update hosts file on the VMs. For convenience, update the /etc/hosts file on your host machine with the following content:

```

192.16.1.101 ceph-node1
192.16.1.102 ceph-node2
192.16.1.103 ceph-node3

```

6.  Generate root SSH keys for ceph-node1 and copy the keys to ceph-node2 and ceph-node3 . The password for the root user on these VMs is vagrant . Enter the root user password when asked by the ssh-copy-id command and proceed with the default settings:

```bash 

[vagrant@ceph-node1 ~]$ sudo su -
[root@ceph-node1 ~]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:0bL+Jxw3CB5u17pA4Nj9BAu51tt4gjwPOZ3Tw+029m0 root@ceph-node1
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|       . .       |
|      + + .      |
|     + *o*       |
|    . =oSo.o     |
|     o *+X+.+    |
|      B.XoB+..   |
|       = *+o=  .E|
|        . o*.o...|
+----[SHA256]-----+
[root@ceph-node1 ~]# 
[root@ceph-node1 ~]# ssh-copy-id  ceph-node1
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'ceph-node1 (127.0.0.1)' can't be established.
ECDSA key fingerprint is SHA256:uVe9KVMbBydyfuFEJyk/ldFIErhtXBy5dR38/EccAgA.
ECDSA key fingerprint is MD5:3a:d9:01:a4:de:e3:bd:5c:24:30:5c:b3:0f:12:37:c0.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@ceph-node1's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'ceph-node1'"
and check to make sure that only the key(s) you wanted were added.

[root@ceph-node1 ~]# ssh-copy-id  ceph-node2
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'ceph-node2 (192.16.1.102)' can't be established.
ECDSA key fingerprint is SHA256:uVe9KVMbBydyfuFEJyk/ldFIErhtXBy5dR38/EccAgA.
ECDSA key fingerprint is MD5:3a:d9:01:a4:de:e3:bd:5c:24:30:5c:b3:0f:12:37:c0.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@ceph-node2's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'ceph-node2'"
and check to make sure that only the key(s) you wanted were added.

[root@ceph-node1 ~]# ssh-copy-id  ceph-node3
/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'ceph-node3 (192.16.1.103)' can't be established.
ECDSA key fingerprint is SHA256:uVe9KVMbBydyfuFEJyk/ldFIErhtXBy5dR38/EccAgA.
ECDSA key fingerprint is MD5:3a:d9:01:a4:de:e3:bd:5c:24:30:5c:b3:0f:12:37:c0.
Are you sure you want to continue connecting (yes/no)? yes
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@ceph-node3's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'ceph-node3'"
and check to make sure that only the key(s) you wanted were added.

[root@ceph-node1 ~]#


```

7.  Once the SSH keys are copied to ceph-node2 and ceph-node3 , the root user from ceph-node1 can do an ssh login to VMs without entering the password:

```bash

[root@ceph-node1 ~]# ssh ceph-node3  hostname
ceph-node3
[root@ceph-node1 ~]# ssh ceph-node2  hostname
ceph-node2

```

8.  Enable ports that are required by the Ceph MON, OSD, and MDS on the operating system's firewall. Execute the following commands on all VMs:

```bash

$ firewall-cmd --zone=public --add-port=6789/tcp --permanent
$ firewall-cmd --zone=public --add-port=6800-7100/tcp --permanent
$ firewall-cmd --reload
$ firewall-cmd --zone=public --list-all


```
```bash

[root@ceph-node1 ~]# firewall-cmd --zone=public --add-port=6789/tcp --permanent
success
[root@ceph-node1 ~]# firewall-cmd --zone=public --add-port=6800-7100/tcp --permanent
success
[root@ceph-node1 ~]# firewall-cmd --reload
success
[root@ceph-node1 ~]# firewall-cmd --zone=public --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0 eth1
  sources: 
  services: ssh dhcpv6-client
  ports: 6789/tcp 6800-7100/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	
[root@ceph-node1 ~]#

```
9.  Install and configure NTP on all VMs:

```bash

yum install ntp ntpdate -y
ntpdate pool.ntp.org
systemctl restart ntpdate.service
systemctl restart ntpd.service
systemctl enable ntpd.service
systemctl enable ntpdate.service

```

#  Installing and configuring Ceph

## Creating the Ceph cluster on ceph-node1
We will first install Ceph and configure ceph-node1 as the Ceph monitor and the Ceph OSD node. Later recipes in this chapter will introduce ceph-node2 and ceph-node3 .

### How to do it...

Copy ceph-ansible package on ceph-node1 from the Ceph-Cookbook-Second-Edition directory.

1. Log in to ceph-node1 and install ceph-ansible on ceph-node1 :

```bash

$ vagrant ssh ceph-node1
[vagrant@ceph-node1 ~]# cd /vagrant
[vagrant@ceph-node1 /vagrant]# sudo -s
[root@ceph-node1 vagrant]# yum install -y  ceph-ansible-2.2.10-38.g7ef908a.el7.noarch.rpm 

```
Or you can

```bash
$ vagrant ssh ceph-node1
[vagrant@ceph-node1 ~]# cd /vagrant
[vagrant@ceph-node1 /vagrant]# sudo -s
[root@ceph-node1 vagrant]# wget https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm
[root@ceph-node1 vagrant]# rpm -ivh  epel-release-7-11.noarch.rpm
[root@ceph-node1 vagrant]# rm -rf epel-release-7-11.noarch.rpm
[root@ceph-node1 vagrant]# yum install -y  ansible
[root@ceph-node1 vagrant]# git clone https://github.com/ceph/ceph-ansible.git
[root@ceph-node1 vagrant]# cd ceph-ansible
[root@ceph-node1 vagrant]# git checkout stable-4.0
```

2.  Update the Ceph hosts to /etc/ansible/hosts :

```
[mons]
ceph-node1

[osds]
ceph-node1
```

3.  Verify that Ansible can reach the Ceph hosts mentioned in /etc/ansible/hosts :

```bash
[root@ceph-node1 vagrant]# ansible all -m ping
ceph-node1 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
[root@ceph-node1 vagrant]#
```

4.  Create a directory under the root home directory so Ceph Ansible can use it for storing the keys:

```bash
[root@ceph-node1 vagrant]# cd ~
[root@ceph-node1 ~]# mkdir  ceph-ansible-keys
[root@ceph-node1 ~]# 
```

5.  Create a symbolic link to the Ansible group_vars directory in the /etc/ansible/ directory:

```bash
[root@ceph-node1 ~]# 
[root@ceph-node1 ~]# ln -s /usr/share/ceph-ansible//group_vars/  /etc/ansible/group_vars
[root@ceph-node1 ~]# 
```

6.  Go to /etc/ansible/group_vars and copy an all.yml file from the all.yml.sample file and open it to define configuration options' values:

```bash
[root@ceph-node1 ~]# 
[root@ceph-node1 ~]# cd /etc/ansible/group_vars/
[root@ceph-node1 group_vars]# cp all.yml.sample   all.yml
[root@ceph-node1 group_vars]# vim all.yml
```


7.  Define the following configuration options in all.yml for the latest jewel  version on CentOS 7:

```bash
fetch_directory: ~/ceph-ansible-keys
centos_package_dependencies:
  - python-pycurl
  - hdparm
  - epel-release
  - python-setuptools
  - libselinux-python
#ceph_origin: 'upstream' # or 'distro' or 'local'
ceph_origin: repository
ceph_repository: community
ceph_stable: true # use ceph stable branch
ceph_stable_release: nautilus  # ceph stable release
ceph_mirror: http://mirrors.aliyun.com/ceph
ceph_stable_key: http://mirrors.aliyun.com/ceph/keys/release.asc
ceph_stable_redhat_distro: el7
cpu_arch: x86_64
cephx: true
monitor_interface: eth1
journal_size: 2048 # OSD journal size in MB
public_network: 192.16.1.0/24
cluster_network: "{{ public_network }}"
osd_mkfs_type: xfs
osd_mkfs_options_xfs: -f -i size=2048
osd_mount_options_xfs: noatime,largeio,inode64,swalloc

```

8.   Go to /etc/ansible/group_vars and copy an osds.yml file from the osds.yml.sample file and open it to define configuration options' values:

```bash

[root@ceph-node1 group_vars]# 
[root@ceph-node1 group_vars]# cp osds.yml.sample  osds.yml
[root@ceph-node1 group_vars]# vim osds.yml
[root@ceph-node1 group_vars]# 

```

9.   Define the following configuration options in osds.yml for OSD disks; we are co-locating an OSD journal in the OSD data disk:

```bash

osd_scenario: collocated # 沒有它無法執行
devices:
  - /dev/sdb
  - /dev/sdc
  - /dev/sdd
journal_collection: true

```

10.  Go to /usr/share/ceph-ansible and add retry_files_save_path option in ansible.cfg in the [defaults] tag:

```bash

[root@ceph-node1 group_vars]# cd ~
[root@ceph-node1 ~]# cd /usr/share/ceph-ansible
[root@ceph-node1 ceph-ansible]# vim ansible.cfg 
[root@ceph-node1 ceph-ansible]# cat ansible.cfg |grep retry_files_save
retry_files_save_path = ~/
[root@ceph-node1 ceph-ansible]# 


```

11.  Run Ansible playbook in order to deploy the Ceph cluster on ceph-node1 : 

To run the playbook, you need site.yml , which is present in the same path: /usr/share/ceph-ansible/ . You should be in the /usr/share/ceph-ansible/ path and should run following commands:

```
# cp site.yml.sample site.yml
# ansible-playbook site.yml
```

```bash

[root@ceph-node1 group_vars]# cd /usr/share/ceph-ansible
[root@ceph-node1 ceph-ansible]# 
[root@ceph-node1 ceph-ansible]# cp site.yml.sample site.yml
[root@ceph-node1 ceph-ansible]#  ansible-playbook site.yml

PLAY [mons,agents,osds,mdss,rgws,nfss,restapis,rbdmirrors,clients,mgrs] ************************************************************************************************************

TASK [check for python2] ************************************************************************************************************
ok: [ceph-node1]

TASK [install python2 for Debian based systems] ************************************************************************************************************
skipping: [ceph-node1]

```


Once playbook completes the Ceph cluster installation job and plays the recap with failed=0 , it means ceph-ansible has deployed the Ceph cluster, as shown in the following screenshot:

```bash
PLAY RECAP ************************************************************************************************************
ceph-node1                 : ok=126  changed=24    unreachable=0    failed=0   
[root@ceph-node1 ceph-ansible]# ceph -s
  cluster:
    id:     73ce9919-a43f-4881-b5fe-920973b31334
    health: HEALTH_OK
 
  services:
    mon: 1 daemons, quorum ceph-node1 (age 8m)
    mgr: ceph-node1(active, since 6m)
    osd: 3 osds: 3 up (since 5m), 3 in (since 5m)
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   3.0 GiB used, 54 GiB / 57 GiB avail
    pgs:     
 
[root@ceph-node1 ceph-ansible]# ceph osd tree
ID CLASS WEIGHT  TYPE NAME           STATUS REWEIGHT PRI-AFF 
-1       0.05576 root default                                
-3       0.05576     host ceph-node1                         
 0   hdd 0.01859         osd.0           up  1.00000 1.00000 
 1   hdd 0.01859         osd.1           up  1.00000 1.00000 
 2   hdd 0.01859         osd.2           up  1.00000 1.00000 


```

You have all three OSD daemons and one monitor daemon up and running in ceph-
node1 .
Here's how you can check the Ceph jewel release installed version. You can run the ceph -v command to check the installed ceph version:

```bash
[root@ceph-node1 ceph-ansible]# ceph -v
ceph version 14.2.1 (d555a9489eb35f84f2e1ef49b77e19da9d113972) nautilus (stable)

```

## Scaling up your Ceph cluster
At this point, we have a running Ceph cluster with one MON and three OSDs configured on ceph-node1 . Now we will scale up the cluster by adding ceph-node2 and ceph-node3 as MON and OSD nodes.

### How to do it...
A Ceph storage cluster requires at least one monitor to run. For high availability, a Ceph storage cluster relies on an odd number of monitors and more than one, for example, 3 or 5,to form a quorum. It uses the Paxos algorithm to maintain quorum majority. You will notice that your Ceph cluster is currently showing HEALTH_WARN ; this is because we have not configured any OSDs other than ceph-node1 . By default, the data in a Ceph cluster is replicated three times, that too on three different OSDs hosted on three different nodes. 

Since we already have one monitor running on ceph-node1 , let's create two more monitors for our Ceph cluster and configure OSDs on ceph-node2 and ceph-node3 :

1.  Update the Ceph hosts ceph-node2 and ceph-node3 to /etc/ansible/hosts :

```bash
[mons]
ceph-node1
ceph-node2
ceph-node3

[osds]
ceph-node1
ceph-node2
ceph-node3

```

2.  Verify that Ansible can reach the Ceph hosts mentioned in /etc/ansible/hosts :

```bash
[root@ceph-node1 ceph-ansible]# ansible all -m ping
ceph-node1 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
ceph-node3 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
ceph-node2 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}

```

3.  Run Ansible playbook in order to scale up the Ceph cluster on ceph-node2 and ceph-node3 :

```bash

[root@ceph-node1 ceph-ansible]# ansible-playbook  site.yml

PLAY [mons,agents,osds,mdss,rgws,nfss,restapis,rbdmirrors,clients,mgrs] ************************************************************************************************************

TASK [check for python2] ************************************************************************************************************
ok: [ceph-node1]

TASK [install python2 for Debian based systems] ************************************************************************************************************
skipping: [ceph-node1]



```

Once playbook completes the ceph cluster scaleout job and plays the recap with failed=0 , it means that the Ceph ansible has deployed more Ceph daemons in the cluster, as shown in the following screenshot.

You have three more OSD daemons and one more monitor daemon running in ceph-node2 and three more OSD daemons and one more monitor daemon running in ceph-node3 . Now you have total nine OSD daemons and three monitor daemons running on three nodes:

```bash
PLAY RECAP ********************************************************************************************************************************************************************************************************
ceph-node1                 : ok=213  changed=8    unreachable=0    failed=0    skipped=319  rescued=0    ignored=0   
ceph-node2                 : ok=200  changed=33   unreachable=0    failed=0    skipped=281  rescued=0    ignored=0   
ceph-node3                 : ok=200  changed=32   unreachable=0    failed=0    skipped=291  rescued=0    ignored=0   


INSTALLER STATUS **************************************************************************************************************************************************************************************************
Install Ceph Monitor           : Complete (0:04:24)
Install Ceph OSD               : Complete (0:02:08)

Friday 12 July 2019  02:36:53 +0000 (0:00:00.033)       0:07:51.407 *********** 
=============================================================================== 
ceph-common : install redhat ceph packages ---------------------------------------------------------------------------------------------------------------------------------------------------------------- 68.63s
ceph-mgr : install ceph-mgr packages on RedHat or SUSE ---------------------------------------------------------------------------------------------------------------------------------------------------- 64.18s
ceph-osd : use ceph-volume lvm batch to create bluestore osds --------------------------------------------------------------------------------------------------------------------------------------------- 62.38s
ceph-common : install centos dependencies ----------------------------------------------------------------------------------------------------------------------------------------------------------------- 20.58s
ceph-common : install yum plugin priorities --------------------------------------------------------------------------------------------------------------------------------------------------------------- 15.38s
ceph-mon : waiting for the monitor(s) to form the quorum... ----------------------------------------------------------------------------------------------------------------------------------------------- 12.10s
ceph-osd : systemd start osd ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 9.14s
ceph-validate : validate provided configuration ------------------------------------------------------------------------------------------------------------------------------------------------------------ 8.61s
ceph-mgr : fetch ceph mgr keyring -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 6.64s
ceph-mon : create monitor initial keyring ------------------------------------------------------------------------------------------------------------------------------------------------------------------ 5.63s
ceph-mon : fetch ceph initial keys ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 5.11s
ceph-infra : install chrony -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 3.89s
ceph-facts : set_fact ceph_current_status (convert to json) ------------------------------------------------------------------------------------------------------------------------------------------------ 3.88s
gather and delegate facts ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 3.86s
ceph-config : run 'ceph-volume lvm batch --report' to see how many osds are to be created ------------------------------------------------------------------------------------------------------------------ 3.34s
ceph-infra : open monitor and manager ports ---------------------------------------------------------------------------------------------------------------------------------------------------------------- 3.22s
ceph-mgr : systemd start mgr ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 2.85s
ceph-common : get ceph version ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 2.76s
ceph-config : generate ceph configuration file: ceph.conf -------------------------------------------------------------------------------------------------------------------------------------------------- 2.45s
ceph-infra : start firewalld ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 2.42s
[root@ceph-node1 ceph-ansible]# ceph -s
  cluster:
    id:     73ce9919-a43f-4881-b5fe-920973b31334
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-node1,ceph-node3,ceph-node2 (age 4m)
    mgr: ceph-node1(active, since 26m), standbys: ceph-node3, ceph-node2
    osd: 9 osds: 9 up (since 66s), 9 in (since 66s)
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   9.0 GiB used, 162 GiB / 171 GiB avail
    pgs:     
 
[root@ceph-node1 ceph-ansible]# ceph osd tree
ID CLASS WEIGHT  TYPE NAME           STATUS REWEIGHT PRI-AFF 
-1       0.16727 root default                                
-3       0.05576     host ceph-node1                         
 0   hdd 0.01859         osd.0           up  1.00000 1.00000 
 1   hdd 0.01859         osd.1           up  1.00000 1.00000 
 2   hdd 0.01859         osd.2           up  1.00000 1.00000 
-7       0.05576     host ceph-node2                         
 4   hdd 0.01859         osd.4           up  1.00000 1.00000 
 5   hdd 0.01859         osd.5           up  1.00000 1.00000 
 7   hdd 0.01859         osd.7           up  1.00000 1.00000 
-5       0.05576     host ceph-node3                         
 3   hdd 0.01859         osd.3           up  1.00000 1.00000 
 6   hdd 0.01859         osd.6           up  1.00000 1.00000 
 8   hdd 0.01859         osd.8           up  1.00000 1.00000 
[root@ceph-node1 ceph-ansible]# ceph osd pool set   rbd  pg_num 128
Error ENOENT: unrecognized pool 'rbd'
```

So We rebuild osd pool

```bash
[root@ceph-node1 ~]# ceph osd pool create rbd 8
pool 'rbd' created
[root@ceph-node1 ~]# rbd pool init rbd
[root@ceph-node1 ~]# ceph osd pool set rbd pg_num 128
set pool 1 pg_num to 128
[root@ceph-node1 ~]# ceph osd pool set rbd pgp_num 128
set pool 1 pgp_num to 128
[root@ceph-node1 ~]# ceph -s
  cluster:
    id:     73ce9919-a43f-4881-b5fe-920973b31334
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-node1,ceph-node3,ceph-node2 (age 81m)
    mgr: ceph-node1(active, since 103m), standbys: ceph-node3, ceph-node2
    osd: 9 osds: 9 up (since 78m), 9 in (since 78m)
 
  data:
    pools:   1 pools, 128 pgs
    objects: 1 objects, 16 B
    usage:   9.0 GiB used, 162 GiB / 171 GiB avail
    pgs:     128 active+clean
 
  io:
    recovery: 2 B/s, 0 objects/s
 
[root@ceph-node1 ~]#
```

4.   We were getting a too few PGs per OSD warning and because of that, we increased the default RBD pool PGs from 64 to 128. Check the status of your Ceph cluster; at this stage, your cluster is healthy. PGs - placement groups are covered in detail in Chapter 8 , Ceph Under the Hood.

## Using the Ceph cluster with a hands-on approach
Now that we have a running Ceph cluster, we will perform some hands-on practice to gain experience with Ceph using some basic commands.

### How to do it...
Below are some of the common commands used by Ceph admins:

1. Check the status of your Ceph installation:
```
# ceph -s or # ceph status
```
2. Check Ceph's health detail:

```
# ceph health detail
```

3. Watch the cluster health:

```
# ceph -w
```

4. Check Ceph's monitor quorum status:
```
# ceph quorum_status --format json-pretty
```

5. Dump Ceph's monitor information:
```
# ceph mon dump
```
6. Check the cluster usage status:

```
# ceph df
```
7. Check the Ceph monitor, OSD, pool, and placement group stats:

```
# ceph mon stat
# ceph osd stat
# ceph osd pool stats
# ceph pg stat
 
```

8. List the placement group:

```
# ceph pg dump
```
9. List the Ceph pools in detail:

```
# ceph osd pool ls detail
```

10. Check the CRUSH map view of OSDs:

```
# ceph osd tree

```
11. Check Ceph's OSD usage:

```
# ceph osd df
```
12. List the cluster authentication keys:
```
# ceph auth list
```

These were some basic commands that you learned in this section. In the upcoming chapters, you will learn advanced commands for Ceph cluster management.