---
date: "2023-12-29T10:21:43-08:00"
title: "Custom Router"
authors: ["michaelye"]
categories:
  - Network
tags:
  - C
  - Mininet
draft: false
---

## Overview
In the project, I designed and implemented a router on a static network topology to process raw Ethernet frames. To emulate a multi-router topology on my laptop, I used [Mininet](https://mininet.org/) which provided the routers, servers and other clients used in the network. The router specifically handled ARP requests and replies (updating the router's forwarding table with IP/MAC mappings), IP packets (with additional functionality to detect TCP/UDP for traceroute support), ICMP packets (requests,replies, and messages for Destination net, host, and port unreachable). After implementing everything above, my router can successfully handle Pings, Traceroutes, and file downloads using HTTP.

## Sample Network Topology
![topology](../../images/topology.jpg)

## Process

This project was split into two parts:
- Implement simple routing (no subnetting, ARP caching)
- Implement subnet routing, ARP caching, tracerroute support

### Implementation
Implementing the router was initially rather straightforward. Most of the coding was mostly just determining what type of packet the router received, and then taking the appropriate decision. After creating a rough outline of my router (similar to the code below), I began filling in the details. 

To make my code efficient, I employed various techniques to keep the number of new packets malloc'd to a minimum. This included cannibalizing some of the packets the router received and simply changing the values in the correct fields. For example, when sending back out an ARP reply, the router would take the ARP request and change the target and source IP and MAC addresses to the appropriate values. 

Implementing subnet was relatively simple with the main portion of the router completed, with the main nuance being that the router now had to recursively search up the forwarding interface from the routing table.

### Debugging
The main issues I came across mostly boiled down to Network Endianness vs Host Endianness mix-ups and incorrect fields stemming from my code incorrectly cannibalizing packets.

To fix these issues, I mainly relied on Wireshark to reveal the contents of the ARP, IP, and ICMP packets.
Then it came down to correct utilization of C's `ntohl()`, `htonl()`, and other functions of the sort.

## Code Outline
*Actual code written in C*
```Python

handlePacket():
  # check parameters
  
  if received ARP PACKET:
    # sanityCheck
    # check if router interface matches arp's target IP

    if RECEIVED ARP REPLY:
      # cache IP/MAC mapping
      # handle invalid ARP reply here
      # send out queued IP packets waiting on ARP reply
    
    else:
      # SEND OUT ARP REPLY

  elif HANDLING IP PACKET:
    # sanity check section
    # CHECK OUR INTERFACES

    if PACKET IS FOR OUR ROUTER:
      # ICMP PORT UNREACHABLE code
      # ECHO REQUEST code

    elif FORWARDING PACKET:
      # ICMP TIME EXCEEDED code
      # FINDING ROUTING TABLE ENTRY
      # ICMP NET UNREACHABLE code

      # Updating Packet

      if ARP IS CACHED:
        # send out packet
      
      else:
        # ARP NOT CACHED, NEED TO ARP REQ
        # OUTGOING ARP REQUEST
        # Queue packet

```

This project was mainly based on a CSE 123 (Computer Networks) programming assignment taught by Professor Alex Snoeren.