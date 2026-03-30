---
title: "Hack The Box — Starting Point: Redeemer Writeup"
date: 2026-03-30 00:00:00 +0800
categories: [CTF, HTB-StartingPoint]
tags: [htb, starting-point, nmap, openvpn, database, redis]
image:
  path: "/assets/img/img-000.png"
  alt: Hack The Box Redeemer

---

# Hack The Box - Redeemer

![Redeemer Solved](/assets/img/img-000.png)

---

## Introduction

Databases are a collection of organized information that can be easily accessed, managed and updated. There are different types of databases and one among them is **Redis**, which is an **'in-memory' database**. In-memory databases are the ones that rely essentially on the primary memory for data storage (meaning that the database is managed in the RAM of the system).

This lab focuses on **enumerating a Redis server remotely** and then **dumping its database** in order to retrieve the flag. In this process, we learn about the usage of the `redis-cli` command line utility which helps interact with the Redis service.

---

## Questions

| # | Question | Answer |
|---|----------|--------|
| 1 | Which TCP port is open on the machine? | `6379` |
| 2 | Which service is running on the port that is open on the machine? | `redis` |
| 3 | What type of database is Redis? | `In-memory Database` |
| 4 | Which command-line utility is used to interact with the Redis server? | `redis-cli` |
| 5 | Which flag is used with the Redis command-line utility to specify the hostname? | `-h` |
| 6 | Once connected to a Redis server, which command is used to obtain the information and statistics about the Redis server? | `info` |
| 7 | What is the version of the Redis server being used on the target machine? | `5.0.7` |
| 8 | Which command is used to select the desired database in Redis? | `select` |
| 9 | How many keys are present inside the database with index 0? | `4` |
| 10 | Which command is used to obtain all the keys in a database? | `keys *` |

---

## What is Redis?

*Redis* (REmote DIctionary Server) is an open-source advanced NoSQL key-value data store used as a database, cache, and message broker. The data is stored in a dictionary format having key-value pairs. It is typically used for **short term storage** of data that needs fast retrieval. Redis does backup data to hard drives to provide consistency.

---

## Enumerate

To verify the connectivity and availability of the target, we run the `ping` command with the IP address. If we get replies, the target is alive.

![Ping the target](/assets/img/img-001.png)

Next, we run a full port scan using `nmap` with service version detection:

```bash
nmap -p- --min-rate 5000 -sV <target_ip>
```

| Flag | Description |
|------|-------------|
| `-p-` | Scan all 65,535 ports |
| `--min-rate 5000` | Send at least 5000 packets/sec (faster scan) |
| `-sV` | Detect service name and version on open ports |

![Nmap scan result](/assets/img/img-002.png)

The scan reveals **port 6379** is open, running **Redis 5.0.7**.

---

## Connect to Redis

Install the Redis CLI tool if not already installed:

```bash
sudo apt install redis-tools
```

Then connect to the target:

```bash
redis-cli -h <target_ip>
```

Once connected, run `info` to get server details:

```bash
10.129.136.187:6379> info
```

![Redis info output](/assets/img/img-003.png)

---

## Enumerate the Database

Scroll down to the **Keyspace** section of the `info` output. We can see `db0` exists with 4 keys.

Select database 0 and list all keys:

```bash
10.129.136.187:6379> select 0
10.129.136.187:6379> keys *
```

![Keys in database](/assets/img/img-004.png)

We can see **4 keys** in the database:
- `numb`
- `flag`
- `temp`
- `stor`

---

## Get the Flag

Use the `get` command to retrieve the value of the `flag` key:

```bash
10.129.136.187:6379> get flag
```

![Getting the flag](/assets/img/img-005.png)

🎉 **Flag: `03e1d2b376c37ab3f5319922053953eb`**

---

> **Summary:** Redis running without authentication exposed all stored keys and values, including the flag. Always secure Redis instances with authentication and firewall rules in production!
