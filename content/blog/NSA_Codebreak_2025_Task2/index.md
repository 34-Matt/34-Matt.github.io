---
title: "NSA Codebreak 2025 Writeup - Task 2"
date: 2026-01-21
tags: ["CTF", "NSA"]
categories: ["Computer Science"]
description: "Forensic analysis on suspicious device"
draft: false
---

# Task 2 - The hunt continues - (Network Forensics)

{{< figure src="badge2.png" alt="Badge" caption="Badge as proof for completing task 2" >}}

## Problem Statement

With your help, the team concludes that there was clearly a sophisticated piece of malware installed on that endpoint that was generating some network traffic. Fortunately, DAFIN-SOC also has an IDS which retained the recent network traffic in this segment.

DAFIN-SOC has provided a PCAP to analyze. Thoroughly evaluate the PCAP to identify potential malicious activity.

Submit all the IP addresses that are assigned to the malicious device, one per line.

## Solution

### Task 1 - Setup Workspace

The provided file is a PCAP file that contains network traffic called `traffic.pcap`. This file is compatible with Wireshark.

### Task 2 - Initial Breakdown

Before looking at Wireshark, I used the TShark terminal commands to identify all IP addresses associated with each device on the network. Using the command below, I created a tab-separated value file with columns for mac addresses, source IP addresses, destination IP addresses, and internet protocol.

```bash
tshark -r traffic.pcap   -Y "sll.src.eth && ip.src && ip.dst && ip.proto"   -T fields   -e sll.src.eth   -e ip.src   -e ip.dst -e ip.proto > mac2ipcount.tsv
```

Using Excel, I created a pivot table, with columns being the protocol, rows being the mac address followed by the source IP address, and values being the source count. This gave me a pivot table that I could easily reference to identify the IP addresses assigned to each device and the types of protocol they were using.

### Task 3 - Network Forensic

> There are many avenues at this point that I pursued, and probably many more that I did not check. Rather than going through every possibility, I will present the line of reasoning I used to obtain the solution.

I began by looking at the protocols sent over the network. Of note, I noticed a few files being shared, multiple ARP requests, networking with UDP and NBNS, and devices being designated and searching for addresses. No malicious pattern emerged from initial review of the traffic.

Next, I began looking at patterns around each device in the pivot table using the following search query: `ip.addr == IP_1 || ip.addr == IP_2 ...` (was unable to search by mac address, likely due to the traffic being simulated). Here was where I identified a device that used the same IP address as another device to step in using ICMP, then used SSH to encrypt communication with another device. This was the most suspicious pattern I observed and validated them as malicious addresses but was told I was missing one address.

At this point, the challenge had ended. I still wanted to finish the task, so reached out to my cybersecurity friend Evan to aid me with the missing IP address.

While I wanted to read the files being shared across the network, I did not realize I could read an entire tcp stream to read the full file with the follow -> TCP Stream option. Beyond the incredible RFC2549 read, I found three router backup files containing various interfaces and gateways. The missing address was a gateway found in one of these files, where the device was assigned the initial IP address used to access the network.