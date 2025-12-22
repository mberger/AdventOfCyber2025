---
layout: post 
title:  "Day 20 - Race Conditions - Toy to The World" 
date:   2025-12-19
categories: welcome 
image: Day20-banner.png
---

# Race Conditions - Race Against the Bunny

TryHackMe Room: [https://tryackme.com/room/race-conditions-aoc2025-d7f0g3h6j9](https://tryhackme.com/room/race-conditions-aoc2025-d7f0g3h6j9?referrer=grok.com) Advent of Cyber 2025 – Day 19

### Quick Overview

* **Difficulty**: Medium
* **Type**: Web race condition exploitation CTF + concurrency bug tutorial
* **Time**: \~40–55 minutes
* **Goal**: Exploit timing vulnerabilities in TBFC's voucher redemption system to duplicate gifts and bankrupt Sir Carrotbane's coal-funding scheme

### The Story in a Nutshell

The North Pole's last-minute gift voucher portal is booming, but Carrotbane's spotted a flaw: the checkout process checks your voucher balance *before* deducting it, without proper locking. Bunnies are spamming redemptions in parallel to get infinite free toys, draining the budget for nice-list kids. With McSkidy recently rescued, the elves need you to demo the race condition, race the requests, duplicate vouchers like a glitchy Santa, and prove why atomic transactions save Christmas.

### What You Actually Do (Task Flow)


 1. Deploy the target VM → browse to the voucher shop at http://MACHINE_IP and register/login.
 2. **Race 101**: Learn TOCTOU (Time-of-Check-to-Time-of-Use) bugs—check balance, then use it, with a gap for racing.
 3. Grab a starting voucher (e.g., 100 credits) from the welcome promo.
 4. **Single Redemption**: Buy a low-cost item normally—see the naive flow (check > deduct).
 5. **Manual Race**: Open two tabs, start two expensive purchases at once—sometimes one succeeds twice (balance goes negative).
 6. **Automated Blitz**: Use Burp Suite's Intruder or Turbo Intruder to fire 50–100 parallel redemption requests for a high-value item.
 7. **Win the Race**: Watch your balance skyrocket (or credits multiply) as duplicates slip through the gap.
 8. **Exploit Payload**: Spam enough to buy the "ultimate gift" or hit the flag threshold.
 9. **Mitigation Theory**: Discuss fixes—database transactions, mutex locks, or deduct-first designs.
10. Answer flags: max duplicated amount, requests needed, final voucher balance.

### Key Concepts/Tools You'll Actually Use and Remember

* **Core Ideas**: Race conditions in web apps; non-atomic operations; high concurrency exploits; TOCTOU vulnerabilities.
* **Tools**: Burp Suite (Intruder for throttling/racing), Turbo Intruder extension (perfect for true parallel blasts), browser multi-tabs for manual testing.
* **Burp Tricks**:
  * Capture the redemption POST request.
  * Send to Intruder → set null payloads → race with 50–200 threads.
  * Or Turbo Intruder script for precise timing.
* **Pro Tip**: Slower networks help manual races; for reliability, crank the threads until the server cries uncle.

### Why It’s Awesome for Beginners (to Concurrency Bugs)

* That "I just duped infinite money!" moment is pure chaotic joy—feels like finding a video game glitch.
* Visual payoff: watch your voucher count explode in real-time with each replay.
* Builds on web vulns from earlier days (XSS, IDOR) but adds the timing dimension everyone's overlooking.
* Festive frenzy: Duplicating toys to counter the bunnies—turns a dry concurrency topic into holiday heist vibes.

Start the machine, proxy through Burp, and start the redemption stampede. By the end, you'll spot race conditions in any checkout flow, and Carrotbane's budget will be racing to zero. Day 19 raced—vouchers multiplied, Christmas funded! 🎄


## Task 1 - Introduction


 ![Task banner for day 20](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1763451486547.png)


The Best Festival Company (TBFC) has launched its limited-edition SleighToy, with only ten pieces available at midnight. Within seconds, thousands rushed to buy one, but something strange happened. **More than ten lucky customers received confirmation emails stating that their orders were successful**. Confusion spread fast. How could everyone have bought the "**last**" toy? McSkidy was called in to investigate.  


She quickly noticed that multiple buyers purchased at the exact moment, slipping through the system’s timing flaw. Sir Carrotbane’s mischievous Bandit Bunnies had found a way to exploit this chaos, flooding the checkout with rapid clicks. By morning, TBFC faced angry shoppers, missing stock, and a mystery that revealed just how dangerous a few milliseconds could be during the holiday rush.

## Learning Objectives


* Understand what race conditions are and how they can affect web applications.
* Learn how to identify and exploit race conditions in web requests.
* How concurrent requests can manipulate stock or transaction values.
* Explore simple mitigation techniques to prevent race condition vulnerabilities.

## Connecting to the Machine


Before moving forward, review the questions on the connection card below:


 ![Connection card.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1763836313348.png)


Start your target VM by clicking the **Start Machine** button below. The machine will need about 2 minutes to fully boot. Additionally, start your AttackBox by clicking the **Start AttackBox** button below. The AttackBox will start in split view. In case you can not see it, click the **Show Split View** button at the top of the page. Once the machine is up and running, you can access the application by visiting <http://MACHINE_IP> in the browser of your AttackBox.


### Set up your virtual environment

To successfully complete this room, you'll need to set up your virtual environment. This involves starting both your AttackBox (if you're not using your VPN) and Target Machines, ensuring you're equipped with the necessary tools and access to tackle the challenges ahead.


