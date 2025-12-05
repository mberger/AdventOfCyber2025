---
layout: post
title:  "Welcome to Advent of Cyber 2025 - Day 4 - AI in Security - old sAInt nick"
date:   2025-12-04
categories: welcome
image: Day4-banner.png # optional – you can delete this block if no image
---

# AI in Security - old sAInt nick

TryHackMe Room: <https://tryhackme.com/room/AIforcyber-aoc2025-y9wWQ1zRgB> Advent of Cyber 2025

### Quick Overview

* **Difficulty**: Easy (gentle intro to AI concepts, no heavy coding)
* **Type**: Interactive AI showcase + guided exploration
* **Time**: \~30 minutes
* **Goal**: Discover how AI boosts cybersecurity—defending, attacking, and building—through hands-on chats with a virtual elf assistant

### The Story in a Nutshell

Forget the North Pole drama for a sec; we're at The Best Festival Company (TBFC), a festive tech hub buried in snow. Their old chatbot, Van Chatty, was a dud, so the elves are testing a shiny new AI sidekick called Van SolveIT. It's all about cranking up productivity by handing off the boring stuff—like spotting bugs or chasing hackers—before the holiday rush hits. You dive in as the tester, chatting with Van SolveIT to see if it can really save Christmas (or at least the company's code).

### What You Actually Do (Task Flow)


1. Fire up the AttackBox and target machine (hit those buttons, wait 2 mins for the magic). Confirm it's rolling.
2. Hop to http://MACHINE_IP and meet Van SolveIT—the AI that's equal parts helpful and holiday-spirited.
3. Tackle three interactive showcases, chatting your way through:
   * **Red Team (Offensive)**: Ask Van to whip up an exploit script for a vulnerable web app at MACHINE_IP:5000. Tweak the IP in the code, run it, and snag the flag it spits out.
   * **Blue Team (Defensive)**: Feed Van some web logs from a sneaky attack and let it break down what went wrong.
   * **Software Dev**: Hand over sketchy source code; watch Van hunt for vulnerabilities like a digital code cop.
4. Wrap it up by answering a quick question on the big flag Van reveals at the end.

### Key Concepts/Tools You'll Actually Use and Remember

* **AI Superpowers**: It crunches massive data for anomaly spotting, generates scripts on the fly, and summarizes logs without the headache.
* **Defensive Plays**: AI automating threat hunts, blocking bad emails, or isolating rogue devices in firewalls/IDS.
* **Offensive Edge**: Recon automation, OSINT digs, and mapping attack surfaces faster than manual grinding.
* **Dev Tools**: AI as your "virtual colleague" for brainstorming, plus SAST/DAST scans to catch code flaws early.
* **Pro Tips**: Always double-check AI outputs (it's smart, but not infallible); watch for privacy pitfalls and don't over-rely on it.
* **Hands-On Bits**: Chat prompts like "Generate an exploit for this vuln" or "Analyze these logs"; simple script runs (no complex commands, just copy-paste and execute).

### Why It’s Awesome for Beginners (to AI)

* Zero setup stress—just chat like you're texting a savvy elf buddy, and it handles the heavy lifting.
* Real "aha" moments: See AI turn vague asks into actionable exploits or vuln reports, tying it all to cyber roles.
* Balanced warnings: Covers the cool stuff *and* the gotchas, like model biases or data leaks, without getting preachy.
* Festive twist: Van SolveIT's got that holiday cheer, making dry topics feel like a cozy workshop session.


Spin up the machines, crack open that chat window, and let Van SolveIT show you the future of cyber (spoiler: it's chatty and capable). By the end, you'll be eyeing your own tools with fresh AI envy. Day whatever-this-is unlocked—elves everywhere are high-fiving. 🎄

## Task 1

 ![task banner for day 4. Showing a wreath, with baubles hanging with various AI and cyber security related iconography](https://tryhackme-images.s3.amazonaws.com/user-uploads/5de96d9ca744773ea7ef8c00/room-content/5de96d9ca744773ea7ef8c00-1763383093538.png)

The lights glimmer and servers hum blissfully at The Best Festival Company (TBFC), melting the snow surrounding the data centre. TBFC has continued its pursuit of AI excellence. After the past two years, they realise that Van Chatty, their in-house chatbot, wasn’t quite meeting their standards. 

Unfortunately for the elves at TBFC, they are also not immune to performance metrics. The elves aim to find ways of increasing their velocity; something to manage the tedious, distracting tasks, which allows the elves to do the real magic. 

TBFC, adventurous as ever, is trialling their brand new cyber security AI assistant, Van SolveIT, which is capable of helping the elves with all their defensive, offensive, and software needs. They decide to put this flashy technology to use as Christmas approaches, to identify, confirm, and resolve any potential vulnerabilities, before any nay-sayers can.

##  ![an open book.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5de96d9ca744773ea7ef8c00/room-content/5de96d9ca744773ea7ef8c00-1762271195732.png)Learning Objectives

* How AI can be used as an assistant in cyber security for a variety of roles, domains and tasks
* Using an AI assistant to solve various tasks within cyber security
* Some of the considerations, particularly in cyber security, surrounding the use of AI

## Connecting to the Machine

Before moving forward, review the questions in the connection card shown below:

 ![Connection card for day 4. Both the target VM and AttackBox need to be deployed.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5de96d9ca744773ea7ef8c00/room-content/5de96d9ca744773ea7ef8c00-1763383175743.png)
Start your target VM by clicking the **Start Machine** button below. The machine will need about 2 minutes to fully boot. Additionally, start your AttackBox by clicking the **Start AttackBox** button below. The AttackBox will start in split view. In case you can not see it, click the **Show Split View** button at the top of the page.


### Set up your virtual environment

To successfully complete this room, you'll need to set up your virtual environment. This involves starting both your AttackBox (if you're not using your VPN) and Target Machines, ensuring you're equipped with the necessary tools and access to tackle the challenges ahead.

 ![](https://tryhackme.com/static/svg/attack-box-to-target-machine.e30b7a02.svg)

Attacker machine ![Machine info](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTciIGhlaWdodD0iMTYiIHZpZXdCb3g9IjAgMCAxNyAxNiIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHBhdGggZD0iTTguODI1MiAwQzQuNDA2NDUgMCAwLjgyNTE5NSAzLjU4MTI1IDAuODI1MTk1IDhDMC44MjUxOTUgMTIuNDE4NyA0LjQwNjQ1IDE2IDguODI1MiAxNkMxMy4yNDM5IDE2IDE2LjgyNTIgMTIuNDE4NyAxNi44MjUyIDhDMTYuODI1MiAzLjU4MTI1IDEzLjI0MzkgMCA4LjgyNTIgMFpNOC44MjUyIDRDOS4zNzczOCA0IDkuODI1MiA0LjQ0NzgxIDkuODI1MiA1QzkuODI1MiA1LjU1MjE5IDkuMzc3MzggNiA4LjgyNTIgNkM4LjI3MzAxIDYgNy44MjUyIDUuNTUzMTIgNy44MjUyIDVDNy44MjUyIDQuNDQ2ODggOC4yNzIwNyA0IDguODI1MiA0Wk0xMC4wNzUyIDEySDcuNTc1MkM3LjE2MjcgMTIgNi44MjUyIDExLjY2NTYgNi44MjUyIDExLjI1QzYuODI1MiAxMC44MzQ0IDcuMTYxMTMgMTAuNSA3LjU3NTIgMTAuNUg4LjA3NTJWOC41SDcuODI1MkM3LjQxMTEzIDguNSA3LjA3NTIgOC4xNjQwNiA3LjA3NTIgNy43NUM3LjA3NTIgNy4zMzU5NCA3LjQxMjcgNyA3LjgyNTIgN0g4LjgyNTJDOS4yMzkyNiA3IDkuNTc1MiA3LjMzNTk0IDkuNTc1MiA3Ljc1VjEwLjVIMTAuMDc1MkMxMC40ODkzIDEwLjUgMTAuODI1MiAxMC44MzU5IDEwLjgyNTIgMTEuMjVDMTAuODI1MiAxMS42NjQxIDEwLjQ5MDggMTIgMTAuMDc1MiAxMloiIGZpbGw9IiM4NzhGQTIiLz4KPC9zdmc+Cg==)Status:Off

Target machine ![Machine info](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTciIGhlaWdodD0iMTYiIHZpZXdCb3g9IjAgMCAxNyAxNiIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHBhdGggZD0iTTguODI1MiAwQzQuNDA2NDUgMCAwLjgyNTE5NSAzLjU4MTI1IDAuODI1MTk1IDhDMC44MjUxOTUgMTIuNDE4NyA0LjQwNjQ1IDE2IDguODI1MiAxNkMxMy4yNDM5IDE2IDE2LjgyNTIgMTIuNDE4NyAxNi44MjUyIDhDMTYuODI1MiAzLjU4MTI1IDEzLjI0MzkgMCA4LjgyNTIgMFpNOC44MjUyIDRDOS4zNzczOCA0IDkuODI1MiA0LjQ0NzgxIDkuODI1MiA1QzkuODI1MiA1LjU1MjE5IDkuMzc3MzggNiA4LjgyNTIgNkM4LjI3MzAxIDYgNy44MjUyIDUuNTUzMTIgNy44MjUyIDVDNy44MjUyIDQuNDQ2ODggOC4yNzIwNyA0IDguODI1MiA0Wk0xMC4wNzUyIDEySDcuNTc1MkM3LjE2MjcgMTIgNi44MjUyIDExLjY2NTYgNi44MjUyIDExLjI1QzYuODI1MiAxMC44MzQ0IDcuMTYxMTMgMTAuNSA3LjU3NTIgMTAuNUg4LjA3NTJWOC41SDcuODI1MkM3LjQxMTEzIDguNSA3LjA3NTIgOC4xNjQwNiA3LjA3NTIgNy43NUM3LjA3NTIgNy4zMzU5NCA3LjQxMjcgNyA3LjgyNTIgN0g4LjgyNTJDOS4yMzkyNiA3IDkuNTc1MiA3LjMzNTk0IDkuNTc1MiA3Ljc1VjEwLjVIMTAuMDc1MkMxMC40ODkzIDEwLjUgMTAuODI1MiAxMC44MzU5IDEwLjgyNTIgMTEuMjVDMTAuODI1MiAxMS42NjQxIDEwLjQ5MDggMTIgMTAuMDc1MiAxMloiIGZpbGw9IiM4NzhGQTIiLz4KPC9zdmc+Cg==)Status:Off

Answer the questions below

I have successfully started the AttackBox and the target machine!

## Task 2

AI for Cyber Security Showcase

## The Boom of AI Assistants


Ah, yes, artificial intelligence, that buzzword that seems here to stay. Who would have thought? Today’s room will highlight some ways AI is utilised in cyber security, along with important considerations to bear in mind when deploying AI for such tasks.

Particularly at the time of writing, AI is increasingly seen as a tool to boost speed by handling often tedious, time-consuming tasks, allowing humans to perform the real magic. Organisations want to see experience, not avoidance, in how these tools are operated.

GPT this, GPT that, we’ve all heard it often. And it’s likely to persist. As AI’s capabilities expand daily, we’ve observed a shift from AI being just “something to ask because you were too lazy to Google” (a mistake I’ve made myself). Now, AI is being embedded into everyday workflows, transforming how tasks are done and boosting productivity like never before. 

With that said, let’s begin today’s room!

 ![a christmas bauble ornament with an AI robot sitting inside of it.](https://tryhackme-images.s3.amazonaws.com/room-icons/5de96d9ca744773ea7ef8c00-1762182384720 "right-50")

## AI in Cyber Security

The use of artificial intelligence has seen a significant boost in cyber security. Visit almost any vendor, and they'll now have some form of AI powering a solution somewhere. Why? Well, it's not just because they're capitalising on the buzzword (although that's certainly *a part of it*), but rather, the benefits from artificial intelligence really do apply here. Let's explore some of these in the table below:


| **Features of AI** | **Cyber Security Relevance** |
|----|----|
| Processing large amounts of data | Analysing vast data from multiple types of sources. For example, system and network logs together. |
| Behaviour analysis | Tracking normal behaviour and activities over a period of time and flagging anything that is out of the ordinary. |
| Generative AI | Summarising or providing context behind a series of events. |


## Defensive Security


 ![a piece of paper on a clipboard with a magnifying glass inspecting it.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5de96d9ca744773ea7ef8c00/room-content/5de96d9ca744773ea7ef8c00-1762271394697.png "left-50")

AI agents are being used in blue teaming to speed up detection, investigation, and response, making them quicker and more dependable. Acting like automated assistants, these agents continuously process telemetry (logs, network flows, endpoint signals) and add context to alerts. Furthermore, we are witnessing the integration of AI into vendor appliances—such as AI-assisted firewalling and intrusion detection systems.

Beyond just detecting threats, AI can also assist in automating responses. Picture your system automatically isolating an infected device, blocking a suspicious email, or flagging an unusual login attempt — all in real time.

## Offensive Security

AI agents have made a notable impact on offensive security by automating and handling the often very labourious and time-consuming tasks that a pentester might traditionally undertake. 

For example, AI can be a powerful tool in a penetration tester's workflow for reconnaissance and information gathering, from OSINT to analysing noisy scanner outputs and mapping attack surfaces. This allows the pentester to spend more time on the crucial tasks that require a human touch.

## Software

 ![three gears/cogs closely placed with one another.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5de96d9ca744773ea7ef8c00/room-content/5de96d9ca744773ea7ef8c00-1762271395568.svg "right-50")

AI-driven software development, rightfully, sounds a bit frightening. Isn't that so? Well, you wouldn't be wrong to feel this way; we've all heard about the popularity of vibe coding and the vulnerabilities introduced by AI.

However, AI has proven to be a valuable addition to the software development process in several ways. One example is a virtual "colleague" to bounce ideas off while writing the code itself. More importantly, it is used as a SAST/DAST scanner. These scanners audit and analyse written code and applications for potential vulnerabilities. 

Yes, it's somewhat ironic. AI agents can be great at identifying vulnerabilities, but are not quite as effective at writing secure code.

## Considerations of AI In Cyber Security


 ![a red closed book with the text "rules" on the cover.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5de96d9ca744773ea7ef8c00/room-content/5de96d9ca744773ea7ef8c00-1762271477832.svg "right-50")

Now, I’m not entirely here to sing the praises of AI and say it’s the silver bullet to all your needs. If you’ve used AI before, you’ll know the pitfalls and frustrations one can face. And nowhere is that truer than in cyber security.

While the usual considerations of using AI apply, such as not owning the output from AI, there are specific factors to consider before deploying it in cyber security.

One such consideration is the use of AI in activities like offensive penetration testing. While we have discussed some of AI's applications in these areas, caution remains essential. You do not want to explain to a client that their services and websites are down because an AI has caused a race condition or overwhelmed their systems.

We must think carefully about the data AI learns from, how transparent and fair its decisions are, and how reliable it remains when the unexpected occurs. We cannot assume the output from AI is 100% correct. Efforts must be made to verify the information it provides. Additionally, managing challenges such as keeping data private, securing AI models, and informing users properly requires careful consideration.

## Practical

Phew! Ready for an exciting exercise? You will be interacting with Van SolveIT, who will guide you through three showcases of how AI can be used in cyber security:

* **Red:** Generate and use an exploit script.
* **Blue:** Analyse web logs of an attack that has occurred.
* **Software:** Analyse source code for vulnerabilities.

When you're ready, you can access Van SolveIT at `http://MACHINE_IP`. Remember, you will need to do so either from the AttackBox or your own device connected to the VPN.

*If you are on a small display, we recommend expanding the AttackBox into full screen mode which can be done by pressing the "two arrows" icon (left of the "+" icon) in the split-screen view to expand it into full screen.*

## Usage Tips

* Chatbot responses may appear blank for a minute or two while it generates the reply. You will start to see Van SolveIT's responses in real time.
* If the chatbot gets confused at any time, press the **Restart Chat** button at the top right of the page.
* As you progress throughout the showcase, stages will unlock. You can go back to any stage that you have unlocked by clicking on the stage name on the top left.

Answer the questions below

Complete the AI showcase by progressing through all of the stages. What is the flag presented to you?
THM{AI_MANIA}

Execute the exploit provided by the red team agent against the vulnerable web application hosted at `MACHINE_IP:5000`. What flag is provided in the script's output after it?
THM{SQLI_EXPLOIT}

Remember, you will need to update the IP address placeholder in the script with the IP of your vulnerable machine (`MACHINE_IP:5000`)

If you enjoyed today's room, feel free to check out the [Defending Adverserial Attacks](https://tryhackme.com/room/defadversarialattacks) room, where you will learn how to harden and secure AI models.
