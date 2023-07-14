# Kubespray
Kubespray is a composition of Ansible playbooks, inventory, provisioning tools, and domain knowledge for generic OS/Kubernetes clusters configuration management tasks
## Cluster model
- Ansible Node (Kubespray Node): CentOS Stream 8 (172.16.10.15)
- 1 Master Node: CentOS 7 (172.16.10.12)
- 1 Worker Node: CentOS 7 (172.16.10.13)
## Configurating OS parameters for nodes
- Turn off swap
  
  ```
  $ sed -i '/swap/d' /etc/fstab
  $ swapoff -a
  ```
- Disable SELinux
  ```
  $ setenforce 0
  $ sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
  ```
- Turn off firewalld service
  ```
  $ systemctl stop firewalld
  $ systemctl disable firewalld
  ```
- Configurating for ip_forward
  ```
  $ sysctl -w net.ipv4.ip_forward=1
  ```
- Generate SSH key and configure SSH connectionư
  - Generating key (using on Ansible Node)

    ```
    $ ssh-keygen
    ```
  - Copy key to connection with nodes (using its hostname)

    ```
    $ ssh-copy-id master.gp
    $ ssh-copy-id worker.gp
    ```
## Setup Kubesray on Ansible node
### Install Python
```
$ sudo yum update
$ sudo dnf groupinstall 'development tools'
$ sudo dnf install wget openssl-devel bzip2-devel libffi-devel

$ sudo curl https://www.python.org/ftp/python/3.9.1/Python-3.9.1.tgz -O
$ tar -xvf Python-3.9.1.tgz

$ cd Python-3.9.1

$ sudo ./configure --enable-optimizations
$ sudo make install

```

* Note
If you have an already existing Python binary installed at /usr/bin/python or /usr/bin/python3, you should run `` sudo make altinstall `` instead

### Install relate packages

```
$ cd kubespray

$ pip3 install -r requirements.txt
```
### Setup Node through CLI

```
$ cp -rfp inventory/sample inventory/mycluster
$ declare -a IPS=(192.168.50.10 192.168.50.20 192.168.50.30)
$ CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```
* Note: You should review the following files and edit it if needed
  
  - inventory/mycluster/hosts.yaml
  - inventory/mycluster/group_vars/all/all.yml
  - inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml
    
### Install K8s Cluster Using Ansible Playbook:
```
$ ansible-playbook -i inventory/mycluster/hosts.yaml --become --become-user=root cluster.yml
```
## The operations of adding, deleting, and resetting nodes
- Add Node \
  Edit hosts.yaml then

  ```
  $ ansible-playbook -i inventory/mycluster/hosts.yaml --user root scale.yaml
  ```
- Delete Node 
  ```
  $ ansible-playbook -i inventory/mycluster/hosts.yaml --user root remove-node.yaml --extra-vars "node=node-name"
  ```
- Reset Node \
  Edit hosts.yaml file to remove nodes which you want
  ```
  $ ansible-playbook -i inventory/mycluster/hosts.yaml --user root reset.yaml
  ```
## Upgrade version for nodes 
  - [Links](https://www.youtube.com/watch?v=M499ckeGZL8&list=PL34sAs7_26wNBRWM6BDhnonoA5FMERax0&index=106)

  
