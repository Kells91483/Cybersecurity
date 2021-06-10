##Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the playbook file may be used to install only certain pieces of it, such as Filebeat.

my_playbook.yml

install-elk.yml

filebeat-playbook.yml

metric-beat.yml



This document contains the following details:

- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

- The load balancers will address the availability of the web servers as it will monitor the available resources for the servers, and also if the server is available to accept connections.  If one of the servers is unavailable, all connections will be sent to surviving servers, until the unavailable server is responding again. Once responding, the server will be put back into the rotation by the load balancer. This ensures minimal downtime to DVWA and also helps debilitate or prevent DDoS attacks as well.

The jump box is designed as a portal into the internal network. With a jump box, traffic trying to enter the internal network can be monitored more effectively and firewall rules are able to be assigned accordingly to fewer devices.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the network, configuration, and system logs.

The ELK server will be collecting system logs for the web servers using Filebeat. As different resources may send files with minor differences, such as timestamp formatting, using Filebeat along with ElasticSearch/LogStash provides a consistent formatting for the logs to make them easier to read and search through. The use of these systems will allow for easier consumption of multiple logs from multiple resources with uniformity to the search parameters.

As often the case, we also need to monitor the resources that are being used by a node on the network. Metricbeat will enable us to collect these resources and take action if we see any nodes reporting resources that are not the baseline of activity.


The configuration details of each machine may be found below.

|         Name         	|   Function   	| IP Address    	| Operating System     	|
|:--------------------:	|:------------:	|---------------	|----------------------	|
| Jump-Box-Provisioner 	| Access Point 	| 104.211.13.46 	| Linux (ubuntu 18.04) 	|
| Web-1                	| DVWA         	| 10.0.0.7      	| Linux (ubuntu 18.04) 	|
| Web-2                	| DVWA         	| 10.0.0.8      	| Linux (ubuntu 18.04) 	|
| Elk-Server1          	| ELK Stack    	| 10.1.0.4      	| Linux (ubuntu 18.04) 	|


### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump-Box-Provisioner machine can accept connections from the Internet. Access to this machine is only allowed from the following IP address:

Whitelisted IP address: 
174.106.81.87


Machines within the network can only be accessed by Jump-Box-Provisioner.

A summary of the access policies in place can be found in the table below:

|         Name         	      | Publicly Accessible |       IP Addresses        |
|:--------------------:	      |:------------------:	|:------------------------:	|
| Jump-Box-Provisioner        |         Yes        	|       174.106.81.87      	| 
|        Web-1        	      |         No         	|         10.0.0.7         	|
|        Web-2        	      |         No         	|         10.0.0.8         	|
|      Elk-Server1     	      |         No         	|         10.1.0.4         	|
|      Elk-Server1     	      |         Yes        	| 157.56.166.106 (Dynamic) 	|

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because it saves time due to its simplicity of YAML, changes can be easily made, involves multiple machines and it’s agentless nature.

The playbook implements the following tasks:

Download and install the needed software for the image (Docker, Python Docker, etc.)
Installs the configured image.
Configures the settings if needed (Memory usage, ports, etc.)
Starts the software, forcibly if required.

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.


### Target Machines & Beats

This ELK server is configured to monitor the following machines:
10.0.0.7 (Web-1)
10.0.0.8 (Web-2)

We have installed the following Beats on these machines:
Web-1
Web-2

These Beats allow us to collect the following information from each machine:
Filebeat will allow forwarding of log output to a centralized location for access and analysis for Web-1 and Web-2. 
Metricbeat specifically collects metrics from your systems and services such as CPU, memory, network and disk statistics on Web-1 and Web-2



### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
Copy the filebeat-playbook.yml file from /ansible to /ansible/roles.
Update the hosts file to include the IP addresses of the Webservers (Web-1, Web-2).
Run the playbook, and navigate to http://157.56.166.106:5601/app/kibana to check that the installation worked as expected.


**Bonus** 

How to guide to setup the Determined_napier Container and run the playbooks:
SSH Into the Jump-Box-Provisioner: ssh azadmin@104.211.13.46
Start Docker: sudo docker start determined_napier
Attach Docker: sudo docker attach determined_napier
Navigate to the ansible directory: cd /etc/ansible
Run the ansible playbook located in that directory: ansible-playbook pentest.yml, ansible-playbook install-elk.yml
Navigate to the ansible directory /roles: cd roles/
Run the ansible playbooks located in that directory: ansible-playbook filebeat-playbook.yml, ansible-playbook metricbeat-playbook.yml


Proof of Performance: 

One of the tests was performed by a script created to loop SSH access to Web-1 with the incorrect credentials to form data that Filebeat was able to log and extract. 

   ELK Stack Filebeat-

Script to emulate: Next page



The failed login attempts:



This test was performed by installing ‘stress’ on Web-1 and Web-1, the command ‘sudo stress --cpu 1’ ran to utilize and overwork the CPU on both machines. This verified Metricbeat was able to gather the info and display this in the dashboard.

Elk Stack Metricbeat-

‘Stress’ on Web-1:


‘Stress’ on Web-2



Metricbeat was also utilized to detect a DoS web attack being performed on Web-1.  This attack was performed by running a script created to perform a continuous loop of 5000 web request, also generating index.html files.

Script:



Baseline:



Attack Detected:


