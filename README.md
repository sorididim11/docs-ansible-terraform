# Features
* Support vagrant, terraform for openstack
* Install DC/OS cluster based advanced custom installation of DC/OS which is upgradable
  - Install/configure prerequisites of DC/OS including docker
  - Generate config.yml file
  - Generate ip detec file 
  - Deploy DC/OS cluster 
* Support both Enterprise and Commmunity  DC/OS 
  - Install Open source or Enterprise DC/OS based on group variable of ansible, dcos_is_enterprise(True or False)
* Enterprise DC/OS
  - Upgrade security mode after installation
  - Reserve resources dynamically, create service accout with secret, assign permissions to the service accounts and install packages based on description files called pkg-desc in playbooks with any security mode.
  - fix DC/OS 1.9 bug in strict mode, when reserving resource dynamically
* Add/remove nodes after installation. 
* Install packages 
  - Install dcos cli & enterprise
  - Install secure/insecure docker registy by creating/updating certificate automactically on each node. 
  - Install monitoring platform based on cadvisor, influxdb and grafana

## Test Enviornment

* OS: Centos above 7.2 (DC/OS requirement)
* ansible:  2.3.1
* vagrant: 1.9.7
* terraform 


## Getting started
### terraform 
1) Clone the project 
2) install terraform 
3) go to dcos-ansible-terraform/terraform
3) source ihos.osrc - set credential for openstack for example. 
4) modify number of masters/slaves/slaves_public in variable.tf
5-1) check out security you wanna set. -  ansible/inventories/helion/group_vars/all.yml
5-1) check out dcos license in case you use enterprise DC/OS. -  ansible/inventories/helion/group_vars/all.yml 
6) terraform apply


## General Parameters for both community and enterprise version 

* dcos_is_enterprise: True or False - enterprise dcos or open source 
* dcos_cluster_name:               - user-defined cluster name
* dcos_bootstrap_root_path: /dcos  - bootstrap installation home   
* docs_boostrap_port: 9000         - bootstrap web server port for distributing installer files 
* dcos_nic_name: eno1              - change Network Inferface card for private ip detection 
* dcos_nic_pub_name: eno1          - change Network Inferface card for public ip detection 
* dcos_is_use_proxy: false         - define when proxy is used 
* dcos_resolver1:                  - dns1 ex) 8.8.8.8
* dcos_resolver2:                  - dns2 
* proxy_env:
  + example)
    proxy_env:
      dummy: dummy
      http_proxy: http://web-proxy.corp.hp.com:8080
      https_proxy: http://web-proxy.corp.hp.com:8080
      no_proxy: ".mesos,.thisdcos.directory,.dcos.directory,.zk,127.0.0.1,localhost,{{ groups['all'] | map('extract', hostvars, ['ansible_' + dcos_nic_name, 'ipv4', 'address']) | join(',') }}"


## Enterprise Parameters when dcos_is_enterprise: True

* dcos_license: <license>          - when enterise dcos used.
* dcos_superuser: root             - enterprise dcos superuser
* dcos_superuser_pwd: root         - enterprise dcos superuser pwd
* edcos_security_mode: permissive  # security type  disabed, permissive, strict 


## docker setting 

* docker_volume_fs                  - default docker overlayfs (DC/OS 1.9 mendatory param for )
* docker_version:                   - default 1.13.1 (DC/OS 1.9 mendatory param for )
* dcos_is_secure_registry: True  - install docker private regisry in insecure or secure 
* dcos_host_volume_registry: '/tmp' - docker registry volume path. ex) /tmp/nfs-share or host diretory 


## One-touch Install  

* install all prerequistes, docker, DC/OS cluster, CLI, Enterprise CLI, docker private registry, monitoring platform 
    * ansible-playbook -i <inventory file> deploy-dcos.yml 


## Manual Install 

1) Install & configure prerequisite of DC/OS
 ansible-playbook -i <inventory file> step1-deploy-precondtions.yml 

2) Install & configure docker engine 
 ansible-playbook -i <inventory file> step2-deploy-docker.yml 

3) Generate DC/OS config and Build installers files.
 ansible-playbook -i <inventory file> step3-build-bootfiles.yml

4) Deploy cluster 
 ansible-playbook -i <inventory file> step4-deploy-cluster.yml 

5) Install DC/OS CLI + Enterprise CLI for only enterprise version & login automatically 
 ansible-playbook -i <inventory file> pkg-install-cli.yml 

6) Install secure/insecure docker private registry in the cluster (self-signed certificate)
 ansible-playbook -i <inventory file> pkg-install-docker-registry.yml 

6) Install monitoring platforom consisting of cadivior, influxdb, grafana 
 ansible-playbook -i <inventory file> pkg-install-monitoring.yml 


## Dynamic Reservation 

Reserve resources dynamically and Install custome marathon on it.
first specifiy roles including cpu, mem, gpu, disk and so on  quota.json in roles/dcos/node/quota/files/quota.json 

ansible-playbook -i <inventory file> op-add-quota.yml 

check the state of agents after the reservation 
dcos node --json

