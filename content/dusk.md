Title: Walkthrough: Dusk CTF
Date: 2020-02-10
Tags: #CTF
Category: CTF
Slug: dusk
Authors: nothing0x00
Summary: Walkthrough for Dusk CTF

This is the first in a series of CTF walkthroughs. The goal of this series is to demonstrate the pathway from boot to root, and to do so with an eye toward clarity and an incremental stepping through of the process used to solve the machine.  More of these walkthroughs will be posted as they become relevant, and as I have the time to complete machines.

**Note: At no point will there be information related to current, non-retired, machines on HackTheBox, or any walkthroughs of the OSCP labs, nor will I respond to inquiries about how to complete these machines**

So, with that, let's begin...

When the machine boots it is allocated an IP of 192.68.56.108 (running on a Host-Only network in VirtualBox).

Initial nmap scanning shows that ports 21, 22, 25, 80, 3306 and 8080 are open.

![nmap_image](/images/dusk/nmap.png)

More detailed scanning did not show any obvious pathways of attack, with most services reporting back as expected. After a significant amount of enumeration the best pathway forward seemed to be to brute force a way into the machine.

Attempts were made to brute force the FTP service, with no luck. Eventually I succeeded in grabbing valid credentials for the MariaDb service on port 3306, using the mysql-brute NSE script within nmap.

![mysql_brute_force](/images/dusk/mysql_brute.png)

Using these credentials it was possible to log into the database remotely.

![mysql_login](/images/dusk/mysql_login.png)

The database itself did not return any useful information, but because the service on port 8080 is running PHP, and because it is possible to write to the filesystem as root in SQL databases, it is theoretically possible to write a webshell to the system and access it over port 8080.

The service on port 8080 is utilizing /var/tmp as a webroot, as indicated by the information displayed by the service.

![webroot](/images/dusk/8080_working_dir.png)

Using the information gathered about the service it was possible to utilize the database access to write a PHP webshell to the webserver using the following command:

![webshell_write](/images/dusk/shell_write.png)

The shell was then accessed at http://192.168.56.108:8080/shell2.php with commands run through the cmd parameter.

![webshell](/images/dusk/webshell.png)

Using netcat a command was constructed which generated a reverse shell from the target to the attacking machine.

![webshell_reverse](/images/dusk/webshell_reverse.png)

---
![webshell_handler](/images/dusk/webshell_reverse_handler.png)

After initial access was gained it was possible to grab the user flag.

![user_flag](/images/dusk/user.png)

Next, I moved to /tmp, pulled down LinEnum and started enumeration.

The initial run showed that the www-data user was able to run ping, make and sl with sudo privileges without providing a password.

![sudo_privs](/images/dusk/sudo_privs.png)

Additionally, the process tree dump contained the credentials used for the FTP server, which for pyftplib are contained in the launch command.

![ftp_creds](/images/dusk/pyftp_creds.png)

Using these credentials I was able to log into the FTP service, which granted me read access to the /root directory, where I was able to grab the flag.

![root_ftp](/images/dusk/root_ftp.png)

---
![r00ted](/images/dusk/root_flag.png)
