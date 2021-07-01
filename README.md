# okd-3.11-Installation
## Install OpenShift Origin OKD 3.11 on Oracle Linux 7
OKD - the Openshift K8s Distribution

OKD is a distribution of K8s optimized for continuous application development and multi-tenant deployment.
> OKD adds developer and Operations-centric tools on top of K8s to enable rapid application development, 
> easy deployment, scaling and long-term life cycle maintenance for small and large teams ~ okd.io

### Setup
We will Install OKD 3.11 on Oracle Linux 7 on a 4 node cluster.
Below are the specs and settings for the nodes.
| Hostname      | IP            | Role           | FQDN                     | CPU cores     | RAM           | Storage       | SSH User      | OS            | 
| ------------- | ------------- | -------------- |------------------------- | ------------- | ------------- | ------------- | ------------- | ------------- | 
| okdtestvm1    | 10.150.150.10 | Master         | vm1.10.150.150.10.nip.io | 8             | 16GB          | 100GB + 120GB | sekayasin     | OL7           |
| okdtestvm2    | 10.150.150.11 | Master         | vm2.10.150.150.11.nip.io | 8             | 16GB          | 100GB + 120GB | sekayasin     | OL7           |
| okdtestvm3    | 10.150.150.12 | Master         | vm3.10.150.150.12.nip.io | 8             | 16GB          | 100GB + 120GB | sekayasin     | OL7           |
| okdtestvm4    | 10.150.150.13 | Master         | vm4.10.150.150.13.nip.io | 8             | 16GB          | 100GB + 120GB | sekayasin     | OL7           |

### Installation
I will assume we have done a fresh OS installation on our nodes, networked and have direct connectivity to the internet. 
- [x] Configure your nodes with the latest oracale repo for developers to download development packages.You can add this yum repo
```
$ sudo cd /etc/yum.repos.d/
$ sudo vi oraclelinux-developer-ol7.repo
```
Add this content
```
[ol7_developer]
name=Oracle Linux $releasever Development Packages ($basearch)
baseurl=https://yum$ociregion.oracle.com/repo/OracleLinux/OL7/developer/$basearch/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-oracle
gpgcheck=1
enabled=1

[ol7_developer_kvm_utils]
name=Oracle Linux $releasever KVM Utilities for Development and test ($basearch)
baseurl=https://yum$ociregion.oracle.com/repo/OracleLinux/OL7/developer/kvm/utils/$basearch/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-oracle
gpgcheck=1
enabled=0

[ol7_developer_UEKR5]
name=Oracle Linux $releasever Unbreakable Enterprise Kernel Release 5  Packages for Development and test ($basearch)
baseurl=http://yum$ociregion.oracle.com/repo/OracleLinux/OL7/developer_UEKR5/$basearch/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-oracle
gpgcheck=1
enabled=0
```
> save & quit -- Hope You know how to Quit Vi Editor :joy:
- [x] Now Update All Your Nodes
> $ sudo yum update -y
- [x] Add your Nodes entries in each Node hosts file - /etc/hosts
```
10.150.150.10 okdtestvm1 vm1.10.150.150.10.nip.io api.staging.nip.io apps.staging.nip.io api-int.staging.nip.io
10.150.150.11 okdtestvm2 vm2.10.150.150.11.nip.io
10.150.150.12 okdtestvm3 vm3.10.150.150.12.nip.io
10.150.150.13 okdtestvm4 vm4.10.150.150.13.nip.io
```
- [x] Choose the Node (TheChosenOne) that you will use to execute Ansible playbooks from. And Install Ansible on it.<br>
Since python2 is bundled by default on the OS, No need to install it. Ansible will use the default version of python
```
$ sudo yum install ansible -y
```
Verify Ansible is installed
```
$ ansible --version
```
From TheChosenOne - Verify if you can `ssh` to all other Nodes.<br>
You setup `ssh passwordless login` to `ssh` automatically to all other Nodes -- optional.

#### OKD Installation Preps
```
$ sudo yum install git -y 

$ git clone https://github.com/sekayasin/okd-3.11-Installation.git

$ cd okd-3.11-Installation/my-prep-files 

NOTE: Edit your inventory file named hosts, add your ansible_ssh_user, your nodes hostnames to master, etcd, & node groups. 
Edit the custom okd-prep playbook to add the remote_user, Edit the okd-users playbook, we shall use it to add a user to htpasswd.

Test using ansible ping module if can reach all your nodes specified the hosts inventory file
Dont use the -k -K flags if u enabled ssh passwordless login

$ ansible all -m ping -i hosts -k -K

Run the okd-prep.yml playbook

$ ansible-playbook okd-prep.yaml -i hosts -k -K

$ cd ~

if this is success -- clone Openshift-Ansible Repo
```

#### Clone Openshift-Ansible Repo and Run the cluster deploy playbook
```
$ git clone https://github.com/openshift/openshift-ansible.git

$ cd openshift-ansible && git fetch && git checkout release-3.11 & cd ..

NOTE: You can run the prerequisites.yml playbook if u want, our custom okd-prep.yml playbook gave us a go ahead.
OPTIONAL RUN:

$ ansible-playbook -i okd-3.11-Installation/my-prep-files/hosts  openshift-ansible/playbooks/prerequisites.yml

Finally Run the cluster deploy playbook

$ ansible-playbook -i okd-3.11-Installation/my-prep-files/hosts  openshift-ansible/playbooks/deploy_cluster.yml
```

### Troubleshooting Issues
#### subscription-manager Issue on Oracle Linux 7







