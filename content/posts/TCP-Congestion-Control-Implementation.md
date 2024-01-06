---
date: "2024-01-05T18:12:44-08:00"
title: "TCP Congestion Control Implementation"
authors: []
categories:
  - Network
tags:
  - C
draft: false
---

## Overview
Implemented communication between multiple hosts using the sliding window protocol and TCP Reno's Congestion Control. The primary task of the project to is ensure messages are completely transmitted between hosts despite the possibility of frames dropping due to data corruption and congestion control measures.

## Objectives
- Deploy the Sliding Window Protocol with selective retransmission
- Integrate the following elements of Congestion Control
  - Slow Start: Rapidly increases congestion window at the start of a connection
  - Fast Recovery: Upon a segment loss, updates congestion window and slow start threshold to circumvent another slow start
  - Fast Retransmission: Retransmits lost segments swiftly to avoid timeouts
  - Additive Increase / Multiplicative Decrease: Increases data sending rate linearly and decreases it exponentially when encountering congestion

## Process
This project was split into two main parts: implementing the sliding window protocol and then implementing Congestion Control. The main functions I implemented on the sender side include the following:

- `host_get_next_expiring_timeval()` - returns the frame closest to expiring
- `handle_incoming_acks()` - determines if ack is valid, updates Last Ack Received, and shifts the remaining frames in `send_window` to the left
- `handle_input_cmds()` - fragments input command into multiple frames to be sent out
- `handle_timedout_frames()` - set the timeout of frames in send_window to NULL if any frame in window times out
- `handle_outgoing_frames()` - loops through the `send_window` to send out all NULL timeout frames first then buffered frames

On the receiving side, I implemented `handle_incoming_frames()` which drops corrupted frames, sends cumulative acknowledgements back, and combines payloads of frames into a final message to print.


### Debugging
The two main tools I used for this project was Valgrind and printing to the command line. Valgrind was mostly used to correct mistakes that occurred when allocating memory for strings. Printing to the command line greatly helped me figure out errors in my circular array used to store buffered frames.

## Output
![CC](../../images/cc.jpg)
**Key**
- Rtt: *Round Trip Time*
- ack_received: *# of acks received*
- dup_acks: *# of duplicate acks received*
- state: *Congestion Control State*
- Cwnd: *Congestion Window Size*
- Ssthresh: *Slow Start threshold*
- Frames_sent: *# of frames sent*
- Frames_dropped: *# of frames dropped*
- Frames_in_sender_window: *# of outstanding frames inside the sender window*
- Timedout_frames: *# of timedout frames inside the sender window*