---
title: WRITE UP OF MY CTF
date: 2023-10-10
categories: [CTF,  WriteUp]
tags: [ctf,writeup]     # TAG names should always be lowercase
image: /assets/images/hackerMachinesWriteUp.png
---

In the following post, I present the write-up of the CTF (Capture The Flag) I have developed. Below, I also share a video that demonstrates the process of solving each of the challenges and the structure of the CTF. I hope you enjoy it.

<p align="center">
    <iframe width="560" height="315" src="https://www.youtube.com/embed/yvZcSENe33Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</p>


## Challenge 1: Becoming an Administrator

The first challenge is to obtain an administrator-level account on the "Hacker Machines" platform by exploiting certain vulnerabilities.
The first step to solve it is to discover a specific endpoint called "/test/" using a tool like Gobuster. This endpoint simulates a test conducted by the platform's developers but was not properly disabled.

![Gobuster scan](/assets/images/image1.png)

By examining this endpoint, you can see that it is a page for checking if a user is an administrator or not.

![Test](/assets/images/image2.png)

The next step would be to perform certain queries to check for any SQL Injection vulnerability. As shown in the following figure by entering `admin’ OR 1=1#`, we can exploit an SQL injection.

![SQLi](/assets/images/image3.png)

Next, there would be several ways to exploit this vulnerability, either by doing it manually or using automated tools for SQL injection exploitation. In this case, we will demonstrate it using tools like SQLMap and BurpSuite.
* SQLMap: An open-source tool designed for automating the detection and exploitation of SQL injection vulnerabilities in web applications.
* Burp Suite: A security toolset designed to assess the security of web applications.

In the described scenario, we will use Burp Suite to capture and save the request made to the server for later use with SQLMap. Burp Suite offers a feature called "Intercept" that allows intercepting and modifying HTTP requests and responses between the client and the server.

![SQLi](/assets/images/image4.png)

Once the request has been saved, it can be used with SQLMap to perform automated SQL injection testing. By providing the captured request, SQLMap will use it as a starting point to perform tests and extract information from the web application.

![SQLi](/assets/images/image5.png)

With the above command, SQLMap automatically displays the contents of the database. As seen in the image, it shows the contents of the "auth_user" table, which contains encrypted user passwords.

![SQLi](/assets/images/image6.png)

SQLMap also cracks password hashes automatically. However, these hashes could also be cracked using tools like Hashcat or John The Ripper.

![SQLi](/assets/images/image7.png)

With this, you can log in with the "admin" user and the password "superman1", and you would find the first flag, as seen in the following image.

![Flag 1](/assets/images/image8.png)

## Challenge 2: Obtaining a Reverse Shell

The next challenge's main objective is to obtain a reverse shell by exploiting a vulnerability in a specific platform feature. A reverse shell is a technique used in the field of cybersecurity that allows an attacker to gain remote access to a compromised system through a reverse connection. Instead of establishing a direct connection from the compromised system to the attacker, as in a typical shell connection, a reverse shell has the compromised system initiate the connection to the attacker.

The way to exploit the existing vulnerability is to inject code using a tool like BurpSuite into the functionality of restarting a virtual machine.

![Restart virtual machine functionality](/assets/images/image10.png)

As you can see in the image, you need to inject code into the "restart" parameter, which indicates which machine to restart. One way to test which is the best method to inject code is by launching the wget command to a server you have deployed with Python for testing.

![Code injection](/assets/images/image11.png)
![Code injection](/assets/images/image12.png)

Once you have figured out how to inject code, you should perform a Reverse Shell. In this case, as Django is a Python framework, Python 3 code is used, as shown in the following image.

![Code injection](/assets/images/image13.png)

On the other hand, you should be listening with netcat on the port you choose to create the connection. Once this is done, as seen in the following image, you would achieve a reverse shell, and you would have a terminal with the "www-data" user.

![Code injection](/assets/images/image14.png)

The last step would be to investigate where the flag might be. In this case, it is located in the "/var/www" directory, which is the typical directory for Apache.

![Code injection](/assets/images/image15.png)

## Challenge 3: Vertical Privilege Escalation

The next challenge involves escalating privileges from the "www-data" user to the "jaime" user. In this scenario, the "www-data" user represents limited access with restricted permissions, while the "jaime" user has higher privileges with access to additional resources and functions in the system.
Privilege escalation is a technique used to elevate access in a system to obtain greater privileges and control over the environment. In this case, the goal is to move from the limitations of the "www-data" user to the privilege level of the "jaime" user.
The privilege escalation in this challenge involves investigating the source code of the web platform for sensitive information. An important piece of information is found in the "settings.py" file, which stores the database password. It turns out that this password matches the one used to access SSH as the "jaime" user.

By examining the source code of the platform, it is possible to identify relevant files like "settings.py" that contain critical configurations and sensitive data. In this case, the database password, which is also used for authentication via SSH as the "jaime" user, is found in this file.

![Flag 3](/assets/images/image16.png)

Once the password is discovered, it is possible to establish an SSH connection with the "jaime" user. By accessing the "jaime" account via SSH, you gain the ability to fetch "flag3".

