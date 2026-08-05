---
title: "Wi-Fi Hacking: A Deep Dive into Wireless Authentication Protocols"
date: 2026-08-05
categories:
  - wireless-security
tags:
  - wifi-hacking
  - wep
  - wps
  - wpa
  - wpa2
  - wpa2-enterprise
  - pentesting
header:
  teaser: /assets/images/posts/wifi-hacking-cover.png
excerpt: "A technical breakdown of wireless authentication mechanisms — WEP, WPS, WPA/WPA2-PSK, and WPA2-Enterprise — their weaknesses, and the tools used to assess them."
---

Wireless networks have evolved significantly since the early 2000s, but many of the authentication mechanisms still in use today carry design flaws inherited from their predecessors. In this post, I'll break down the main Wi-Fi authentication protocols from a technical perspective, covering how each one works, their known weaknesses, and the tools typically used during a wireless penetration test.

## Frequencies and Channels: The Physical Layer

Before diving into authentication, it's worth understanding the radio spectrum Wi-Fi operates on, since it directly affects both attack surface and tooling.

- **2.4 GHz band**: Channels 1–14 (availability varies by region), each 20 MHz wide but spaced only 5 MHz apart, causing significant channel overlap. Only channels 1, 6, and 11 are truly non-overlapping in most regulatory domains. This band has longer range but is more congested and prone to interference.
- **5 GHz band**: Offers many more non-overlapping channels (36, 40, 44, 48, up to 165 depending on region), higher throughput, and shorter range. Less congested, but some channels require DFS (Dynamic Frequency Selection) due to radar coexistence requirements.
- **6 GHz band (Wi-Fi 6E)**: Newer addition, exclusively for WPA3 and OWE — legacy protocols like WEP/WPA aren't even permitted here.

For auditing purposes, your wireless adapter needs to support **monitor mode** and **packet injection** on the relevant band. Chipsets like Atheros (AR9271) or Realtek (RTL8812AU) are common choices for 2.4/5 GHz auditing.

**Key tools for this layer:**
- `airmon-ng` — enables monitor mode on a compatible adapter
- `airodump-ng` — passive scanning across channels, identifying SSIDs, BSSIDs, channel, encryption type, and connected clients

---

## WEP (Wired Equivalent Privacy)

### How it works
WEP uses the **RC4 stream cipher** with either a 40-bit or 104-bit key, combined with a 24-bit **Initialization Vector (IV)** that's transmitted in plaintext with every packet. The IV is meant to ensure each packet is encrypted with a unique keystream, but its small size is WEP's fatal flaw.

### Why it's broken
With only 2^24 possible IVs, on a busy network they start repeating relatively quickly. Once enough packets share the same IV, statistical attacks (like the **Korek** or **PTW** attack) can recover the key without needing to crack anything by brute force — it's a mathematical weakness in the protocol itself, not a password strength issue.

### Attack methodology (conceptual)
1. Capture traffic in monitor mode while the network operates normally
2. If traffic is low, use **ARP replay** or **fragmentation attacks** to artificially generate more IVs
3. Once enough unique IVs are captured (typically 40,000–100,000+), run a statistical key-recovery attack

### Tools
| Tool | Purpose |
|---|---|
| `airodump-ng` | Capture IVs into a `.cap` file |
| `aireplay-ng` | Traffic injection (ARP replay, fake auth) to accelerate IV generation |
| `aircrack-ng` | Statistical key recovery from captured IVs |

WEP has been formally deprecated since 2004, but it's still occasionally found on legacy industrial or IoT equipment.

---

## WPS (Wi-Fi Protected Setup)

### How it works
WPS was designed for consumer convenience — letting users connect via an 8-digit PIN or a physical button, instead of typing the full Wi-Fi password. It runs as a protocol layered on top of whatever encryption the AP uses (usually WPA2-PSK).

### Why it's broken
The 8-digit PIN is validated in **two halves** by the access point: the first 4 digits and the last 4 digits (where the last digit is a checksum) are verified **independently**. This reduces the actual keyspace from 10^8 to roughly 11,000 combinations — small enough to brute-force in hours.

A more modern flaw, **Pixie Dust**, exploits weak entropy in the pseudo-random number generation used by some chipsets during the WPS handshake, allowing the PIN to be calculated **offline**, in seconds, from a single captured exchange.

### Tools
| Tool | Purpose |
|---|---|
| `wash` | Scans for APs with WPS enabled |
| `reaver` | Online brute-force of the WPS PIN |
| `bully` | Alternative WPS PIN brute-forcer, often more stable on certain chipsets |
| `pixiewps` | Offline Pixie Dust attack using data captured from the WPS handshake |

Once the WPS PIN is recovered, the underlying WPA/WPA2 passphrase is also revealed — making WPS a common weak link even when the actual Wi-Fi password is strong.

---

## WPA / WPA2 - PSK (Pre-Shared Key)

### How it works
WPA replaced WEP's flawed key scheduling with **TKIP** (Temporal Key Integrity Protocol), and WPA2 introduced **CCMP**, based on **AES**. Both operate in **Personal mode**, where all clients share the same passphrase.

The core mechanism is the **4-way handshake**, which happens every time a client connects:

1. AP sends an **ANonce** (random value) to the client
2. Client generates its own **SNonce** and derives the **PTK** (Pairwise Transient Key) using both nonces, the MAC addresses, and the **PMK** (derived from the passphrase via PBKDF2)
3. Client sends its SNonce + a MIC (Message Integrity Check) back to the AP
4. AP confirms the exchange and installs the encryption keys

### Why it's attackable
The security of WPA/WPA2-PSK is entirely dependent on the **strength of the passphrase**. The handshake itself isn't broken cryptographically — but since the PMK is derived deterministically from the passphrase and SSID, an attacker who captures a handshake can attempt an **offline dictionary or brute-force attack** with no further interaction with the network.

