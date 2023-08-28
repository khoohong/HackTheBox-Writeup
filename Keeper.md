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

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/92353e24-f853-4840-af64-819d6b1df48a)

We can see that there are port 22 for SSH and port 80, indicating that there is a website.

Upon browsing [http://MACHINE\_IP](http://MACHINE_IP/), we can see that the page is showing a link redirecting to [http://tickets.keeper.htb/rt](http://tickets.keeper.htb/rt). At first I tried to click on the link directly, but wasn't able to resolve the host. After which I had google abit to find out that I had to tie the url in my hosts file (/etc/hosts).

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/2adb1b97-a915-48f0-9d84-672ef331eddf)

After tying the hosts file, I was able to access to the request tracker page.

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/c5296f27-9f2d-48b8-8cba-7e7f47069ec9)

I tried to do ffuf, gobuster, dirb on the web application, but I was not able to find for any web directory or hidden directory.

Next, I tried to run through a few defaults password on admin/root account to test if I was able to access. However, I wasn't getting any results from a few tries.

Following the same idea, I used Burp Suite to configure a mini intruding session on the web application using the /usr/share/wordlists/fasttrack.txt on both admin and root username.

But before I could continue with the brute force attack, I had to first configure for the CSRF protection, in the Burp Suite settings.

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/a8e7dbfc-0926-452e-8804-8c8241033188)


Next up, I configured for Clusterbomb attack type, on the web application using the following payload list.

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/a8c2bbc2-d888-4a4a-af7e-7a7328998385)

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/1ad1b9a3-5529-4ab5-88e1-8722efdbaa64)

/usr/share/wordlists/fasttrack.txt

Using the following settings, I was able to find that the password for Root is "password"

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/12d53dda-9030-43db-83cb-89693dc86290)

Finally, after 10-20minutes of fiddling, I was able to gain access to the website

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/ae28a0b8-b7f6-47bc-8873-00190fb09bf6)


Now I would have to find some form of exploits/vulnerabilities or information within the request tracker.

I first head to Exploit DB and google, but I wasn't able to find any form of exploits or Code Execution to get a backdoor access.

Next I browse through the request tracker catalogues, I managed to stumbled upon the "Users" section, which leads me to another user called "lnorgaard".

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/9c338e3b-9559-474d-8e32-d5e1499244e4)


When I click into the user, I can see that there are password written in the comment section for the following user.

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/eaabb6b5-795f-4f19-a2c6-825312a680e7)


With the username and password, I tried to access the Keeper Server using SSH. I was able to connect to SSH and get access to the user.txt flag.

\*\*1. Get User Access /flag1\*\*

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/c106bcfe-fa88-439c-b339-76558aec894e)

\*\*2 Get Root Access /root.txt\*\*

Following that, I checked for the SUID, Capabilities, Crontab, and Sudo -l settings. But all did not result in any misconfiguration or exploits that were available.

Next, I uploaded a linpeas shell script to run and check for any linux privilege escalation, however I did not find any useful information.

From here, I was abit stumbled, I thought of web config password, as well as iterate through user folder to see what was available.

Because this machine was a shared machine, I was not sure if the KeePass files was from. So firstly I unzipped the RT30000.zip file. Inside this RT30000.zip is actually the file containing KeePassDumpFull.dmp and passcodes.kdbx. While I wasn't sure what the file is for, I continued to look for other information. In the /var/mail/lnorgaard, I found a email which describes of a ticket regarding the KeePass files and the detail.

From this, I began to find my way to debug and analyze the files. While doing research on KeePass, there wasn't any direct exploit on the KeePass database file or application. So I installed KeePass on my Kali Linux to open the database. However I was prompted with master password.

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/5753cf64-1f90-42d1-92b9-2a5e032901e1)


At first, I tried to use the previous password, but I was not able to access the password database. I continued to research on how to debug for the .dmp file that was generated along with the database file. I was able to find out that there is apparently ways to extract the master password from the .dmp file.

I found a github repository that is able to extract the master password from the .dmp file called KeePass Password Dumper(https://github.com/CMEPW/keepass-dump-masterkey)

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/98062792-1552-4f04-973f-544c745e3bce)
 I got the results of some weird character, which from the username, I was able to determine that the user and password is of Danish origin. I google for the possible password, it came out as "Rødgrød Med Fløde" which is a Danish red berry pudding name.

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/3f3437f2-d895-437c-8d27-a88ca534dcb7)


Having the possible password, I copied pasted it into the passcode, and it works.

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/d5dcaa92-6aa1-4042-a23a-50105890b7e9)


In the KeePass Manager, its revealed that the pass for Root is "F4\>\<3K0nd!". However I tried to do su – root and ssh login to root, however the password does not seem to work.

I then google abit to find that its possible to convert the following into. ppk rsa key file. The format was known as a PuTTY Key Format as detailed in the following site.

[https://www.baeldung.com/linux/ssh-key-types-convert-ppk - 4.2](https://www.baeldung.com/linux/ssh-key-types-convert-ppk%20-%204.2) PuTTY Key Format.

So I saved the line and convert the .ppk file, using puttygen to convert ppk to pem as detailed in this stackoverflow thread [https://askubuntu.com/questions/818929/login-ssh-with-ppk-file-on-ubuntu-terminal](https://askubuntu.com/questions/818929/login-ssh-with-ppk-file-on-ubuntu-terminal)

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/c121394b-0aa0-49f8-bb13-a669af87f159)


Once done, I ran the ssh with the pem key, and I was connected as root to machine. Now I just had to go and retrieve the root.txt

![image](https://github.com/khoowh1996/HackTheBox-Writeup/assets/74196098/9ca254d4-ab97-4fb4-a50a-1226c0c8901b)


### Reflections

I need to improve on my initial recon steps. I was already somewhat stucked when I couldn't access tickets.keeper.thb/rt.

Secondly, I was also stucked at the KeePass section, especially the .dmp file, as I was trying to find out how to open and read the .dmp file. I should google .dmp file together with the KeePass keyword to get the correct results.

After getting the password and accessing the KeePass, the rest was easily solvable.
