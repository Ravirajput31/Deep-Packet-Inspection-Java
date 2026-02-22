DPI Engine - Deep Packet Inspection System (Java Edition)
This document explains everything about this project—from basic networking concepts to the complete Java code architecture. After reading this, you should understand exactly how packets flow through the system without needing to read the code.

Table of Contents
What is DPI?

Networking Background

Project Overview

File Structure

The Journey of a Packet

Deep Dive: Each Component

How Signature Matching Works (Aho-Corasick)

Building and Running

Understanding the Output

1. What is DPI?
Deep Packet Inspection (DPI) is a technology used to examine the contents of network packets as they pass through a checkpoint. Unlike simple firewalls that only look at packet headers (source/destination IP), DPI looks inside the packet payload to identify the specific application being used.

Real-World Uses:
Security: Detect malware or intrusion attempts hidden in data.

Network Management: Identify bandwidth-heavy apps like YouTube or Netflix.

Parental Controls: Block inappropriate websites or keywords.

2. Networking Background
The Network Stack (Layers)
When you visit a website, data travels through multiple "layers":

┌─────────────────────────────────────────────────────────┐
│ Layer 7: Application    │ HTTP, TLS, DNS (Our Target!)  │
├─────────────────────────────────────────────────────────┤
│ Layer 4: Transport      │ TCP (reliable), UDP (fast)    │
├─────────────────────────────────────────────────────────┤
│ Layer 3: Network        │ IP addresses (routing)        │
├─────────────────────────────────────────────────────────┤
│ Layer 2: Data Link      │ MAC addresses (local hardware)│
└─────────────────────────────────────────────────────────┘
A Packet's Structure
Every network packet is like a Russian nesting doll—headers wrapped inside headers:

3. Project Overview
What This Java Engine Does
Live Traffic (Wi-Fi) → [PcapReader] → [PacketParser] → [RuleManager]
                            ↓              ↓              ↓
                     Capture Raw Data   Decode TCP/IP   Scan Content
This project uses Pcap4J for hardware interaction and the Aho-Corasick algorithm for high-speed pattern matching.

4. File Structure
Plaintext
DeepPacketInspection/
├── src/main/java/org/example/
│   ├── PcapReader.java        # Main engine: Opens Wi-Fi and starts the loop
│   ├── PacketParser.java      # Analyst: Decodes IP/TCP layers
│   ├── RuleManager.java       # The Brain: Matches keywords in payloads
│   ├── ConnectionTracker.java # Scoreboard: Tracks packet counts per app
│   └── NetworkScanner.java    # Utility: Finds available network cards
├── pom.xml                    # Maven: Manages Pcap4J and Aho-Corasick libraries
└── README.md                  # This file!
5. The Journey of a Packet
Step 1: Capture Raw Data (PcapReader.java)
The engine identifies your Wi-Fi card (e.g., Intel Wi-Fi 6E) and opens a "Live Handle". It puts the card in Promiscuous Mode to see all traffic passing through your machine.

Step 2: Protocol Decoding (PacketParser.java)
The raw bytes are converted into Java objects using Pcap4J. We extract the "Five-Tuple" (Src IP, Dst IP, Src Port, Dst Port, Protocol) to identify the connection.

Step 3: Payload Extraction
If a packet is TCP, we extract the rawData (the actual message).

Port 80 (HTTP): The data is visible in plain text.

Port 443 (HTTPS): The data is encrypted, but keywords can often be found in the initial handshake.

Step 4: Pattern Matching (RuleManager.java)
The raw bytes are sent to a Trie (Finite State Machine). This allows us to search for "youtube," "google," and "facebook" all at the same time in a single pass through the data.

Step 5: Scoreboard Update (ConnectionTracker.java)
If a keyword is found, an alert is triggered, and a HashMap increments the packet count for that specific application.

6. Deep Dive: Each Component
PcapReader.java
Purpose: The entry point of the application. It manages the PcapHandle.loop(), which ensures every packet captured by the hardware is processed by our Java code.

PacketParser.java
Purpose: Translates bits into information. It specifically looks for IpV4Packet and TcpPacket classes within the raw capture.

RuleManager.java
Purpose: High-speed inspection. By using the org.ahocorasick.trie.Trie, it avoids the slowness of standard String.contains() calls, making it fast enough for real-time traffic.

7. How Signature Matching Works (Aho-Corasick)
Instead of checking signatures one by one, we use a single automated search tree:

Build Phase: All keywords (YouTube, Google, etc.) are added to a Trie.

Scan Phase: As packet data flows through, the engine moves between "states" in the tree.

Emit Phase: If the engine reaches a "leaf" node, it "emits" a match.

8. Building and Running
Prerequisites
Java 21+ and Maven.

Npcap (Windows): Must be installed in "WinPcap API-compatible mode".

Administrator Privileges: Required to access raw network hardware.

How to Run
Open IntelliJ IDEA as Administrator.

Run PcapReader.java.

Open a browser and visit http://neverssl.com (Unencrypted) to see immediate detections.

9. Understanding the Output
When running, you will see a live update in the IntelliJ console:

Plaintext
Opening device: Intel(R) Wi-Fi 6E AX211
Listening for packets...
[TCP] 10.104.115.61:61234 -> 142.250.190.46:443
>>> ALERT: Detected Application: GOOGLE <<<

--- LIVE TRAFFIC SCOREBOARD ---
APPLICATION: GOOGLE | TOTAL PACKETS: 15
APPLICATION: YOUTUBE | TOTAL PACKETS: 4
--------------------------------
