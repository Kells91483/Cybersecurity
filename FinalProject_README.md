# The Final Cybersecurity Bootcamp Project

## Offensive Stage

You are working as a Security Engineer for X-CORP, supporting the SOC infrastructure. The SOC analysts have noticed some discrepancies with alerting in the Kibana system and the manager has asked the Security Engineering team to investigate.
To start, your team needs to confirm that newly created alerts are working. Once the alerts are verified to be working, you will monitor live traffic on the wire to detect any abnormalities that aren't reflected in the alerting system.
You will then report back all your findings to both the SOC manager and the Engineering Manager with appropriate analysis.


## Red Team: Summary of Operations

- Exposed Services
- Critical Vulnerabilities
- Exploitation
- Exposed Services

Nmap scan results for each machine reveal the below services and OS details:

    $ nmap -A -T4 192.168.1.110
  
 This scan identifies the services below as potential points of entry:

Target 1

- Port 80: Apache httpd 2.4.10
- Port 22: SSH OpenSSH 6.7p1
- Port 111: rpcbind 2-4
- Port 139 netbios-ssn Samba smbd 3.X-4.X
- Port 445: netbios-ssn Samba smbd 4.2.14-Debian

The following vulnerabilities were identified on target:

Target 1

- Lack of password parameters to ensure strong, complex user passwords, not being able to include any of the user name in the password.
- Open Port 22 that was not filtered by any firewall rules to only allow certain remote IP addresses to remote in.
- Open Port 80 allowed us to connect to the website. While performing recon using OSINT, we examined the HTML code to potentially execute a code quality attack with possible orphaned web developer notes.
- Potential faulty implementation of Least Privilege access to users. User Michael having access to the wp-config.php in the /var/www/html/wordpress directory as well as the wordpress database, may not have been needed for his role.
- Broken links on the web page that lead to a backdoor URL: 192.168.1.110/wordpress.

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Offensive%20Images/NmapScan.jpg)

Exploitation

The Red Team was able to penetrate Target 1 and retrieve the following confidential data:

Target 1
    
    Flag1.txt: {b9bbcb33e11b80be759c4e844862482d}

- Exploit Used
  - Navigate to 192.168.1.110 using a browser.
  - Performing reconnaissance using OSINT to find potential code quality vulnerability by opening the different links on the page and exploring the HTML code.
  - Found flag1 on the Service page by searching for “flag”

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Offensive%20Images/Flag1.jpg)

        Flag2.txt: {fc3fd58dcdad9ab23faca6e9a34e581c}

- Exploit Used
  - Remote SSH connection after identifying Michael as a user, using wpscan.
  - Wpscan --url http://192.168.1.110/wordpress --enumerate u

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Offensive%20Images/wpscanfl2.jpg)

  - Exploited the weak password for user “michael” by guessing the password was “michael”.
  - Exploited the open Port 22 SSH with command: ssh michael@192.168.1.110 and using the password “michael”
  - Searched for the flag by navigating to the root directory and using command: find -type f -iname ‘flag*’

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Offensive%20Images/SSHCommand.jpg)

  - The search results rendering the “flag2.txt” file in directory /var/www/:

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Offensive%20Images/SearchFl2.jpg)

  - Using the cat command to display the content of the “flag2.txt” file:

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Offensive%20Images/CatFlag2.jpg)

        Flag3.txt: {afc01ab56b50591e7dccf93122770cd2} and Flag4.txt: {715dea6c055b9fe3337544932f2941ce}

- Exploit Used
  - Inspecting wp-config.php, while connected to Michael, to look for root access password to MySQL database, discovered user name and password.
  - Cat /var/www/html/wordpress/wp-config.php 
 
 ![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Offensive%20Images/DBCreds.jpg)
 
    Logged into MySQL server with the following command: Mysql -u root -PR@v3nSecurity - D wordpress
    
![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Offensive%20Images/mysqlcommand.jpg)

    Used the following command to show the tables present in the database: SHOW TABLES;
    
![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Offensive%20Images/ShowTables.jpg)

    Used the following command to inspect communications on the website: SELECT * FROM wp_posts;
  - Discovered Flags 3 and 4 within wp_posts.

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Offensive%20Images/Flag3and4.jpg)


## Defensive Stage

## Blue Team: Summary of Operations

Table of Contents
- Network Topology
- Description of Targets
- Monitoring the Targets
- Patterns of Traffic & Behavior

Suggestions for Going Further

Network Topology



The following machines were identified on the network:

- Target 1:
  - Operating System: Linux  3.6
  - Purpose: Exposes a vulnerable WordPress server.
  - IP Address: 192.168.1.110

- Target 2
  - Operating System: Linux 4.15
  - Purpose: A Bonus victim machine to attack if time permits.
  - IP Address: 192.168.1.115

- Kali
  - Operating System: Kali Linux 5.4.0
  - Purpose: A standard Kali Linux machine for use in the penetration test for Target 1.
  - IP Address: 192.168.1.90

- ELK
  - Operating System: Linux 4.15.0
  - Purpose: The same ELK setup created in Project 1. It holds the Kibana dashboards.
  - IP Address: 192.168.1.100

- Capstone
  - Operating System: Linux 4.15.0
  - Purpose: Filebeat and Metricbeat are installed and will forward logs to the ELK machine.
  - IP Address: 192.168.1.105

Description of Targets
The target of this attack was: Target 1 192.168.1.110
Target 1 is an Apache web server and has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers. As such, the following alerts have been implemented:
Monitoring the Targets
Traffic to these services should be carefully monitored. To this end, we have implemented the alerts below:
Name of Alert 1
HTTP Request Size Monitor is implemented as follows:
Metric: Packetbeat 7.8.0
Threshold: 3500 request in span of a minute
Vulnerability Mitigated: Directory Transversal as well as other potential remote attacks being performed by malicious actors.
Reliability: This will break down into 58 requests per second which should not lead to an amount of false positives that will create alert fatigue, keeping the SOC effective. Medium Reliability.
 



