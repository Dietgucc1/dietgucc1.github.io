---
title: "0day Academy CTF: WiFi Edition - WifiLoverZzz Writeup"
date: 2026-03-10 00:00:00 +0800
categories: [CTF, Wireless]
tags: [ctf, wireless, wifi, wpa3, wep, wireshark, pcap]
image:
  path: /assets/img/WhatsApp Image 2026-03-10 at 11.07.31 AM.jpeg
  alt: 0day Wireless Penetration Tester
description: Writeup for 0day Academy CTF WiFi Edition by team WifiLoverZzz — covering recon, hidden SSID, WPA3-SAE exploitation, and WEP cracking.
---

0day Academy CTF - WiFi Edition was a wireless security challenge featuring recon analysis, hidden SSID discovery, WPA3-SAE exploitation, and WEP cracking. Our team **WifiLoverZzz** tackled 10 challenges across three difficulty tiers.

## Group Details

| Role | Name | CTF Username |
|------|------|--------------|
| Team Leader | Muhammad Harith Muanis bin Masrur | AhmadBatu |
| Member | Musyrif Thaqif | Keeps |
| Member | Nur Adlina Auji  | Dietgucci |
| Member | Nur Alia Izzati | yoss |

---

## Part 1: Easy Peasy Lemon Squeezy

### Challenge 1: I See You

**Description:** From the recon output, identify the BSSID for SSID `CampusLab`.

**Flag format:** `waifu{bssid=MAC}` — Given file: `recon_capture.txt`

**Analysis:**

Looking at `recon_capture.txt`, I located the entry for **CampusLab**. The BSSID is listed in the first column.

**Flag:** `waifu{bssid=A4:5E:60:1B:9C:10}`

![I See You - recon output](/assets/img/Iseeu.jpg)
![I See You - BSSID entry](/assets/img/Iseeuu.jpg)

---

### Challenge 2: I See You 2

**Description:** From the recon snapshot, identify the channel for SSID `LibraryNet_01`.

**Flag format:** `waifu{easy01_channel=CHANNEL}` — Given file: `EASY_01_channel.txt`

**Analysis:**

Using the `EASY_01_channel.txt` file, I scanned the metadata fields. The `Channel` field explicitly listed the value as **36**.

**Flag:** `waifu{easy01_channel=36}`

![I See You 2 - channel result](/assets/img/iseeu2.jpg)

---

### Challenge 3: I See You 3

**Description:** From the recon snapshot, identify the cipher used for SSID `HallWiFi_02`.

**Flag format:** `waifu{easy02_cipher=CIPHER}` — Given file: `EASY_02_cipher.txt`

**Analysis:**

I inspected the `EASY_02_cipher.txt` file. Under the `Cipher` header, the protocol was identified as **CCMP** (Counter Mode Cipher Block Chaining Message Authentication Code Protocol). This means the cipher used for SSID `HallWiFi_02` is CCMP.

**Flag:** `waifu{easy02_cipher=CCMP}`

![I See You 3 - cipher result](/assets/img/iseeu3.jpg)

---

### Challenge 4: I See You 4

**Description:** From the recon snapshot, identify the security mode for SSID `LibraryNet_03`.

**Flag format:** `waifu{easy03_security=MODE}` — Given file: `EASY_03_security.txt`

**Analysis:**

In `EASY_03_security.txt`, I looked at the `Security` field. The network is configured with **WPA2-PSK** (Pre-Shared Key).

**Flag:** `waifu{easy03_security=WPA2-PSK}`

![I See You 4 - security mode](/assets/img/iseeu4.jpg)

---

### Challenge 5: Big Mac

**Description:** What's the MAC address of the AP?

**Given file:** `EASY_04_bssid.txt`

**Analysis:**

Analyzing the `EASY_04_bssid.txt` file, I extracted the hardware address from the BSSID field.

**Flag:** `waifu{DC:A6:32:34:2F:C2}`

![Big Mac - MAC address](/assets/img/bigmac.jpg)

---

### Challenge 6: Hidden in Plain Sight

**Description:** You've performed a site survey and captured a `.pcap` file. One network is "Hidden," but a client just connected to it, revealing its name in the probe response.

**Given file:** `hidden.pcap`

**Analysis:**

By inspecting the Tagged Parameters within the second packet of the capture — specifically the **802.11 Probe Response** — we can see the exact moment the router revealed its name to the client.

Although the Access Point (BSSID: `00:11:22:33:44:55`) was not broadcasting its identity in its Beacon Frames, we were able to "listen in" on the management exchange and extract the hidden SSID **Sakura_Garden** in plain text.

**Flag:** `waifu{Sakura_Garden}`

![Hidden in Plain Sight - probe response](/assets/img/hips.png)
![Hidden in Plain Sight - Wireshark capture](/assets/img/hipss.png)

---

## Part 2: Insane in the Membrane!!!

### Challenge 7: Dragon's Egg

**Description:** A modern WPA3-SAE (Dragonfly) handshake was captured from a high-security facility. The client appears to be using a modified driver to exfiltrate data during the initial key exchange. Stop trying to crack the password — look at the math being sent over the air.

**Given file:** `dragons_egg.pcap`

#### Method 1

We inspected the packets in the capture, then double-clicked the malformed packet to see what had been sent. The flag was embedded directly inside the packet data.

**Flag:** `waifu{sae_scalar_leak}`