Answer the questions below

I can successfully connect to the machine.


## Task 2 - Race Condition

A race condition happens when two or more actions occur at the same time, and the system’s outcome depends on the ![bunny character showing car racing.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1763712901620.png) order in which they finish. In web applications, this often happens when multiple users or automated requests simultaneously access or modify shared resources, such as inventory or account balances. If proper synchronisation isn’t in place, this can lead to unexpected results, such as duplicate transactions, oversold items, or unauthorised data changes.


## Types of Race Conditions


Generally, race condition attacks can be divided into three categories:


* **Time-of-Check to Time-of-Use (TOCTOU)**: A TOCTOU race condition happens when a program checks something first and uses it later, but the data changes in between. This means what was true at the time of the check might no longer be true when the action happens. It’s like checking if a toy is in stock, and by the time you click "**Buy**" someone else has already bought it. For example, two users buy the same "last item" at the same time because the stock was checked before it was updated.
* **Shared resource**: This occurs when multiple users or systems try to change the same data simultaneously without proper control. Since both updates happen together, the final result depends on which one finishes last, creating confusion. Think of two cashiers updating the same inventory spreadsheet at once, and one overwrites the other’s work.
* **Atomicity violation**: An atomic operation should happen all at once, either fully done or not at all. When parts of a process run separately, another request can sneak in between and cause inconsistent results. It’s like paying for an item, but before the system confirms it, someone else changes the price. For example, a payment is recorded, but the order confirmation fails because another request interrupts the process.


## Time for Some Action


Now that we understand the basic concepts, let's take the example of the TBFC shopping cart app and exploit the race condition.


## Configuring the Environment


First, we will configure Firefox to route traffic through Burp Suite. On the AttackBox, open `Firefox`, click the `FoxyProxy` icon (1) and select the `Burp` profile (2) so all browser requests are sent to Burp, as shown below:


 ![Firefox window to select Burp profile.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1763482270396.png)


Next, click on the Burp Suite icon on the `Desktop` to launch the application. 


 ![Ubuntu desktop screen with Burp icon.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1761026130664.png)


You will see an introductory screen; choose `Temporary project in memory` and click `Next`.


 ![Introductory screen in Burp Suite.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1760974022370.png)


On the configuration screen, click `Start Burp` to start the application.


 ![Startup screen in Burp Suite.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1760974022091.png)


Once the application is started, you will see the following screen, where we will use the `Proxy` and `Repeater` option to exploit the race condition.


 ![Welcome screen in Burp Suite.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1763482551368.png)


##  


