---
layout: post
title:  "Day 17 -CyberChef - Hoperation Save McSkidy
date:   2025-12-17
categories: welcome
image: Day17-banner.png
---

# CyberChef - Hoperation Save McSkidy

TryHackMe Room: [https://tryhackme.com/room/encoding-decoding-aoc2025-s1a4z7x0c3](https://tryhackme.com/room/encoding-decoding-aoc2025-s1a4z7x0c3?referrer=grok.com) Advent of Cyber 2025 – Day 17

### Quick Overview

* **Difficulty**: Medium
* **Type**: Encoding/decoding puzzle chain + CyberChef mastery CTF
* **Time**: \~45–60 minutes
* **Goal**: Crack five progressively trickier "locks" on the Quantum Fortress by encoding/decoding messages to/from guards and reverse-engineering login logic—all with CyberChef

### The Story in a Nutshell

The elves are launching a full rescue op for McSkidy, who's locked deep in King Malhare's Quantum Warren behind five digital gates guarded by Sir BreachBlocker III's minions. McSkidy's earlier bunny pics gave the clue: chat with the guards (they only understand encoded speech), extract credentials, reverse the exact "encryption" recipes applied server-side, and log in to each gate. Success disables the locks and springs McSkidy just as TBFC teeters on EASTMAS disaster.

### What You Actually Do (Task Flow)


1. Deploy the VM → head to http://MACHINE_IP:8080 and face the first gate's chat/login panel.
2. **Lock 1 – Outer Gate**: Base64-encode the guard name as username, inspect headers for the "magic question," encode and ask it, get Base64 password → decode once.
3. **Lock 2 – Outer Wall**: Double Base64 on everything—encode your asks twice, decode the response twice.
4. **Lock 3 – Guard House**: XOR chaos—extract key, chat for the XOR'd-then-Base64'd password, reverse by XORing twice with the same key.
5. **Lock 4 – Inner Castle**: Straight MD5 hash—grab the hash from the guard, hit CrackStation or similar lookup to get plaintext.
6. **Lock 5 – Prison Tower**: The boss level—inspect Debugger tab or headers for recipe ID, match to cheat sheet (Base64 → Reverse → ROT13, etc.), build the exact reverse recipe in CyberChef, apply to the final encoded password, log in, and free McSkidy (plus grab the flag).

### Key Concepts/Tools You'll Actually Use and Remember

* **Core Ideas**: Encoding (reversible, no key) vs. Encryption (key-based); chaining operations; reversible transforms (Base64, XOR, ROT13, Reverse, Hex).
* **Star Tool**: CyberChef — drag operations into recipe, watch Magic wand suggest next steps, use Input/Output panes like a pro.
* **Operations You'll Chain**:
  * To/From Base64 (multiple times)
  * XOR (with key, brute if needed)
  * ROT13 / ROT47
  * Reverse String
  * From Hex
  * MD5 (and online hash crackers)
* **Browser Tricks**: DevTools Network/Headers tab for hidden hints; Debugger to pause on login and inspect JS logic; console for quick tests.

### Why It’s Awesome for Beginners (to Data Transforms)

* CyberChef is pure wizardry—drop ops, see instant results, no coding required.
* Escalates perfectly: starts with simple Base64, ends with multi-step custom recipes you'll build yourself.
* That "I reversed the exact chain!" dopamine hit on each lock is addictive.
* Festive flair: Guards chat in encoded bunny-speak, delays on responses, and the final prison break feels epic.

Boot the machine, open CyberChef in another tab, and start chatting with those encoded guards. By the end, you'll wield CyberChef like a decoding superhero, and McSkidy will be hopping free—mission accomplished! Day 17 decoded—Hoperation success! 🎄


## Task 1 - Introduction


 ![Task banner for day 17](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763533892160.png)


*McSkidy is imprisoned in King Malhare's Quantum Warren. Sir BreachBlocker III was put in charge of securing the fortress and implemented several access controls to prevent any escape. His defenses are worthy of his name.*


*However, McSkidy managed to send vital clues to his team using harmless bunny pictures. One message revealed that five locks needed to be disabled to secure an escape route. The locks can be broken by examining their logic and leveraging the system's built-in chat for the guards. They can be eluded in revealing vital details or even passwords. However, you will need to speak their language.*


## Learning Objectives


* Introduction to encoding/decoding
* Learn how to use CyberChef
* Identify useful information in web applications through HTTP headers


## Connecting to the Machine


 ![Connection Card - Attack Box and Start VM](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763965652654.png)


Start your target machine by clicking the **Start Machine** button below. The machine will need about 2 minutes to fully boot. Additionally, start your AttackBox by clicking the **Start AttackBox** button below. The AttackBox will start in split view. In case you can not see it, click the **Show Split View** button at the top of the page.


### Set up your virtual environment

To successfully complete this room, you'll need to set up your virtual environment. This involves starting both your AttackBox (if you're not using your VPN) and Target Machines, ensuring you're equipped with the necessary tools and access to tackle the challenges ahead.


