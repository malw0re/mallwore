---
title: "MSF CTF II"
date: 2023-10-05
draft: false
tags: ["Metasploit-ctf"]
description: "Metasploit ctf series two"
---

## CTF Scenario.

- This lab is dedicated to you! No other users are on this network
- Once you start the lab, you will have access to a Kali terminal based instance. This machine has an interface with IP address 192.X.Y.Z.
- The lab has two target machines which should be located on the same subnet  i.e. 192.X.Y.0/24
- The target systems most of times, belong to "admin" or "root" user.

# Flags.

- This lab has only 1 Flag to be collected.
- The Flag can be identified either by the strings flag, flagX, FLAG, FLAGX.

## Recon

Get the IP for the target we need to compromise, the run some nmap on them.
![](https://i.imgur.com/Oi76QKA.png)
![](https://i.imgur.com/831j10q.png)

## Target 1

Checking off the scan from `target-1` we can see it's running an outdated version of Werkzeug, a utility library for Python that provides tools and libraries for building web applications hence vulnerable to remote code execution. 

Hoping on metasploit we find the module that can exploit the outdated version.
![](https://i.imgur.com/dBobIFE.png)

To be continued ......