A more efficient modern variant is the **PMKID attack**, which doesn't require capturing a full handshake or even a connected client — the PMKID is sometimes broadcast by the AP itself in the very first EAPOL frame.

### Tools
| Tool | Purpose |
|---|---|
| `airodump-ng` | Capture the handshake (or PMKID) |
| `aireplay-ng` | Deauthenticate connected clients to force a re-handshake capture |
| `hcxdumptool` | Modern capture tool, supports PMKID extraction |
| `hcxtools` | Converts captures into hashcat-compatible formats |
| `hashcat` (mode 22000) | GPU-accelerated offline cracking |
| `aircrack-ng` | CPU-based dictionary/brute-force cracking (slower, but no GPU needed) |

This is why passphrase strength matters far more here than in enterprise environments — a weak or common passphrase can be recovered in minutes with a good wordlist and GPU.

---

## WPA2 - Enterprise (802.1X)

### How it works
Enterprise mode replaces the shared passphrase with **individual authentication** via **802.1X**, involving three components:

- **Supplicant** — the client device
- **Authenticator** — the access point, acting as a relay
- **Authentication Server** — typically a **RADIUS** server, which validates credentials against a directory (e.g., Active Directory)

Authentication happens over **EAP** (Extensible Authentication Protocol), with several common variants:
- **EAP-TLS** — certificate-based, mutual authentication (client and server both present certificates) — the strongest variant
- **PEAP-MSCHAPv2** — server authenticates via certificate, client via username/password wrapped in a TLS tunnel — very common in corporate environments
- **EAP-TTLS** — similar tunneling concept to PEAP, more flexible on the inner authentication method

### Why it's attackable
Enterprise networks are generally far more resilient than PSK networks, but the most common weakness isn't cryptographic — it's **client misconfiguration**. If a client isn't configured to validate the RADIUS server's certificate, it can be tricked into authenticating against a **rogue Access Point** impersonating the legitimate network.

In that scenario, the attacker's rogue AP acts as a fake authentication server, capturing the **MSCHAPv2 challenge-response exchange** from a connecting client — which can then potentially be cracked offline (MSCHAPv2 has known cryptographic weaknesses due to its reliance on DES).

### Tools
| Tool | Purpose |
|---|---|
| `hostapd-wpe` | Sets up a rogue AP that impersonates a legitimate Enterprise network and harvests EAP credentials |
| `eaphammer` | Automates rogue AP attacks against WPA2-Enterprise, supports EAP downgrade attacks |
| `asleap` | Offline cracking of captured MSCHAPv2 exchanges |
| Wireshark | Manual inspection of EAP frames during analysis |

The defense here is straightforward but often overlooked: enforcing **certificate validation** on the client side (so it refuses to authenticate against an untrusted server) neutralizes this entire attack class.

---
## All-in-One Automated Suites

While understanding each protocol and running tools manually is essential for learning, in practice many pentesters rely on automated frameworks that wrap all the tools mentioned above into a single guided interface — especially useful for quickly covering all attack vectors during an engagement.

### airgeddon
Probably the most widely used all-in-one wireless auditing suite. It's a bash script that acts as a front-end for most of the tools already covered (`airodump-ng`, `aireplay-ng`, `reaver`, `bully`, `hashcat`, `hostapd-wpe`, etc.), presented through an interactive menu. It supports:
- WEP attacks (IV capture and statistical cracking)
- WPS PIN attacks (both online brute-force and Pixie Dust)
- WPA/WPA2 handshake and PMKID capture, plus dictionary cracking
- Evil Twin attacks with captive portal phishing to harvest the passphrase directly from users
- DoS/deauthentication attacks for testing client resilience

It's a good tool for efficiency once you already understand *why* each attack works — using it without that understanding just turns you into someone clicking menu options.

### wifite
Similar philosophy to airgeddon but more automated and less interactive — you point it at a target (or let it scan and choose automatically) and it cycles through WEP, WPS, and PMKID/handshake attacks with minimal input, prioritizing the fastest/most likely attack vector first.

### Fluxion
Focused specifically on **social engineering attacks** against WPA/WPA2 — it creates an Evil Twin AP with a fake captive portal (often mimicking a router firmware update or login page) to trick the user into typing the real passphrase, which is then validated against a captured handshake in real time.

### EAPHammer (revisited)
Already mentioned for WPA2-Enterprise, but worth repeating here — it's essentially the automated/framework equivalent for Enterprise rogue AP attacks, the same way airgeddon/wifite are for PSK and WEP.

### A note on using automated tools
These frameworks are excellent for speed and coverage during real engagements, but they can create a false sense of competence if used without understanding the underlying protocol weaknesses. In an OSCP-style exam or a real client audit, being able to explain *why* an attack worked — not just that it worked — is what separates a script-runner from an actual pentester.

---
## Summary Comparison

| Protocol | Encryption | Main Weakness | Practical Risk Today |
|---|---|---|---|
| WEP | RC4 | Small IV space, statistically broken | Critical — deprecated, avoid entirely |
| WPS | N/A (PIN-based) | Split PIN validation / weak entropy (Pixie Dust) | High — disable WPS entirely |
| WPA/WPA2-PSK | TKIP / AES-CCMP | Passphrase strength, offline handshake cracking | Medium — depends entirely on passphrase quality |
| WPA2-Enterprise | AES-CCMP + 802.1X | Client-side cert validation, rogue AP impersonation | Low-Medium — strong if properly configured |

---

Understanding these mechanisms at a protocol level is what separates running a tool blindly from actually knowing *why* an attack works — and, just as importantly, how to properly harden a network against it.