Answer the questions below

Let us siege the fortress!


## Task 2 - Important Concepts

## Encoding and Decoding


Encoding is a method to transform data to ensure compatibility between different systems. It differs from encryption in purpose and process.



|   | Encoding | Encryption |
|----|----|----|
| **Purpose** | Compatibility \n Usability | Security \n Confidentiality |
| **Process** | Standardized | Algorithm + Key |
| **Security** | No | Yes |
| **Speed** | Fast | Slow |
| **Examples** | Base64 | TLS |


Decoding is the process of converting encoded data back to its original, readable, and usable form.


## CyberChef Overview


[CyberChef](https://cyberchef.io/) is also known as the Cyber Swiss Army Knife. Ready to cook some recipes?



| Area | Description |
|----|----|
| Operations | Repository of diverse CyberChef capabilities |
| Recipe | Fine-tune and chain the operations area |
| Input | Here you provide the input for your recipe |
| Output | Here is the output of your recipe |


## Simple Example


Try your first recipe:


* Open either the online [CyberChef](https://cyberchef.io/) version in your regular browser, or use the offline CyberChef version available in the bookmarks section of the AttackBox. Drag and drop the `To Base64` operation from the **Operations** area on the left side to the **Recipe** area in the center, and add `IamRoot` into the **Input** area.


* Add another operation, `From Base64`, to show the initial input again, showcasing chain operations.


**Note:** You can enable/disable an operation in the recipe by toggling the middle button on the right of the operation.


 ![Cyberchef simple example of how to encode an input in Base64.](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1762941123967.png)


Congratulations! You took the first steps to become a master Chef.


## Inspecting Web Pages


Besides the rendered content of a web page, your browser usually receives and can show additional information.


For this challenge, you will get the chance to have a deeper look at that information and put it to good use.


To do this, depending on your browser, you can access the functionality as shown below:



| Browser | Menu path |
|----|----|
| Chrome | `More tools` > `Developer tools` |
| Firefox | `Menu` (☰) > `More tools` > `Web Developer Tools` |
| Microsoft Edge | `Settings and more (...)` > `More tools` > `Developer tools` |
| Opera | `Developer` > `Developer tools` |
| Safari | `Develop` > `Show Web Inspector` (Requires enabling the "Develop" menu in `Preferences` > `Advanced`) |


> **Note:** For a better experience, you can reposition the console on the right side of the browser. Look for the three dots on the right side of the console.
>
>  ![Docking Firefox console to the Right](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1762941124118.png)

Answer the questions below

Locked and loaded.


## Task 3 - First Lock - Outer Gate

## Key Information


If not already, start the target machine, give it a few minutes to boot up, and then, from the AttackBox, you can access the web app at `http://MACHINE_IP:8080`.


McSkidy revealed some vital clues in his message. You will have to leverage any useful piece of information in order to break the locks.


Below are key points to look out for:


* **Chat is Base64 encoded**. Try decoding this in CyberChef. This will be leveraged to extract useful information from the guards. Be aware that from Lock 3 onwards, the guards will take a longer time to respond.


 ![Example of encoded Bunnygram chat](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1761561410887.png)


* **Guard name**. This logic will persist throughout the levels. Make sure to note down the guard’s name for each level.


 ![Example of hint in the login form](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1761136162115.png)


* **Headers**. Again, inspecting the page but switching to the ‘*Network*’ tab this time. Make sure to refresh the page once after switching to this tab and select the first response.


 ![Example of finding header information](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1764069898747.png)


* **Login Logic**. You will inspect the page and switch to the ‘*Debugger*’ tab. Match the lock with the respective logic. You can also find helpful comments that explain what you need to cook in CyberChef.


 ![Example of finding login logic](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1761136162547.png)


##  ![Image with the outer gate.](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763631263323.png)First Lock - Outer Gate


Ok, it’s time to siege the fortress. Ready?


     1. First, identify the guard name and encode it to Base64. You will use this as the username input.


     2. Next, using the information from the page headers, identify the magic question and encode it in Base64 as well.


 ![Shows first lock magic question: "What is the password for this level?"](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763622327932.png)


     3. Use the encode magic question in the chat. The guard will answer with the encoded level password.


     4. Now, switch to the ‘*Debugger*’ tab and identify the login logic. In this case, the password is encoded to Base 64.


 ![Shows first lock login logic: simple Base64 encoding](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763622523221.png)


     5. By decoding the answer from the guard, you will have the plaintext password.


     6. Use the encoded username and plaintext password to log in.


Excellent work! One lock is down, and only four remain to be broken.

Answer the questions below

What is the password for the first lock?


Task 4 - Second Lock - Outer Wall

##  ![Image with the outer wall](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763631294782.png) Second Lock - Outer Wall


Excellent job breaking that first level.


This level nudges the difficulty up a little bit, but don’t worry, you will figure it out. Let’s go!


     1. Again, identify the guard's name and save the encoded output for later.


     2. Then, extract and encode the magic question and retrieve the encoded password from the guard.


 ![Shows the second lock magic question: "Did you change the password?"](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763623056230.png)


     3. Looking again at the login logic, you see that the encoding is applied twice this time. That means you have to decode from Base64 twice.


     4. Go ahead and log in with the newfound password and the saved username.


 ![Shows the second lock login logic: double Base64 encoding](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763623056841.png)


You are getting closer to securing an escape route; only three locks remain. Keep up the good work.

Answer the questions below

What is the password for the second lock?


Task 5 - Third Lock - Guard House

##  ![Image with the guard house](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763631323714.png) Third Lock - Guard House


So far, so good. As you saw in the previous level, the login logic begins to use chained operations.


This will be the trend for this and the following levels.



1. As always, collect all the needed information (encoded username, encoded password from the guard, XOR key).


 ![Shows the third lock XOR key: "cyberchef"](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763623688692.png)


**Note:** From this lock onwards, there is no magic question, but sometimes you can ask the guard nicely to give you the password. It will still need to be decoded as per the login logic. Be aware that the guard may sometimes fall asleep or take a long time to respond (\~2-3 minutes) so keeping the message short will help get the answer. Even a simple 'Password please.' will go a long way.


     2. If you look at the login logic, there is a slight twist. The password is first XOR’ed with a key and then encoded to Base64.


## Theory Time


> XOR is a popular operation that, besides the input data, also uses a key. The process involves a bitwise exclusive OR between the data and key.
>
>  ![Shows the XOR logic diagram](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763638274313.png)
>
> You might ask, “*Ok, but how do I reverse this?*”. Well, skipping the long math explanation, XOR has a magic property: when you XOR the result with the key again, the new result will be the initial data. Go ahead, try this in CyberChef. Put two XOR operations one after another, use the same key for both, and the output should be identical.


 ![Shows that double XOR-ing an input with the same key provides the same output.](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763536921553.png)


     3. With this newfound knowledge, build the needed recipe to find the plaintext password.


 ![Shows the third lock login logic: XOR with key and then encode Base64](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763625972684.png)


 ![Shows the recipe for reversing the encoding: From64 and XOR with key](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1764064441038.png)


    4. Use the credentials and unlock the next level.

Answer the questions below

What is the password for the third lock?


## Task 6 - Fourth Lock - Inner Castle

##  ![Image of the inner castle](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763632423450.png) Fourth Lock - Inner Castle


We are almost there. In this level, Sir BreachBlocker III throws you a curveball. Let’s see how to tackle this.


     1. But first, go ahead and look at the login logic as before. We will not be needing header information for this one.


 ![Shows the fourth lock login logic: MD5 hash](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763626852157.png)


     2. After asking the guard for the password and looking at it's reply, it seems a bit odd. At the same time, the login logic shows the use of a MD5 hash.


 ![Shows that the decoded guard answer reveals a hash string](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763626851107.png)


## Theory Time


> MD5, or Message-Digest Algorithm 5, is a cryptographic algorithm that produces a fixed-size hash value. While this is supposed to be a one-way function, meaning you cannot reverse it, precomputed hashes can be leveraged to identify the input.


     3. Putting the two together, the plaintext password is passed through MD5, and you have the hash. This looks like a job for [CrackStation](https://crackstation.net/).


     4. Go ahead and open the site and paste the hash to retrieve the password.


 ![Shows Crackstation with the hash input and decoded password](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763626852611.png)


     5. Use the credentials and advance to the final level.


Fantastic. One more lock and you will ensure McSkidy has safe passage and escapes.

Answer the questions below

What is the password for the fourth lock?


## Task 7 - Fifth Lock - Prison Tower

##  ![Image with a medieval tower](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763631352514.png) Fifth Lock - Prison Tower


Ready for the final hurdle?


As the defenses weaken, you receive another hidden message from McSkidy:


*“I can see you are ready to break the last lock. Be aware that Sir BreachBlocker III implemented different mechanisms for the last lock, which change occasionally. Make sure you match the correct approach when decoding the password.”*


That sounds tricky, but do not despair. You will find a way.


     1. Let’s start. Extract the information as before, noting down the encoded guard name.


 ![Shows the fifth lock login logic hint: to look at the recipe number](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763627625991.png)
https://files.slack.com/files-pri/T037ZR4HR-F0A4CV3JUDB/image.png
     2. Additionally, note the recipe ID from the header and match the corresponding login logic. Below is a quick cheat sheet for decoding each recipe.



| Recipe ID | Reverse Logic |
|----|----|
| 1 | From Base64 ⇒ Reverse ⇒ ROT13 |
| 2 | From Base64 ⇒ From Hex ⇒ Reverse |
| 3 | ROT13 ⇒ From Base64 ⇒ XOR(extracted key) |
| 4 | ROT13 ⇒ From Base64 ⇒ ROT47 |


     3. Build the reverse recipe with CyberChef and extract the final password.


Finally, the last lock has been breached, and you provided a safe path for McSkidy to escape.

Answer the questions below

What is the password for the fifth lock?
51i51rBr34chBl0ck3r

What is the retrieved flag?
THM{M3D13V4L_D3C0D3R_4D3P7}

## Task 8 - Epiloge

 ![Shows McSkidy escaping from the castle cased by bunny guards](https://tryhackme-images.s3.amazonaws.com/user-uploads/68baea2454c82afe90fd7020/room-content/68baea2454c82afe90fd7020-1763536556206.png)

i51rBr34chBl0ck3r
*As McSkidy passed by the Inner Castle, she heard a thunderous voice: “Why should Christmas have all the fun?”*

*McSkidy managed to get back to Wareville just in time as TBFC was about to be hit by another disaster.*

Answer the questions below

If you found decoding secrets interesting, you can also check out the [Introduction to Cryptography](https://tryhackme.com/room/cryptographyintro), which dives into the world of cryptography.



Looking for the key to **Side Quest 3**? Hopper has left us this [cyberchef link](https://gchq.github.io/CyberChef/#recipe=To_Base64('A-Za-z0-9%2B/%3D')Label('encoder1')ROT13(true,true,false,7)Split('H0','H0%5C%5Cn')Jump('encoder1',8)Fork('%5C%5Cn','%5C%5Cn',false)Zlib_Deflate('Dynamic%20Huffman%20Coding')XOR(%7B'option':'UTF8','string':'h0pp3r'%7D,'Standard',false)To_Base32('A-Z2-7%3D')Merge(true)Generate_Image('Greyscale',1,512)&input=SG9wcGVyIG1hbmFnZWQgdG8gdXNlIEN5YmVyQ2hlZiB0byBzY3JhbWJsZSB0aGUgZWFzdGVyIGVnZyBrZXkgaW1hZ2UuIEhlIHVzZWQgdGhpcyB2ZXJ5IHJlY2lwZSB0byBkbyBpdC4gVGhlIHNjcmFtYmxlZCB2ZXJzaW9uIG9mIHRoZSBlZ2cgY2FuIGJlIGRvd25sb2FkZWQgZnJvbTogCgpodHRwczovL3RyeWhhY2ttZS1pbWFnZXMuczMuYW1hem9uYXdzLmNvbS91c2VyLXVwbG9hZHMvNWVkNTk2MWM2Mjc2ZGY1Njg4OTFjM2VhL3Jvb20tY29udGVudC81ZWQ1OTYxYzYyNzZkZjU2ODg5MWMzZWEtMTc2NTk1NTA3NTkyMC5wbmcKClJldmVyc2UgdGhlIGFsZ29yaXRobSB0byBnZXQgaXQgYmFjayE) as a lead. See if you can recover the key and access the corresponding challenge in our [Side Quest Hub](https://tryhackme.com/adventofcyber25/sidequest)!