Before proceeding, ensure that you turn off "`Intercept`" in Burp Suite. Open the `Proxy` tab and check the `Intercept` sub-tab; If the button says "**Intercept on**", click it so it changes to "**Intercept off**". This step ensures that Burp Suite no longer holds your browser requests and allows them to pass through normally.


 ![Turn off intercept option in Burp Suite.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1765978025648.png)


## Making a Legitimate Request


Now that the environment is configured, make a normal request. Open `Firefox`, visit the webapp at `http://MACHINE_IP`, and you will see the following login page:


 ![TBFC login page.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1760974022467.png)


On the site’s login panel, enter the credentials, **username:** `attacker` and **password:** `attacker@123`. After logging in, you’ll arrive at the main dashboard, which shows the limited-edition SleighToy with only 10 units available.


 ![TBFC welcome page.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1760974022385.png)


To make a legitimate purchase, click `Add to Cart` for the SleighToy and then click `Checkout` to go to the checkout page.


 ![TBFC inventory page.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1760974022318.png)


On the checkout page, click `Confirm & Pay` to complete the purchase. You should see a success message confirming the order, as shown below:


 ![TBFC checkout page.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1760970837165.png)


## Exploiting the Race Condition


Now that we have made a legitimate request, navigate back to Burp Suite and click on  `Proxy > HTTP history` and find the **POST** request to the `/process_checkout` endpoint created by our legitimate checkout request. Right-click that entry and choose `Send to Repeater`, which will copy the exact HTTP request (headers, cookies, body) into Burp’s Repeater tool as shown below:


 ![Sending Burp Suite with multiple requests.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1760720808915.png)


Next, switch to the `Repeater` tab and confirm the request appears there, right-click on the first tab, select `Add tab to group`, and click on `Create tab group`.


 ![Burp Suite to create tab group.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1760971316638.png)


Enter a name for the tab group, such as `cart`, and click **Create**, which will create a tab group named `cart`.


 ![Create Tab Group screen in Burp Suite.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1760971316538.png)


Then, right-click the request tab and select `Duplicate tab`. When prompted, enter the number of copies you want (for example, 15). You’ll now have that many identical request tabs inside the cart group.


 ![Duplicate requests in Burp Suite.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1760971316687.png)


 


Next, use the Repeater toolbar `Send` dropdown menu and select `Send group in parallel (last-byte sync)`, which launches all copies at once and waits for the final byte from each response, maximising the timing overlap to trigger race conditions.


 ![Send Parallel requests in Burp Suite.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1760971773327.png)


Once this is done, click `Send group (parallel)`; this will launch all 15 requests to the server simultaneously. The server will attempt to handle them simultaneously, which will cause the timing bug to appear (due to multiple orders being processed at once).


 ![Response after sending successful requests.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1760971773182.png)


Finally, visit the web app, and you will see multiple confirmed orders and the SleighToy stock reduced (possibly going negative). 


 ![Negative stocks and flag value.](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/62a7685ca6e7ce005d3f3afe-1760972351248.png)


## Mitigation


```javascript


The attacker logged in and made a normal purchase of the limited SleighToy. Using Burp Suite, he captured the checkout request and sent it multiple times in parallel. Because the app didn’t handle simultaneous checkouts correctly,
 each request succeeded before the stock could update. This allowed the 
attacker to buy more toys than available, resulting in a race condition 
and pushing the stock into negative values. Here are a few mitigation 
measures to avoid the vulnerability:

Use atomic database transactions so stock deduction and order creation execute as a single, consistent operation.
Perform a final stock validation right before committing the transaction to prevent overselling.
Implement idempotency keys for checkout requests to ensure duplicates aren’t processed multiple times.
Apply rate limiting or concurrency controls to block rapid, repeated checkout attempts from the same user or session.


```

Answer the questions below

What is the flag value once the stocks are negative for **SleighToy Limited Edition**?
THM{WINNER_OF_R@CE007}

Repeat the same steps as were done for ordering the SleighToy Limited Edition. What is the flag value once the stocks are negative for **Bunny Plush (Blue)**?
THM{WINNER_OF_Bunny_R@ce}

Feel free to check out the [Race Conditions](https://tryhackme.com/r/room/raceconditionsattacks) room if you enjoyed this task.
