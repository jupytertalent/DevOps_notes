Ansible is an open-source software provisioning, configuration management, and application-deployment tool. It runs on many Unix-like systems, and can configure both Unix-like systems as well as Microsoft Windows. It includes its own declarative language to describe system configuration. DevOps teams use Ansible to deploy and manage applications on servers. Ansible connects to nodes and pushes out small programs, called "Ansible modules" to them. These programs are written to be resource models of the desired state of the system. Ansible then executes these modules over SSH and removes them when finished.

manual instllation means you should the all the things

/etc/ansible/hosts —> slave system ips

/etc/ansible/ansible.cfg  —> config file 

ADHOC COMMANDSk

ansible all -i slave.txt -m ping  —> check the slave connected or not 

ansible all -i slave.txt - a uname  —> action based cmd

ansible all -i slave.txt -m apt -a “name=<httpd software name> state=present” 

ansible all -i slave.txt -m service-a “name=<httpd software name> state=started”  #start some service means use this

ansible all -i slave.txt -m copy -a “src=<location of the file index.html(master) dest=”/var/www/html/index.html <master file name >”>” 

automated means we never use -i slave.txt

# noted words

install - present

uninstall - absent

start - started

stop - stopped

restart - restarted

if need a root permission means use -b 

# yaml

```yaml
- hosts : all
	remote_user: ec2-user (master node name)
	become: yes
	tasks:
		- name: Install Apache2 and host a site
	  tasks:
			    - name: Install Apache2
		      apt:
		        name: apache2
		        state: present
		
		    - name: Enable Apache2 mod_rewrite
		      apache2_module:
		        name: rewrite
		        state: present
		
		    - name: Start Apache2 service
		      service:
		        name: apache2
		        state: started
		        enabled: yes
		
		    - name: Copy index.html to remote server
		      copy:
		        src: /path/to/your/local/index.html
		        dest: /var/www/html/index.html
		        owner: www-data
		        group: www-data
		        mode: "0644"
		      notify:
		        - Restart Apache2
		
		  handlers:
		    - name: Restart Apache2
		      service:
		        name: apache2
		        state: restarted

```

ansible-playbook -i slave.txt basic.yaml

loop

```yaml

- name: install a application via loop
  apt:
		  name : "{{item}}"
		  state : present
- loop:
		- php
		- mysql
		- unzip
```





vpc ---> subnet 

- private
- public 2 --> internet -- > 2 ec2 