![Dragon's Egg - malformed packet](/assets/img/dragonsegg.png)
![Dragon's Egg - flag found](/assets/img/dragonssegg.jpg)

#### Method 2

- Looked at the handshake data. Instead of trying to crack passwords (which the challenge said was pointless), I examined the actual numbers being sent.
- **Spotted the anomaly:** In normal WPA3, the "scalar" should look like a big random number. But in this capture, part of it was readable ASCII text.
- **Found the message:** Right there in the data, where the random number should be, was the flag hidden as a readable string.

**Flag:** `waifu{sae_scalar_leak}`

---

### Challenge 8: Dragon's Heart

**Description:** The facility has upgraded to WPA3, and brute-force tools are useless against the Dragonfly exchange. A captured memo suggests the network architect was "leaving himself a backdoor in the math." The SAE Commit contains more than just cryptographic noise. Find the key, solve the scalar, and recover the secret.

**Given file:** `dragonheart.pcap`

#### Method 1

The SSID name is the key for an XOR (`0x42`). We take the hex dump of the SAE Commit and decode it using CyberChef with the XOR key.

**Flag:** `waifu{SAE_DRAGON_SAY_HAY}`

![Dragon's Heart - CyberChef XOR decode](/assets/img/dragonsheart.png)
![Dragon's Heart - result](/assets/img/dragonsheart.jpg)

#### Method 2

I started by looking at the network name (SSID): **`Hidden_Key_0x42`**. In CTFs, when you see a hex value like `0x42` in a name, it's almost always a hint for an **XOR key**.

Since the challenge mentioned a "backdoor in the math," I went straight to the **SAE Commit** frames in the pcap. Usually these frames are full of random cryptographic noise, but I spotted the ASCII string `5#+$79` — not normal for WPA3. I knew the flag was hidden there.

![Dragon's Heart Method 2 - SAE Commit anomaly](/assets/img/dragonsheart method2.png)

**Step 1: Carve the raw data**

```bash
grep -obUa "5#+\$79" dragonheart.pcap | cut -d: -f1 | xargs -I % dd \
  if=dragonheart.pcap bs=1 skip=% count=32 2>/dev/null | xxd -p
```

- `grep -ob` — gives you the exact byte offset of the string
- `dd` — jumps to that byte offset and grabs the next 32 bytes
- `xxd -p` — outputs a clean hex string to copy

**Step 2: XOR decode the hex**

```python
python3 -c "
h='35232b2437391103071d061003050d0c1d11031b1d0a031b3f'
print(''.join(chr(int(h[i:i+2],16)^0x42) for i in range(0,len(h),2)))
"
```

The logic: take each byte of the hex string, convert to an integer, XOR it with `0x42`, and convert back to a character. The result reveals the flag.

**Conclusion:**

The "backdoor" was the architect being lazy — instead of doing real WPA3 math, he XORed the flag with `0x42` and tucked it into the Scalar field of the SAE handshake.

**Flag:** `waifu{SAE_DRAGON_SAY_HAY}`

---

## Part 3: Now We're Cooking

### Challenge 9: Target Lock In Sight

**Description:** From the raw events, determine which client STA is targeted most frequently by DEAUTH for the primary SSID and legitimate BSSID.

**Given file:** `events.csv`

**Analysis:**

**Flag:**

![Target Lock - flag output](/assets/img/tlis.png)

What I did:

![Target Lock - full command pipeline](/assets/img/tliss.png)

Okay first of all, I created the csv file which is `wifi_data.csv`:

![Target Lock - csv file created](/assets/img/kali tlis.png)

Then I checked if the file looks right with a simple header command `head -5 wifi_data.csv` — so it shows the header and first few lines:

![Target Lock - head command output](/assets/img/kali tlis2.png)

Then I typed the counting command to show the output:

![Target Lock - counting command output](/assets/img/kali tlis3.png)

I built a command pipeline to find the most targeted device:

```bash
grep "deauth" wifi_data.csv \
  | grep "TrainingAP_201" \
  | grep "DC:A6:32:23:11:39" \
  | awk -F',' '{print $5}' \
  | sort | uniq -c | sort -rn
```

Breaking down each step:

| Step | Command | What it does |
|------|---------|--------------|
| 1 | `grep "deauth"` | Filter only deauthentication attack lines |
| 2 | `grep "TrainingAP_201"` | Keep only the primary SSID |
| 3 | `grep "DC:A6:32:23:11:39"` | Filter for the legitimate BSSID only |
| 4 | `awk -F',' '{print $5}'` | Extract column 5 — the destination MAC address |
| 5 | `sort \| uniq -c \| sort -rn` | Count occurrences and rank highest first |

The CSV column structure:

```
1:time | 2:ssid | 3:ap_bssid | 4:src | 5:dst | 6:subtype | 7:reason_code
```

**Flag:** `waifu{5C:AA:FD:05:B7:FA}`

---

### Challenge 10: Counting from 1234567890

**Description:** A legacy inventory scanner at a shipping warehouse is still running WEP. The supervisor uses a simple, sequential factory-default key. Crack the encryption and find the flag inside the packet.

**Given file:** `WEPisNoob.pcap`

**Analysis:**

We used `airdecap-ng` to decrypt the capture. Since WEP is trivially broken with a known sequential key, the traffic was revealed in plaintext, exposing the flag.

```bash
airdecap-ng -w 1234567890 WEPisNoob.pcap
```

Once decrypted, the underlying data was readable and the flag was visible in plain text.

**Flag:** `waifu{WEP_is_dead_long_live_WPA}`

![WEP - airdecap-ng output](/assets/img/1234567.png)
![WEP - decrypted traffic](/assets/img/12345678.jpg)

---

*Writeup by team WifiLoverZzz — 0day Academy CTF, WiFi Edition.*
