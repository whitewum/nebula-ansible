# nebula-ansible

## Introduction

nebula-ansible is a nebula cluster deployment tool based on ansible playbook, which helps you deploy nebula services on cluster quickly.

## Usage

### Install ansible in the central control machine.
```shell
$ pip install --user ansible
```

Other installation methods can be seen [here](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

### Create nebula user account on central control machine and generate ssh key
Login as root user on central control machine， and create nebula user account

```shell
$ useradd -m -d /home/nebula nebula
```

Set password of nebula

```shell
$ passwd nebula
```

### Make nebula user free of sudo password

Execute visudo and append `nebula ALL=(ALL) NOPASSWD: ALL` to the end

```shell
$ visudo
```

```text
nebula ALL=(ALL) NOPASSWD: ALL
```

Switch from root user to nebula user

```shell
$ su - nebula
```

Create ssh key of nebula user

```shell
$ ssh-keygen -t rsa
```

Download nebula-ansible to central control machine

```shell
$ git clone https://github.com/jievince/nebula-ansible.git
```

### Configure the deployment machine on the central control machine ssh mutual trust and sudo rules
Login as nebula user on central control machine, add the ip of the machines to be deployed to the [servers] group of hosts file.

```shell
$ cd /home/nebula/nebula-ansible
$ vi hosts
```
```shell
[servers]
192.168.8.16
192.168.8.18
192.168.8.122
192.168.8.188

[all:vars]
username=nebula
```

Execute the following command and enter the root password of the deployment target machine as prompted. 

This step will create a nebula user on the deployment target machine and configure the sudo rule to configure ssh mutual trust between the central control machine and the target machine.

```shell
$ ansible-playbook -i hosts create_users.yml -u root -k
```

Execute the following command and if all servers return nebula, the ssh mutual trust configuration is successful

```shell
$ ansible -i inventory.ini all -m shell -a 'whoami'
```

Execute the following command and if all servers return root it indicates nebula user sudo password-free configuration is successful

```shell
$ ansible -i inventory.ini all -m shell -a 'whoami' -b
```

### Allocate machine resources

Edit the inventory.ini file

```shell
$ vi inventory.ini
```

```shell
[metad_servers]
192.168.8.122
192.168.8.188

[graphd_servers]
192.168.8.16
192.168.8.18
192.168.8.122

[storaged_servers]
192.168.8.16
192.168.8.18
192.168.8.122
192.168.8.188
```

### Deploy nebula to machines of cluster

Execute local_prepare.yml playbook, it will download nebula package to the central control machine

```shell
$ ansible-playbook local_prepare.yml
```

Deploy nebula services on machines of cluster

```shell
$ ansible-playbook deploy.yml
```

### Start nebula services

```shell
$ ansible-playbook start.yml
```
This will start metad, graphd, and storaged services on the cluster.

You can also add `-t metad/graphd/storaged` to start only specific service.

```shell
$ ansible-playbook start.yml -t graphd
```
This command will start only the graphd services on the cluster.

### Stop nebula services

Stop nebula services on machines of cluster

```shell
ansible-playbook stop.yml
```
This command will stop all of the services on the cluster.

You can also add `-t metad/graphd/storaged` to stop only specific service.
