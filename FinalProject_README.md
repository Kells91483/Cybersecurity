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

![](https://github.com/Kells91483/Cybersecurity/blob/main/FFP4.jpg)

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

### Description of Target

The target of this attack was: Target 1 192.168.1.110

Target 1 is an Apache web server and has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers. As such, the following alerts have been implemented:

### Monitoring the Targets

Traffic to these services should be carefully monitored. To this end, we have implemented the alerts below:

Name of Alert 1

- HTTP Request Size Monitor is implemented as follows:
  - Metric: Packetbeat 7.8.0
  - Threshold: 3500 request in span of a minute
  - Vulnerability Mitigated: Directory Transversal as well as other potential remote attacks being performed by malicious actors.
  - Reliability: This will break down into 58 requests per second which should not lead to an amount of false positives that will create alert fatigue, keeping the SOC effective. Medium Reliability.

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Defensive%20Images/HTTPSize.jpg)

Name of Alert 2

- Excessive HTTP Errors:
  - Metric: Packetbeat 7.8.0
  - Threshold: Above 400 request for the last 5 minutes
  - Vulnerability Mitigated: Brute Force Attack or DDOS
  - Reliability: This will break down into 0.75 requests per second which should not lead to an amount of false positives that will create alert fatigue, keeping the SOC effective. High Reliability.

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Defensive%20Images/HTTPErrors.jpg)

Name of Alert 3

- CPU Usage Monitor:
  - Metric: Metricbeat 7.7.0
  - Threshold: Above 0.5 for the last 5 minutes
  - Vulnerability Mitigated: Brute Force Attack, Possible Malware, or Cryptomining
  - Reliability:  This will alert if the CPU usage is greater than 50%, this could potentially create an excessive amount of false alerts. This may need to be raised to 0.6 or 60% usage. Potential Low Reliability.

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Defensive%20Images/CPUusage.jpg)

### Suggestions for Going Further

The logs and alerts generated during the assessment suggest that this network is susceptible to several active threats, identified by the alerts above. In addition to watching for occurrences of such threats, the network should be hardened against them. The Blue Team suggests that IT implement the fixes below to protect the network:

- Excessive HTTP Errors
  - Patch: Implement WordPress Hardening by installing WordPress security plugin, website firewall such as Sucuri.
  - Why It Works: The Sucuri Firewall is a cloud-based WAF that stops website hacks and attacks. It routes traffic through the firewall on the DNS level and back to the WordPress host. Updates with virtual patching and hardening, prevents wpscan access and blacklist suspected lPs.

- HTTP Request Size Monitor
  - Patch: Configure HTTP request limits on the web server
  - Why It Works: Request limits define the validation criteria for incoming requests by enforcing size limits on HTTP request header fields. The requests that have fields larger than the specified maximums are dropped. This can mitigate buffer overflow exploits and prevent DoS attacks.

- CPU Usage Monitor
  - Patch: Install cpulimit, limits excessive CPU use by setting limits. Install with apt-get install cpulimit
  - Why It Works: Cpulimit enables the user to set limits on the CPU usage that is found to be consuming excessive CPU by using top. It can work in conjunction with this programmed alert. If the alert is triggered the notified SOC analyst can detect which process is consuming the most by using top and then can set a percentage limit utilizing cpulimit.


### Additional Suggestions for Going Further

The logs and alerts generated during the assessment suggest that this network is susceptible to several active threats, identified by the alerts above. In addition to watching for occurrences of such threats, the network should be hardened against them. The Blue Team suggests that IT implement the fixes below to protect the network:

- Exposed/Unfiltered  Port 22
  - Patch: Filter or whitelist specific IP addresses to connect using Port 22 with a firewall rule.
  - Why It Works: This creates another layer of security from remote malicious actors by only allowing certain endpoints to connect using Port 22.

- Broken link leading to 192.168.1.110/wordpress
  - Patch: Broken Link Checker plug-in for WordPress
  - Why It Works: Automates the process of checking for Broken Links that could expose URLs that will lead to backdoor resources to the general public.

- Weak User Passwords
  - Patch: sudo apt-get install libpam-pwquality  
  - Why It Works: Forces users to only use complex passwords with upper, lower and special characters after editing the command-password file at the following directory: /etc/pam.d/. This will help prevent the creation of easily guessed passwords.


# Network Stage

## Network Forensic Analysis Report

Time Thieves

You must inspect your traffic capture to answer the following questions:

1. What is the domain name of the users' custom site?

        Frank-N-Ted

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Network%20Images/FrankNTed.jpg)

2. What is the IP address of the Domain Controller (DC) of the AD network?

        10.6.12.157

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Network%20Images/DCAD.jpg)

3. What is the name of the malware downloaded to the 10.6.12.203 machine?
        
        June11.dll

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Network%20Images/June.jpg)

  - Once you have found the file, export it to your Kali machine's desktop.

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Network%20Images/KaliDesktopJune.jpg)

4. Upload the file to VirusTotal.com.

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Network%20Images/VirusTotal.jpg)

5. What kind of malware is this classified as?

        Trojan

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Network%20Images/Trojan.jpg)

Vulnerable Windows Machine

1. Find the following information about the infected Windows machine:

        Host name: Rotterdam-PC
        IP address: 172.16.4.205
        MAC address: 00:59:07:b0:63:a4

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Network%20Images/RotterdamPC.jpg)

2. What is the username of the Windows user whose computer is infected?

        admin-ajax
 
![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Network%20Images/ajax.jpg)
 
3.  What are the IP addresses used in the actual infection traffic?

        172.16.4.205, 31.7.62.214, 185.243.115.84, 166.62.111.64

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Network%20Images/IPaddressTraffic.jpg)

4. As a bonus, retrieve the desktop background of the Windows host.

        Potentially one of the pictures that are saved on the desktop below: 
        
![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Network%20Images/PossibleDesktopImages.jpg)        
        
Illegal Downloads

1. Find the following information about the machine with IP address 10.0.0.201:

        MAC address: 00:16:17:18:66:c8
        Windows username: elmer.blanco
        OS version: Windows 10 64 Bit
        
![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Network%20Images/IllegalDown1.jpg)

2. Which torrent file did the user download?
	
        Btdownload.php...Betty_Boop_Rhythm_on-the_Reservation.avi.torrent

![](https://github.com/Kells91483/Cybersecurity/blob/main/Final%20Project/Network%20Images/TorrentDownload.jpg)
![](https://github.com/Kells91483/Cybersecurity/blob/main/TorrentDownload2.jpg)




