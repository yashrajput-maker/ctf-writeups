---
title: Rogue Tower
event: picoCTF 2026
category: Forensics
difficulty: Medium
points: ""
date: 2026-11-10
status: WriteUp
publish_github: true
flag_verified: true
---
# Rogue Tower

## Challenge Information

|Field|Details|
|---|---|
|Event|picoCTF 2026|
|Challenge Name|Rogue Tower|
|Category|Forensics|
|Difficulty|Medium|
|Author|Samuel Dinesh|
|Date|2026-11-10|

## Challenge Description

A suspicious cell tower has been detected in the network. The objective is to analyze the captured network traffic to identify the rogue tower, locate the compromised device, and recover the exfiltrated flag.

The challenge provides a network capture file (`rogue_tower.pcap`) for investigation.

Challenge hints:

1. Look for unauthorized test network broadcasts on UDP port `55000`.
    
2. Find the device that connected to the rogue tower by checking HTTP User-Agent headers.
    
3. The encryption key is derived from the victim device's IMSI.
    
4. The exfiltrated data is split across multiple HTTP POST requests.
    ![[overview.png]]

## Objective

Analyze the provided PCAP file to identify the suspicious cell tower, extract the victim device's IMSI, reconstruct the exfiltrated data, and recover the challenge flag.

## Tools and Environment

- Operating System: Kali Linux
    
- Network Analysis: Wireshark
    
- Data Decoding: CyberChef
    
- Input File: `rogue_tower.pcap`
    

## Solution Walkthrough

### Step 1: Initial Network Traffic Analysis

First, download the provided `rogue_tower.pcap` file from the challenge resources.

Open the capture file in Wireshark using:

```
wireshark rogue_tower.pcap
```

After opening the capture, examine the network traffic and organize the packets by protocol.

According to the first hint, the investigation should focus on unauthorized test network broadcasts over UDP port `55000`.

Locate and analyze the UDP packets associated with this port.

During the investigation, the following Cell ID was identified:

```
Cell ID: 94309
```

This identifies the cell ID observed in the suspicious UDP traffic and provides a starting point for investigating the rogue tower.
![[udpfollow.png]]
### Step 2: Identifying the Compromised Device

The second hint points toward HTTP User-Agent headers.

Analyze the HTTP GET requests in Wireshark and inspect their User-Agent headers to identify the device associated with the suspicious network activity.

One packet contained the relevant device information, including the following IMSI:

```
IMSI: 310410743419304
```

According to the third hint, the victim device's IMSI is used to derive the encryption key.

Therefore, this IMSI becomes an important piece of information for decrypting the recovered data.
![[httpsget.png]]
### Step 3: Extracting the Exfiltrated Data

The fourth hint indicates that the exfiltrated information is divided across multiple HTTP POST requests.

Analyze the HTTP POST packets and follow their respective HTTP streams in Wireshark.
![[post1.png]]
![[post2.png]]
![[post3.png]]
![[post4.png]]
![[post5.png]]
![[post6.png]]
After examining the POST requests, the following Base64-encoded string was recovered:

```
VVBVVVxeSU9GA1NECmxTB1hfa0UJRANGa1ZRVQoLAQVRTg==
```

The recovered data is encoded, so the first step is to decode it using CyberChef.

#### 3.1 Base64 Decoding

Pass the extracted string through the Base64 Decode operation in CyberChef.

The resulting output is:

```
UPUU\^IOF
SD
lS X_kE    D
FkVQU

    QN
```

The output is not readable plaintext, indicating that another decoding or decryption step is required.
![[base64decode.png]]
#### 3.2 XOR Decryption Using the IMSI

Since the challenge mentions that the encryption key is derived from the victim device's IMSI, XOR decryption is a reasonable next step.

Use the previously identified IMSI:

```
310410743419304
```

Apply XOR decryption in CyberChef using the IMSI as the key and UTF-8 as the output format.

The resulting output is:

```
faeamn~{u7b}9\g4io_t9s7u_ghf:?24az
```

This output still does not represent the final flag.
![[XOR1.png]]
Further examination of the HTTP GET requests reveals that the initial numbers in the IMSI are repeated.

This suggests that the repeated portion should not be included when deriving the XOR key.

The non-repeated portion of the IMSI is:

```
43419304
```

This becomes the key for the next XOR decryption attempt.

### Step 4: Recovering the Final Flag

Return to CyberChef and perform XOR decryption again, this time using the non-repeated portion of the IMSI as the key.

XOR key:

```
43419304
```

After applying the XOR operation, the resulting plaintext is:

```
academy{r0gu3_c3ll_t0w3r_eed3811e}
```

This is the flag recovered from the exfiltrated network data.
![[finalflag.png]]
## Commands and Reproduction

The following command opens the provided PCAP file in Wireshark:

```
wireshark rogue_tower.pcap
```

The remaining analysis was performed using Wireshark's packet inspection and TCP stream-following functionality, followed by Base64 decoding and XOR decryption in CyberChef.

## Final Flag

```
academy{r0gu3_c3ll_t0w3r_eed3811e}
```

Flag marked verified in source notes

## Key Takeaways

- PCAP Analysis: Wireshark can be used to inspect network traffic and identify suspicious communication.
    
- UDP Investigation: Filtering traffic by a specific port, such as `55000`, helps narrow down relevant packets.
    
- HTTP Analysis: User-Agent headers can provide information about devices communicating over a network.
    
- IMSI Identification: Device identifiers can provide important clues when investigating suspicious network activity.
    
- Data Reconstruction: Following multiple HTTP POST request streams helps recover fragmented exfiltrated data.
    
- Encoding and Encryption: Base64 decoding followed by XOR decryption can reveal information hidden within captured network traffic.
    