![Flag 3](/assets/images/image17.png)

## Challenge 4: Horizontal Privilege Escalation

The next challenge aims to achieve horizontal privilege escalation to gain access to the "vero" user. Horizontal privilege escalation involves moving laterally within the system, transitioning from one user account to another, to obtain higher levels of access and control.
The goal is to examine the cron jobs that are running. It can be observed how the "vero" user is executing a file called "backup.sh."

![Flag 4](/assets/images/image18.png)

As shown in the following image, this file has execute permission for other users. By examining it, you can see how to perform a backup using the command `tar -czf /var/importantBackup/backup.tar.gz *`

![Flag 4](/assets/images/image19.png)

The privilege escalation challenge involves exploiting the wildcard vulnerability in the "tar" command to make the cron job executed by the "vero" user also run a Reverse Shell. The wildcard vulnerability in the "tar" command allows a user to manipulate the command's behavior using wildcard characters. In this case, the goal is to exploit this vulnerability so that when the "vero" user's cron job executes the "tar" command, it also triggers a Reverse Shell.

![Flag 4](/assets/images/image20.png)

Once Netcat is configured to listen on port 4444, you can achieve the execution of the Reverse Shell, granting access to the system with the privileges of the "vero" user. This will allow the attacker to perform actions and manipulate data with the privileges and permissions associated with that user, ultimately reading "flag4."

![Flag 4](/assets/images/image21.png)

## Challenge 5: Privilege Escalation to Root User

The last challenge involves escalating privileges from the "vero" user to the "root" user. To achieve this, an SSH connection with the "vero" user must be established to obtain a more complete and robust shell. To do this, you need to download the "id_rsa" key from the "/home/vero/.ssh" directory, which allows authentication on the server.
Once connected with the "vero" user, it can be observed that this user is part of the "docker" group. This is important for privilege escalation, as the "docker" group has certain privileges and special capabilities on the system. It can be seen that an Alpine image is already downloaded.

![Flag 5](/assets/images/image22.png)

Taking advantage of membership in the "docker" group, insecure configurations can be exploited to obtain additional access or execute commands with elevated privileges. To do this, the command shown in the following image is executed, mounting the Alpine image in the "/mnt" directory.

![Flag 5](/assets/images/image23.png)

## Challenge 6: Forensic Analysis of a Brute Force Attack

In this challenge, a simulated brute force attack is directed at the "/login" endpoint. A .pcap file containing the captured packets during this attack is provided. From this file, a series of questions are raised with the aim of having participants investigate and discover what exactly happened.
### Username
The first question focuses on identifying the username being targeted during the brute force attack. Forensic analysis of the captured packets will reveal this crucial information. To solve it using WireShark [17], open the file and filter for HTTP packets, within which you can see the parameters being sent.

![Flag 6](/assets/images/image24.png)

### Password
The next question is to find out what the password is for the user being targeted in the brute force attack. By forensically analyzing the captured packets, you should filter for HTTP packets, and you need to examine the packet that corresponds to the successful login.

![Flag 6](/assets/images/image25.png)

### Profile Picture
The last question of this challenge is to discover which animal is shown in the profile picture of the target user. To do this, you must filter the captured packets based on their size to identify the packet corresponding to the profile picture upload. Once located, you should extract the image file from the packet to view it and determine which animal appears in the photo.

![Flag 6](/assets/images/image26.png)

## Challenge 7: Forensic Analysis of a Connection

In this test, participants will analyze the packets sent during a reverse shell to identify the file that the attacker downloads on the compromised system. The forensic analysis of the captured packets will provide clues about the actions taken by the attacker, allowing a better understanding of the scope and nature of the attack.
Participants must carefully examine the content of the packets and look for any evidence revealing the name of the file downloaded by the attacker during the reverse shell. The specific file is "linpeas.sh," which is a security audit script for Linux systems.

![Flag 7](/assets/images/image27.png)

### Downloaded File
In this test, the goal is to determine what actions the attacker is taking. Specifically, you want to identify the permissions it is looking for, known as SUID permissions. These special permissions allow a script to be executed with superuser (root) privileges.
SUID permissions are attributes that can be assigned to files in Unix/Linux systems. When a file with these permissions is executed, it inherits the privileges of the file's owner, rather than those of the user running it. This means that a file with these permissions can be executed by a normal user and yet perform actions with elevated privileges. In this context, the attacker is looking for files with specific SUID permissions to exploit them in malicious activities.

![Flag 7](/assets/images/image28.png)

### Privilege Escalation
In this final test, the goal is to determine how the attacker managed to obtain superuser (root) privileges. Specifically, you need to identify the command used to escalate privileges, which is ```find . -exec /bin/sh ; -quit ```

The attacker takes advantage of the SUID permissions of the "find" command to execute a shell as an administrator. SUID permissions on the "find" executable allow the user executing it to temporarily gain elevated privileges. This way, the attacker can use this command in combination with other parameters to execute a shell with superuser privileges.

![Flag 7](/assets/images/image29.png)

If you've made it this far, you're quite brave :). Thank you very much, and I hope you enjoyed it.


