# Ansible setup
---

1. what is ansible
2. basic commands
	1. files located in /etc/ansible
		1. ansible.cfg
		2. hosts
			1. inventory list of hosts for ansible to control
			2. hosts will need their ip address staticaly set to ensure playbook stays current
		3. roles folder
	2. in hosts file you define group
		1. \[group-name] 
	3. login to device
		1. ansible_user=***ansible user on server***
		2. ansible_password=***password*** 
			1. this is only for password authentication
			2. if you are using passwords you need to disable the host key checking.
				1. go to the ansible.cfg
				2. find **host_key_checking** ensure it is false
	4. to make sure your hosts are up you can ping them with the following
		1. ansible group-name -m ping
	5. to run commands via adhoc
		1. ansible group-name -a "ls"
3. how i will be using it
	1. updating truenas/proxmox and all my vms
	2. setting up proxmox automaticaly after install
	3. setting up debian/ubuntu server environment for apache web server
	4. setting up docker host on ubuntu
4. network/system automation
	1. how to control network devices
	2. how to control systems
	3. graceful shutdown on power outage
	4. graceful power on when power is back on using autostart bash script
5. playbook creation
	1. condensing jobs into playbooks
		1. will be in yaml format
			1. find example online for basic one or ask chat gpt
	2. to run a playbook do the following
	3. ansible-playbook playbook-file.yml
	4. 
6. authentication methods
	1. password
	2. ssh key