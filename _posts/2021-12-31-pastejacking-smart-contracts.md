---
layout: post
title: 'Pastejacking Smart Contracts: Replacing Wallet Addresses to Steal Data'
categories: [blog]
tags: crypto cybersecurity
---

The last several years have brought incredible gains in the number of cyberattacks waged. These attacks entail a variety of exploits, with some using methods as simple as social engineering. Human behavior is often the weakest link, and a particular javascript exploit takes advantage of trust in a computer’s copy-and-paste functionality.  This form of attack, commonly referred to as [pastejacking](https://www.geeksforgeeks.org/what-is-pastejacking/), can fit into many different cyberattack campaigns. This attack takes advantage of one of the most common user interactions with a computer.

Pastejacking attacks exploit temporary storage memory by replacing copied data stored in device memory with different, often malicious, data. This exploit may take the form of a client-side or a server-side attack. 

Server-side pastejacking attacks may fit within a software supply chain attack: attackers can manipulate a compromised entity to attack downstream users or clients. 
Using methods such as [cross-site scripting (XSS) attacks](https://owasp.org/www-community/attacks/DOM_Based_XSS), an attacker can wage client-side pastejacking attacks, which can be especially effective if targeting developers. Cyberattacks waged on developers often have a high return on investment because capturing developer credentials affords an attacker a significant amount of lateral movement throughout an organization.