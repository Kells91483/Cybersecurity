## The Final Cybersecurity Bootcamp Project

Offensive Stage

You are working as a Security Engineer for X-CORP, supporting the SOC infrastructure. The SOC analysts have noticed some discrepancies with alerting in the Kibana system and the manager has asked the Security Engineering team to investigate.
To start, your team needs to confirm that newly created alerts are working. Once the alerts are verified to be working, you will monitor live traffic on the wire to detect any abnormalities that aren't reflected in the alerting system.
You will then report back all your findings to both the SOC manager and the Engineering Manager with appropriate analysis.

## Red Team: Summary of Operations

Table of Contents

    -Exposed Services

    -Critical Vulnerabilities

    -Exploitation

    -Exposed Services

Nmap scan results for each machine reveal the below services and OS details:

$ nmap -A -T4 192.168.1.110
  
This scan identifies the services below as potential points of entry:

Target 1

    Port 80: Apache httpd 2.4.10

    Port 22: SSH OpenSSH 6.7p1

    Port 111: rpcbind 2-4

    Port 139 netbios-ssn Samba smbd 3.X-4.X

    Port 445: netbios-ssn Samba smbd 4.2.14-Debian

The following vulnerabilities were identified on target:

Target 1

        Lack of password parameters to ensure strong, complex user passwords, not being able to include any of the user name in the password.

        Open Port 22 that was not filtered by any firewall rules to only allow certain remote IP addresses to remote in.

        Open Port 80 allowed us to connect to the website. While performing recon using OSINT, we examined the HTML code to potentially execute a code quality attack with          possible orphaned web developer notes.

        Potential faulty implementation of Least Privilege access to users. User Michael having access to the wp-config.php in the /var/www/html/wordpress directory as well as the wordpress database, may not have been needed for his role.

        Broken links on the web page that lead to a backdoor URL: 192.168.1.110/wordpress.

![](
