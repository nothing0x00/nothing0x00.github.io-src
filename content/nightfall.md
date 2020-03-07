Title: Walkthrough: Nightfall CTF
Date: 2020-02-17
Tags: #CTF
Category: CTF
Slug: nightfall
Authors: nothing0x00
Summary: Walkthrough for Nightfall CTF

Nightfall is the second machine in a series of three (Dusk, Nightfall, Sunrise) that we will be walking through here.

So, without further adieu, let's begin...

The initial enumeration of the system shows that a number of ports are open, including ports 21, 22, 80, 139, 445 and 3306.

![initial_nmap](/images/nightfall/initial_nmap.png)

The initial foothold was difficult to identify, with no attack paths being made apparent through additional scanning, attempts to find pages using gobuster and attempting to brute force the SQL service.

The first piece of useful information came from running enum4linux on the SMB and NBT services. This yielded not only information on the workgroups and domains which are related to the machine, but it also netted some usernames.

![workgroups](/images/nightfall/workgroups.png)

![users](/images/nightfall/users.png)

After more enumeration it was clear that no technical forms of exploitation were going to emerge as likely, so the strategy moved into less technical forms of exploitation, specifically brute force attacks. After attempting attacks against FTP, SSH and MYSQL using the fasttrack list, I then moved to the rockyou list and got some valid credentials for the FTP service.

![brute_force_ftp](/images/nightfall/brute_ftp.png)

The FTP server did not contain any usable data, but the directory was the /home/matt directory. Users with valid credentials for the matt account also have read/write access to the directory So, to gain access to the machine I first created a .ssh directory, then wrote an authorized_keys file locally and transfered it to the remote server over FTP.

![ftp_auth_keys](/images/nightfall/ftp_put_authorized_keys.png)

With the authorized_keys file in place it was possible to use the SSH keys stored on the attacking machine to log into the machine as matt.

![ssh_login_matt](/images/nightfall/ssh_login_matt.png)

After landing on the machine the first step, after seeing if I can get into other home folders and access files (I cannot), was to pull down LinEnum and enumerate the machine. The initial results show an interesting SUID and SGID file, owned and editable by nightfall.

![suid](/images/nightfall/suid.png)

The /scripts/find file runs as nightfall. If we assume that this is the same as the find binary, then by extension the find command is capable of executing commands through the -exec flag. After a test we can confirm that the functionality of this /scripts/find binary functions similarly to the find command. Using this information it was possible to execute /bin/bash in the context of the nightfall user to achieve lateral privileges to the nightfall user.

![suid_exploitation](/images/nightfall/nightfall_suid_exploitation.png)

After achieving elevated access the first step was to grab the user.txt flag.

![user_flag](/images/nightfall/user_flag.png)

Then, a similar process of establishing SSh access was replicated; a .ssh directory was created in /home/nightfall, then the public ssh key for my attacking machine was echoed into an authorized_keys file.

![nightfall_ssh](/images/nightfall/nightfall_ssh.png)

![nightfall_ssh_access](/images/nightfall/nightfall_ssh_access.png)

Initial enumeration shows that we can run the cat command with root privileges through sudoing without a password.

![nightfall_sudo_l](/images/nightfall/nightfall_sudo_l.png)

Using this access I read out the /etc/shadow file to the screen.

![shadow](/images/nightfall/shadow.png)

Then I copied the root entry to a file called shadow on the attacking machine, and the root entry in the /etc/passwd file to a passwd file on the attacking machine. Using the unshadow command from the john password cracking suite I combined these into a crackable password hash and started john. Using this process I was able to access the root password; miguel2.

![john_process](/images/nightfall/john_process.png)

After extracting the password I used it to su to root, then grabbed the flag.

![r00ted](/images/nightfall/r00ted.png)
