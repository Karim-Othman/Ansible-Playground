# Ansible playbooks

![Ansible logo](https://www.marksei.com/wp-content/uploads/2019/06/Ansible-Logo-720x210.png)


This playground repo contains a collection of Ansible playbooks 

# common environment Setup
 environment exists on linux academy playground 

![Cluster](https://user-images.githubusercontent.com/17851915/104832522-4aea2980-589a-11eb-8d00-32d259ea5f60.png)
OS version for all machines: Ubuntu 18.04 LTS (Bionic Beaver)

for ansible installation follow this [link](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-ubuntu-18-04)
- host names
     - Ansible main machine:  KarimOthman2c.mylabserver.com 
     - machine 1:  KarimOthman1c.mylabserver.com (trusted ssh)
     - machine 2:  KarimOthman3c.mylabserver.com 
        ***All machines has cloud_user sudoer user ***
![image](https://user-images.githubusercontent.com/17851915/104844744-9e7c6780-58da-11eb-9b33-33ce3d4d62b0.png)
- Inventory: create custom inventory.yml file in local directory (mentioned in this repo)
	
- To configure trust between main machine and machine 1 follow this [link](https://documentation.clearos.com/content:en_us:kb_o_setting_up_ssh_trust_between_two_servers)

- ping each machine with ansible ad-hoc command to ensure that all is set as planned!
![image](https://user-images.githubusercontent.com/17851915/104845039-0d0df500-58dc-11eb-86d1-eb7bed2d23a2.png) 

# Playbooks list

| Playbook |
| ---------- |
| [Create users](#create_users) |
| [Setup tomcat server](#setup-tomcat-server) |

## Create users
this playbook should create two local users on a remote machine
- restrictions:
	- don't repeat the module twice (use loop)
	- passwords should be random
	- create home directories for the users
	- users must be sudoers

### implementation steps
1- you will need privilege escalation in order to create user at remote host; use become in playbook to do so

2- create password.yml with ansible-vault to save ansible_become_pass
 
    $ansible-vault create password.yml

3- you will be permitted to insert vault password (remember this pass)

4- vi terminal will open up; write your ansible_become_pass in yml format
ansible_become_pass: xx

5- create "CreateUsers_Playbook.yml" file
- This playbook should
    -  run only on trustedworker
    -  use become for privilege escalation
    -  refer to "ansible_become_pass" in password.yml
    -  lookup to generate random pass [documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/password_lookup.html)
    -  groups should be set with sudo
    -  create_home should set to true in order to create local directory
    -  apply loop for the same task for each user

6- Run playbook using the below command and insert vault pass when permitted


    $ansible-playbook -i inventory.yml CreateUsers_Playbook.yml --ask-vault-pass --extra-vars '@password.yml'
## Setup tomcat server
this playbook should setup and start tomcat server on remote machine
- restrictions:
	- Connect to the server with username/password (without the use of trust keys)
	- Change Tomcat default port to 9095
	- Deploy any Single HTML file
	- Start the service
	- Stop the service

### implementation steps
1- This playbook should run over non-trusted ssh server. so, we must provide ssh password when login

2- reuse the same password file and edit it to add ssh pass

    $ansible-vault edit password.yml

3- insert vault pass when permitted, then add ssh_pass in yml format
ssh_pass: xx


4- follow along with this [link](https://www.bogotobogo.com/DevOps/Ansible/Ansible-Tomcat9-Ubuntu18-Playbook.php) to setup tomcat server 

5- Create tomcat.service file in local directory to be copied to dest machine/server at ansible playbook execution time

6- create "SetupTomcat.yml" playbook

7- modify SetupTomcat.yml to update default port located in '/opt/tomcat/conf/server.xml <Connector port="8080"' using Ansible replace module. ******adding "name: Replace default port" task******

8- create HelloWorld.html and modify SetupTomcat.yml to move this html file to "/opt/tomcat/webapps/ROOT" at remote machine. ******adding "name: Deploy html file" task******

9- play SetupTomcat.yml and insert vault pass when permitted

        $ansible-playbook -i inventory.yml SetupTomcat.yml --ask-vault-pass --extra-vars '@password.yml'

10- goto remote machine and curl to check deployed html file
![image](https://user-images.githubusercontent.com/17851915/104847234-80693400-58e7-11eb-83ca-e7c61c4181df.png)


11- modify SetupTomcat.yml to add stop tomcat service as required. ******adding "name: stop Tomcat service" task******

12- play SetupTomcat.yml and insert vault pass when permited

        $ansible-playbook -i inventory.yml SetupTomcat.yml --ask-vault-pass --extra-vars '@password.yml'

13- goto remote machine and check if tomcat process is up

        $ ps -ef | grep -i tomcat
![image](https://user-images.githubusercontent.com/17851915/104847328-07b6a780-58e8-11eb-817b-419616b41869.png)
