## Challenge Name: Hack The Box - Keepers

Category: Linux

Points: 20

Solves: 2

Challenge Description:

Request Tracker website, which contains a KeePass password manager.

Artifact Files:

\* KeePassDumpFull.dmp

\* passcodes.kdbx

### Approach

Began by doing nmap to enumerate for open ports and services.

![](RackMultipart20230828-1-nvm838_html_6e170c34dd23670e.png)

We can see that there are port 22 for SSH and port 80, indicating that there is a website.

Upon browsing [http://MACHINE\_IP](http://MACHINE_IP/), we can see that the page is showing a link redirecting to [http://tickets.keeper.htb/rt](http://tickets.keeper.htb/rt). At first I tried to click on the link directly, but wasn't able to resolve the host. After which I had google abit to find out that I had to tie the url in my hosts file (/etc/hosts).

![](RackMultipart20230828-1-nvm838_html_6456103f0ace044d.png)

After tying the hosts file, I was able to access to the request tracker page.

![](RackMultipart20230828-1-nvm838_html_280a748d1f630b06.png)

I tried to do ffuf, gobuster, dirb on the web application, but I was not able to find for any web directory or hidden directory.

Next, I tried to run through a few defaults password on admin/root account to test if I was able to access. However, I wasn't getting any results from a few tries.

Following the same idea, I used Burp Suite to configure a mini intruding session on the web application using the /usr/share/wordlists/fasttrack.txt on both admin and root username.

But before I could continue with the brute force attack, I had to first configure for the CSRF protection, in the Burp Suite settings.

![](RackMultipart20230828-1-nvm838_html_b7bfa3abea534a0.png)

Next up, I configured for Clusterbomb attack type, on the web application using the following payload list.

![](RackMultipart20230828-1-nvm838_html_d1c57692a4c9db2c.png)

![](RackMultipart20230828-1-nvm838_html_3631c1e7349f7cdf.png)

/usr/share/wordlists/fasttrack.txt

Using the following settings, I was able to find that the password for Root is "password"

![](RackMultipart20230828-1-nvm838_html_36164e13667a3653.png)

Finally, after 10-20minutes of fiddling, I was able to gain access to the website

![](RackMultipart20230828-1-nvm838_html_5cbfd7106756060d.png)

Now I would have to find some form of exploits/vulnerabilities or information within the request tracker.

I first head to Exploit DB and google, but I wasn't able to find any form of exploits or Code Execution to get a backdoor access.

Next I browse through the request tracker catalogues, I managed to stumbled upon the "Users" section, which leads me to another user called "lnorgaard".

![](RackMultipart20230828-1-nvm838_html_e59484264a58fa98.png)

When I click into the user, I can see that there are password written in the comment section for the following user.

![](RackMultipart20230828-1-nvm838_html_971ff142aad456aa.png)

With the username and password, I tried to access the Keeper Server using SSH. I was able to connect to SSH and get access to the user.txt flag.

\*\*1. Get User Access /flag1\*\*

![](RackMultipart20230828-1-nvm838_html_1d00c5ee9592eaa7.png)

\*\*2 Get Root Access /root.txt\*\*

Following that, I checked for the SUID, Capabilities, Crontab, and Sudo -l settings. But all did not result in any misconfiguration or exploits that were available.

Next, I uploaded a linpeas shell script to run and check for any linux privilege escalation, however I did not find any useful information.

From here, I was abit stumbled, I thought of web config password, as well as iterate through user folder to see what was available.

Because this machine was a shared machine, I was not sure if the KeePass files was from. So firstly I unzipped the RT30000.zip file. Inside this RT30000.zip is actually the file containing KeePassDumpFull.dmp and passcodes.kdbx. While I wasn't sure what the file is for, I continued to look for other information. In the /var/mail/lnorgaard, I found a email which describes of a ticket regarding the KeePass files and the detail.

From this, I began to find my way to debug and analyze the files. While doing research on KeePass, there wasn't any direct exploit on the KeePass database file or application. So I installed KeePass on my Kali Linux to open the database. However I was prompted with master password.

![](RackMultipart20230828-1-nvm838_html_bdf29cd85ec6794d.png)

At first, I tried to use the previous password, but I was not able to access the password database. I continued to research on how to debug for the .dmp file that was generated along with the database file. I was able to find out that there is apparently ways to extract the master password from the .dmp file.

I found a github repository that is able to extract the master password from the .dmp file called KeePass Password Dumper(https://github.com/CMEPW/keepass-dump-masterkey)

![](RackMultipart20230828-1-nvm838_html_9553f26fdeee5351.png) I got the results of some weird character, which from the username, I was able to determine that the user and password is of Danish origin. I google for the possible password, it came out as "Rødgrød Med Fløde" which is a Danish red berry pudding name.

![](RackMultipart20230828-1-nvm838_html_b3b09e9d7d3f0169.png)

Having the possible password, I copied pasted it into the passcode, and it works.

![](RackMultipart20230828-1-nvm838_html_392b0d925a7f4f2b.png)

In the KeePass Manager, its revealed that the pass for Root is "F4\>\<3K0nd!". However I tried to do su – root and ssh login to root, however the password does not seem to work.

I then google abit to find that its possible to convert the following into. ppk rsa key file. The format was known as a PuTTY Key Format as detailed in the following site.

[https://www.baeldung.com/linux/ssh-key-types-convert-ppk - 4.2](https://www.baeldung.com/linux/ssh-key-types-convert-ppk%20-%204.2) PuTTY Key Format.

So I saved the line and convert the .ppk file, using puttygen to convert ppk to pem as detailed in this stackoverflow thread [https://askubuntu.com/questions/818929/login-ssh-with-ppk-file-on-ubuntu-terminal](https://askubuntu.com/questions/818929/login-ssh-with-ppk-file-on-ubuntu-terminal)

![](RackMultipart20230828-1-nvm838_html_e81a0c95a6580b2a.png)

Once done, I ran the ssh with the pem key, and I was connected as root to machine. Now I just had to go and retrieve the root.txt

![](RackMultipart20230828-1-nvm838_html_a0a5835718cd641e.png)

### Reflections

I need to improve on my initial recon steps. I was already somewhat stucked when I couldn't access tickets.keeper.thb/rt.

Secondly, I was also stucked at the KeePass section, especially the .dmp file, as I was trying to find out how to open and read the .dmp file. I should google .dmp file together with the KeePass keyword to get the correct results.

After getting the password and accessing the KeePass, the rest was easily solvable.