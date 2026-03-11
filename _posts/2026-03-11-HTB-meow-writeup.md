---
title: "Hack The Box — Starting Point: Meow Writeup"
date: 2026-03-11 00:00:00 +0800
categories: [CTF, HTB-StartingPoint]
tags: [htb, starting-point, telnet, nmap, openvpn, linux]
image:
  path: /assets/img/meow-badge.jpeg
  alt: Hack The Box Meow
description: Writeup for Hack The Box Starting Point — Meow. Covers VPN setup, Nmap scanning, and exploiting a misconfigured Telnet service.
---

**Platform:** Hack The Box  
**Category:** Starting Point  
**Difficulty:** Very Easy  
**OS:** Linux  

---

---

## Introduction

Meow is the first machine in Hack The Box's Starting Point series. It is designed for complete beginners and introduces fundamental concepts such as VPN connectivity, network scanning, and exploiting a misconfigured Telnet service. The goal is to gain access to the target machine and retrieve the flag.

---

## Tools Used

- **OpenVPN** — to connect to the HTB network
- **ping** — to verify connectivity to the target
- **Nmap** — to scan for open ports and services
- **Telnet** — to connect to the target machine

---

## Step 1 — Connect to the HTB VPN

Before anything else, need to connect to the HTB VPN. Without this, our machine cannot communicate with the target.

1. Log in to [app.hackthebox.com](https://app.hackthebox.com)
2. Go to **Starting Point** → **Connect to HTB** → **OpenVPN**
3. Select your region and download the `.ovpn` file
![Download VPN](/assets/img/meow-2.jpeg)
4. Open a terminal and run:

![Meow Banner](/assets/img/meow-1.jpeg)

```bash
sudo openvpn ~/Downloads/starting_points_eu-starting-point-2-dhcp.ovpn
```
Make sure your dir file is correct

5. Wait until you see the following line — this confirms you are connected:

```
Initialization Sequence Completed
```

![VPN Connected](/assets/img/meow-8.jpeg)

> **Important:** Keep this terminal open the entire time. Open a new terminal tab for all other commands.

---

## Step 2 — Spawn the Machine and Verify Connectivity

1. On the HTB website, go to the **Meow** machine page and click **Spawn Machine**
2. Wait for the **Target IP Address** to appear (e.g. `10.129.11.129`) ps:in my picture given I have already respawned the IP address for the screenshot purpose
![VPN Connected](/assets/img/meow-3.jpeg)
3. In a **new terminal**, verify you can reach the target using ping:

```bash
ping -c 4 10.129.11.129
```

![Ping Target](/assets/img/meow-4.jpeg)

If you receive replies with no packet loss, your connection to the target is stable and you are ready to proceed.

---

## Step 3 — Scan for Open Ports with Nmap

Now we use **Nmap** (Network Mapper) to discover which ports are open and what services are running on the target.

```bash
nmap -sV -Pn -T4 10.129.11.129
```

**Flags explained:**
- `-sV` — detects the version of services running on open ports
- `-Pn` — skips the ping check and scans even if the host appears down
- `-T4` — uses aggressive timing for a faster scan (haha if I dont use this, it took a bit longer to wait for the scan)

![Nmap Scan](/assets/img/meow-5.jpeg)

**Output:**

```
PORT   STATE SERVICE VERSION
23/tcp open  telnet  Linux telnetd
```

Port **23** is open and running **Telnet** — a protocol used for remote management of hosts over a network. Telnet is considered insecure because it transmits data, including credentials, in plain text.

---

## Step 4 — Connect via Telnet

Since Telnet is running on the target, I can attempt to connect to it:

```bash
telnet 10.129.11.129
```

I have been greeted with a login prompt. Many misconfigured systems leave default credentials in place. I try logging in as `root` with no password:

```
Meow login: root
Password: (press Enter — leave blank)
```

This works! I logged in as the **root** user, which means I have full administrative access to the machine.

![Telnet Login](/assets/img/meow-6.jpeg)

---

## Step 5 — Retrieve the Flag

Now that I have access, I simply read the flag file:

```bash
cat flag.txt
```

![Flag](/assets/img/meow-7.jpeg)

Yeay! found the flag! 

---

## Summary

| Step | Action | Tool |
|------|--------|------|
| 1 | Connect to HTB network | OpenVPN |
| 2 | Verify connectivity to target | ping |
| 3 | Discover open ports and services | Nmap |
| 4 | Connect to Telnet service as root | Telnet |
| 5 | Read the flag | cat |

---

## Key Takeaways

- Always connect to the VPN before attempting to reach HTB machines
- Use `-Pn` with Nmap if the host appears to be down
- **Telnet is insecure** — it sends all data in plain text and should never be used in production environments
- Default or blank credentials are a critical misconfiguration that attackers can easily exploit
- If the machine stops responding, simply reset/respawn it on HTB and use the new IP

---

*Written by DietGucci | Hack The Box Starting Point Series*
