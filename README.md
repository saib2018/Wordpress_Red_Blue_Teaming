# Wordpress_Red_Blue_Teaming

![Network_Diagaram](Network_Diagaram.png)

# Red Team: Summary of Operations

## Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

### Exposed Services

- Target 1
  - List of Exposed Services
     
Ports - 22/tcp - ssh
        80/tcp - http
        111/tcp - rpcbind
Nmap scan results for each machine reveal the below services and OS details:

```bash
$ nmap  -sV -sC 192.168.1.110

```
 ![nmap](nmap.png)
 
This scan identifies the services below as potential points of entry:
_TODO: Fill out the list below. Include severity, and CVE numbers, if possible._

The following vulnerabilities were identified on each target:
- Target 1
  - List of Critical Vulnerabilities

CVE-2008-5161 ssh remote login was active at the user level with port 22 being open
CVE-2017-7760 exposed username which allowed brute force of password information. User access to the wp-config.php file via nano. This exposed the root user and password. 

_TODO: Include vulnerability scan results to prove the identified vulnerabilities._

### Exploitation
_TODO: Fill out the details below. Include screenshots where possible._

The Red Team was able to penetrate `Target 1` and retrieve the following confidential data:
- Target 1
  - `flag1.txt`: b9bbcb33e11b80be759c4e844862482d
    - **Exploit Used** - Unprotected HTML file
      
      After logging to Target 1 as michael, looked around for a unprotected files and found the flag in service.html
                        cat /var/www/html/service.html | grep flag
            
![flag1](flag1.png)

  - flag2.txt: fc3fd58dcdad9ab23faca6e9a36e581c
    - **Exploit Used**  - Remote Login with users with weak passwords
     
      Wpscan in the previous step had provided with the users that had access on the servers
 
 ![Users](Users.png)
 
After obtaining the user name ‘michael’ with WPScan, an attacker can attempt to login via SSH. After several attempts with weak passwords such as ‘password’, ‘12345’, a successful login is made by using ‘michael’ as the password. 

 ![ssh](ssh.png)

After gaining access to the server, browsed different file paths in linux and identified the flag in /var/www folder

 ![flag2](flag2.png)
 

- flag3.txt:afc01ab56b50591e7dccf93122770cd2 

 ![flag3](flag3.png)

- flag4.txt: 715dea6c055b9fe3337544932f2941ce 
 
 ![flag4](flag4.png)
 
    - **Exploit Used** - Unprotected PHP file containing database credentials

After gaining access to the server using michael’s account, identified an unprotected file under php file under /var/www/html/wordpress

 ![wordpress1](wordpress1.png)

Wp-config.php file contained the credential information for the mysql database - wordpress

 ![mysqlinfo](mysqlinfo.png) 

Using the above information, logged into mysql and viewed the available databases 

![mysql1](mysql1.png) 

Navigated to the “wordpress” database and perform a select query to identify the contents of the DB tables

![wordpressdb](wordpressdb.png) 

Select query from wp_posts table displayed both flags -3 and 4

![mysqlquery](mysqlquery.png) 
 
![flags_3_4](flags_3_4.png) 
 

flag4.txt: 715dea6c055b9fe3337544932f2941ce 
One more way to find flag 4
Exploit used -  escalating to root privileges

Navigated to wp_users table in sql to identify the hash values corresponding to michael & steven


 

Copying the hash values to a text file named - wp_hashes.txt with following format 
user1:$P$hashvalu3
user2:$P$hashvalu3

and running john the ripper against the text file

 

Using the password cracked for steven, logging to the server using steven’s credentials


 

Identified that we could run python script with Steven’s account using sudo -l command

 

Utilized the below python script to escalate to root account on the server
 
After gaining root access, navigated to root folder to find the flag 4
 

# Blue Team: Summary of Operations

## Table of Contents
- Network Topology
- Description of Targets
- Monitoring the Targets
- Patterns of Traffic & Behavior
- Suggestions for Going Further

### Network Topology
_TODO: Fill out the information below._

The following machines were identified on the network:
- Name of VM 1
  - **Operating System**: Linux
  - **Purpose**: First Target
  - **IP Address**: 192.168.1.110


### Description of Targets
_TODO: Answer the questions below._

The target of this attack was: `Target 1` 192.168.1.110

Target 1 is a web server and has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers. As such, the following alerts have been implemented:

### Monitoring the Targets

Traffic to these services should be carefully monitored. To this end, we have implemented the alerts below:

Excessive HTTP Errors

[Excessive HTTP Errors] is implemented as follows:

-	Metric: HTTP Errors
-	Threshold: Above 400 for the last 5 minutes
-	Vulnerability Mitigated: Brute Force Attacks
-	Reliability: High Reliability

![Excessive_HTTP_Errors](Excessive_HTTP_Errors.png) 


HTTP Request Size Monitor

[HTTP Request Size Monitor] is implemented as follows:

-	Metric: http.request.bytes
-	Threshold: Above 3500 for the last minute
-	Vulnerability Mitigated: Denial of Service Attacks.
-	Reliability: High Reliability.
  
![HTTP_Request_Size_Monitor](HTTP_Request_Size_Monitor.png) 


CPU Usage Monitor


[CPU Usage Monitor] is implemented as follows:

-	Metric: system.process.cpu.total.pct
-	Threshold: Above 0.5 for the last 5 minutes.
-	Vulnerability Mitigated: Excessive CPU Usage
-	Reliability: Medium Reliability.
 
![CPU_Usage_Monitor](CPU_Usage_Monitor.png) 

### Suggestions for Going Further

- Each alert above pertains to a specific vulnerability/exploit. Recall that alerts only detect malicious behavior, but do not stop it. For each vulnerability/exploit identified by the alerts above, suggest a patch. E.g., implementing a blocklist is an effective tactic against brute-force attacks. It is not necessary to explain _how_ to implement each patch.

The logs and alerts generated during the assessment suggest that this network is susceptible to several active threats, identified by the alerts above. In addition to watching for occurrences of such threats, the network should be hardened against them. The Blue Team suggests that IT implement the fixes below to protect the network:


**Vulnerability 1: Brute Force Attacks**

-	Patch: Invalid Credentials Lock out. 
-	Why It Works:  It limits the number of attempts the attacker can commit. 

**Vulnerability 2: DOS Attacks**

-	Patch: Deploy a Load Balancer 
-	Why It Works: Distributes connection to a number of servers to reduce the load

**Vulnerability 3: Excessive CPU Usage** 

-	Patch: Limit max cpu usage for each core.
-	Why It Works:  To avoid server resource constraints and failure of server functionality due to server utilization
