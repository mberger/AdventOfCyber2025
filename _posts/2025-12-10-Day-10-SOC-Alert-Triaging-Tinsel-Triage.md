---
layout: post
title:  "SOC Alert Triaging - Tinsel Triage"
date:   2025-12-10
categories: welcome
image: Day10-banner.svg
--- 

## During the real time Advent of Cyber for 2025, this day's activities were no available. The answers below are not answered at this time, but if they do open them I will finish.


# SOC Alert Triaging - Tinsel Triage

TryHackMe Room: <https://tryhackme.com/room/azuresentinel-aoc2025-a7d3h9k0p2> Advent of Cyber 2025 – Day 10

### Quick Overview

* **Difficulty**: Medium
* **Type**: Hands-on Microsoft Sentinel SIEM exploration + alert triage CTF
* **Time**: \~45 minutes
* **Goal**: Triage a storm of alerts in Azure Sentinel to separate real threats from noise and uncover the Evil Bunnies' latest intrusion

### The Story in a Nutshell

The Best Festival Company's Azure tenant is getting hammered—alerts are raining down like a digital blizzard, overwhelming the SOC elves. Suspicious activity everywhere, and the Evil Bunnies (looking at you, Sir Carrotbane) are likely behind it. McSkidy needs your help in Microsoft Sentinel to sift through incidents, correlate logs, validate real attacks, and clear the chaos before the bunnies escalate their EASTMAS sabotage.

### What You Actually Do (Task Flow)


1. Deploy the lab → grab Azure credentials and verify logs are flowing into the Syslog_CL table.
2. Re-enable analytics rules to trigger fresh alerts and incidents.
3. Jump into Microsoft Sentinel → head to the Incidents tab and prioritize high-severity ones (e.g., Linux PrivEsc alerts).
4. Triage step-by-step: Check severity, timestamps, MITRE tactics, affected entities, and timelines.
5. Correlate alerts → use the Evidence section and Similar Incidents to connect the dots.
6. Dive deep with KQL queries in the Logs tab to validate suspicious events (filter by host, project messages).
7. Determine verdicts: True positive, benign, or false—and hunt for attacker actions in the raw logs.
8. Answer questions on specifics—like affected hosts, alert counts, or key log messages—to grab the flags.

### Key Commands/Tools You'll Actually Use and Remember

```bash
// Example KQL queries in Sentinel Logs
Syslog_CL 
| where host_s == "app-02" 
| project _timestamp_t, host_s, Message

Syslog_CL 
| where Message contains "sudo" 
| sort by _timestamp_t desc
```


* **Core Tools**: Microsoft Sentinel (Incidents, Analytics rules, Logs blade); Azure Portal for access; KQL (Kusto Query Language) for log hunting.
* **Concepts**: Alert triage process (severity + context + impact); entity mapping; incident correlation; true/false positives.

### Why It’s Awesome for Beginners (to SOC Work)

* Perfect follow-up to Splunk Day—shifts to cloud-native SIEM with real Azure Sentinel interface.
* Structured triage workflow feels like actual SOC duty without drowning you in alerts.
* Hands-on KQL practice that's gentle but powerful—query once, see the attack unfold.
* Festive overload: Alerts popping like tinsel, Evil Bunnies causing havoc—makes the grind feel merry.

Log into Azure, open Sentinel, and start triaging that tinsel storm. By the end, you'll handle real SOC alerts like a seasoned elf, and the North Pole's cloud will be a little safer. Day 10 triaged—bunnies on notice! 🎄


## Task 1 - Introduction


 ![Task banner for day DAY 10](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1761798411436.svg)


The Best Festival Company's Security Operations Center was in chaos. Screens flickered, lights flashed, and the sound of alerts echoed through the room like a digital thunderstorm. The elves rushed between consoles, their faces lit by the glow of red and orange warnings. It was raining alerts, and no one knew where the storm had begun.


Whispers spread through the SOC as tension filled the air. Something strange was happening across the cloud environment, and the timing couldn't be worse. As the blizzard of alerts grew heavier, one name surfaced among the worried elves: the evil Easter Bunnies. But why now? And what were they after this time?

## Learning Objectives


* Understand the importance of alert triage and prioritisation
* Explore Microsoft Sentinel to review and analyse alerts
* Correlate logs to identify real activities and determine alert verdicts
* \

## Connection Details

Before moving forward, review the questions in the connection card shown below:


 ![Connection details for AOC Day 10.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1762875279962.png)


Please follow the instructions below to access your lab for the task.


To get started, click the **Cloud Details** button below.


**In case the Lab does not load and results in an error, due to high volumes on Azure, please wait 30 minutes before retrying.**


On the **Environment** tab, click the **Join Lab** button to deploy your lab environment.


 ![Join lab button.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1762097337275.png)

After joining the lab, select the **Actions** tab. From here, first click the (1) **Deploy** **Lab**. Please **wait 2 - 3 minutes** before proceeding to click the (2) **Deploy Rules** button in order to make sure your environment will be set up correctly.


 ![Action buttons to deploy the lab and rules.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1762537683517.png "right-50")


Then, click the **Copy Lab URL** button to copy the Azure portal URL and open it in a new tab.


 ![Copy URL button for the Azure portal link.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1762537690747.png "left-50")


Next, select the **Credentials** tab to view your login credentials to access the Microsoft Azure portal.


 ![Azure Credentials with Temporary Access Pass.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5efbaebdaaea011c857b438d/room-content/5efbaebdaaea011c857b438d-1762185327373.png "right-50")


Sign in using your **Username** and **Temporary Access Pass (TAP)** from the **Credentials** tab. Then click **Next** to continue. This will take you directly to the Microsoft Azure portal.


 ![Microsoft Sign In page](https://tryhackme-images.s3.amazonaws.com/user-uploads/5f9c7574e201fe31dad228fc/room-content/5f9c7574e201fe31dad228fc-1759243758498.png "left-50")


 ![Login with Temporary Access Pass.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5efbaebdaaea011c857b438d/room-content/5efbaebdaaea011c857b438d-1761753973488.png "right-50")


**Note:** The full deployment process after pressing **Deploy Lab** and **Deploy Rules** takes about 4 - 5 minutes. Make sure to give it enough time. Also, lab access will expire after 1 hour.


On a quick note, the contents of the **Cloud Details** modal will be visible in the topmost part of the room once the lab is deployed. This allows you to easily access all your details and control the lab.


 ![Azure Lab Controls](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1761800447059.png)

Answer the questions below

I have successfully deployed the lab and its rules, and I'm now ready to access my Azure account!


## Task 2 - Alert Triaging Primer

## It's Raining Alerts

McSkidy was notified that it's raining alerts; something unusual is happening within the Azure tenant. The dashboards are lighting up with suspicious activities, and early signs indicate a possible attack from the Evil Bunnies. The Best Festival Company must act fast to survive this onslaught before it affects the entire season's operations.


 ![Depiction of raining alerts, affecting the festivities of the Best Festival Company.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1763630654527.png)


Before investigating these alerts in Microsoft Sentinel, McSkidy must step back and understand what's happening. When alerts start flooding in, jumping straight into each one isn't efficient since not all alerts are equal. Some are noise, others are false positives, and a few may indicate real threats in progress.


This is where alert triaging becomes critical. Triaging helps security teams identify which alerts deserve immediate attention, which can be deprioritised, and which can be safely ignored for a moment. The process separates chaos from clarity, allowing analysts like McSkidy's SOC team to focus their time and resources where it truly matters.


## Alert Triaging


Now, let's continue the discussion about alert triaging. In the previous section, we introduced triaging and why it is essential. This time, we'll focus on how to approach it, what to prioritise, what to look for, and what to do right after an alert.


When multiple alerts appear, analysts should have a consistent method to assess and prioritise them quickly. There are many factors you can consider when triaging, but these are the fundamental ones that should always be part of your process of identifying and evaluating alerts:




| **Key Factors** | **Description** | **Why It Matters?** |
|----|----|----|
| Severity Level | Review the alert's severity rating, ranging from Informational to Critical. | Indicates the urgency of response and potential business risk. |
| Timestamp and Frequency | Identify when the alert was triggered and check for related activity before and after that time. | Helps identify ongoing attacks or patterns of repeated behaviour. |
| Attack Stage | Determine which stage of the attack lifecycle this alert indicates (reconnaissance, persistence, or data exfiltration). | It gives insight into how far the attacker may have progressed and their objective. |
| Affected Asset | Identify the system, user, or resource involved and assess its importance to operations. | Prioritises response based on the asset's importance and the potential impact of compromise. |


In short, these four represent the essential dimensions of triage:


* **Severity:** How bad?
* **Time:** When?
* **Context:** Where in the attack lifecycle?
* **Impact:** Who or what is affected?


They form a balanced foundation that's simple enough for analysts to apply quickly but comprehensive enough for informed decisions.


After reviewing these factors, decide on your next step: escalate to the incident response team, perform a deeper investigation, or close the alert if it's confirmed to be a false positive. A structured triage process like this helps ensure that time and resources are focused on what truly matters.


## Diving Deeper into an Alert


After identifying which alerts deserve further attention, it's time to dig into the details. Follow these steps to investigate and correlate effectively:


* **Investigate the alert in detail.** \n Open the alert and review the entities, event data, and detection logic. Confirm whether the activity represents real malicious behaviour. \n  \n 
* **Check the related logs.** \n Examine the relevant log sources. Look for patterns or unusual actions that align with the alert. \n  \n 
* **Correlate multiple alerts.** \n Identify other alerts involving the same user, IP address, or device. Correlation often reveals a broader attack sequence or coordinated activity. \n  \n 
* **Build context and a timeline.** \n Combine timestamps, user actions, and affected assets to reconstruct the sequence of events. This helps determine if the attack is ongoing or has already been contained. \n  \n 
* **Decide on the following action.** \n If there are indicators of compromise, escalate to the incident response team. Investigate further if more evidence or correlation is needed. Close or suppress if the alert is a confirmed false positive, and update detection rules accordingly. \n  \n 
* \*\*Document findings and lessons learned. \n \*\*Keep a clear record of the analysis, decisions, and remediation steps. Proper documentation strengthens SOC processes and supports continuous improvement.


With the triage complete and the investigation in motion, McSkidy begins piecing together the puzzle. Every alert, log entry, and timestamp brings her closer to uncovering what the Evil Bunnies are up to inside the Azure tenant. It's time to connect the dots and reveal the bigger picture behind the noise.


Answer the questions below

Let's proceed to the action!


## Task 3 - Environment Setup

## Environment Setup

Before proceeding with alert triaging, let’s first set up your lab environment. This step ensures that all simulated alerts are ready within your Azure environment.


To get started, head over to the Azure Portal and search for Microsoft Sentinel.


 ![Accessing Microsoft Sentinel from the Azure Portal.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1761801801838.png)


Then, click your dedicated Sentinel instance, go to the **Logs** tab and select the custom log table named **Syslog_CL**.


 ![Reviewing Sentinel Logs Syslog_CL table.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1765381600047.png)


After running the query, the logs for this lab environment should be rendered.


 ![](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1765381600008.png)


**Note:** It may take a few minutes for the logs to be fully ingested. Kindly wait before proceeding to the next step. You may click the refresh button to see the updated table.


## Preparing the Sentinel Rules


After reviewing the logs, navigate to the **Configuration** tab and select **Analytics,** as shown in the image below.


 ![Rule deployment instructions.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1761801801893.png)


From here, our goal is to re-enable the deployed ruleset and force it to trigger the alerts, thereby producing the incidents. Starting from the **Analytics** tab, select all active rules and click **Disable**. Once the rules are disabled, select them again and click **Enable**.


**Note:** Refresh the page if you encounter the message "This page has been moved to the Defender portal". Keep this in mind whenever you encounter that message.


After doing these steps, you will receive a notification stating that you have successfully re-enabled the rules.


 ![Azure notification for enabling alerts.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1761801893096.png)


Now that we have prepared the environment, let's proceed with the discussion of alert triaging in the next task.


Answer the questions below

I have now checked the logs and enabled the ruleset.


## Task 4 - Investigation Proper

## McSkidy Goes Triaging

Now that we have learned about triaging, let's move to the fun part, working inside the actual SOC environment of the Best Festival Company hosted in Azure. This is where McSkidy will put her triage skills to the test using Microsoft Sentinel, a cloud-native SIEM and SOAR platform. Sentinel collects data from various Azure services, applications, and connected sources to detect, investigate, and respond to threats in real time.


Through Sentinel, McSkidy can view and manage alerts, analyse incidents, and correlate activities across multiple logs. It provides visibility into what's happening within the Azure tenant and efficiently allows analysts to pivot from one alert to another. In this next part, we'll explore how McSkidy reviews alerts, drills into the evidence, and uses Sentinel's investigation tools to uncover the truth behind the Evil Bunnies' attack.


## Microsoft Sentinel in Action


To start the activity, navigate to [Microsoft Sentinel](https://portal.azure.com/#browse/microsoft.securityinsightsarg%2Fsentinel) and select your dedicated Sentinel instance. Then, under the **Threat management** dropdown, select the **Incidents** tab to view the incidents triggered during the current timeframe. You may also press the `<<` button to expand the view as shown in the image below.


 ![Microsoft Sentinel Incidents page.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1761812013900.png)


**Note:** We have initially preloaded the rules from the previous steps. In some instances, the alerts might not have triggered yet. But don't worry, redo the enable/disable rules step to trigger them.


From the task images, there are eight open incidents, four of high severity and four of medium severity. Note that these numbers might differ in your lab environment.


Since we focus on addressing the most critical threats first, we’ll begin with the high-severity alerts. These represent potential compromise points or privilege-escalation activities that could lead to complete host control if left unchecked.


To begin the triage, we’ll examine one high-severity incident in detail: the **Linux PrivEsc—Kernel Module Insertion** alert. By clicking the alert, additional details appear on the right-hand side.


 ![Sentinel Incident: Kernel Module Insertion](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1761813693949.png)


Upon checking the alert, as seen in the above image, the following details can be initially inferred:




1. There are three events related to the alert.
2. The alert was recently created (note that this varies based on your lab instance).
3. There are three entities involved in the alert.
4. The alert is classified as a Privilege Escalation tactic.


We can get further details from here by clicking the **View full details** button.


 ![View full details button](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1762540605044.png)


In the new view, we can see that in addition to the details shown in the summary, we can also view the possible **Incident Timeline** and **Similar Incidents**.


 ![Similar incidents and incident timeline.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1762540510566.png)


### Understanding Related Alerts


From the view above, you may notice that several alerts point to the same affected entities. This helps us understand the relationship and the possible sequence of events that impact the same host or user.


When multiple alerts are linked to a single entity, such as the same **machine**, **user**, or **IP address**, it typically indicates that these detections are not isolated incidents, but somewhat different stages of the same intrusion.


By analysing which alerts share the same entities, we can start to trace the attack path, from the initial access to privilege escalation and persistence.


For example, if the same VM triggered the following alerts:




| **Alert** | **What does it suggest?** |
|----|----|
| Root SSH Login from External IP | The attacker gained remote access (via SSH) to the system (Initial Access) |
| SUID Discovery | The attacker looked for ways to escalate privileges. |
| Kernel Module Insertion | The attacker installed a malicious kernel module for persistence. |


At this stage, McSkidy has reviewed the high-severity alerts, identified the affected entities, and noticed that several detections are linked together. This initial triage allows her to prioritise which incidents need immediate attention and recognise when multiple alerts are actually part of a larger compromise.


With this foundational understanding, McSkidy is ready to move beyond surface-level triage and dive deeper into the underlying logs, which will be discussed in the next task.


Answer the questions below

How many entities are affected by the **Linux PrivEsc - Polkit Exploit Attempt** alert?


What is the severity of the **Linux PrivEsc - Sudo Shadow Access** alert?


How many accounts were added to the sudoers group in the **Linux PrivEsc - User Added to Sudo Group** alert?



## Task 5 - Diving Deeper Into Logs

## In-Depth Log Analysis with Sentinel

With the initial triage complete, McSkidy now examines the raw evidence behind these alerts. The next task involves diving into the underlying log data within Microsoft Sentinel to validate the alerts and uncover the exact attacker actions that triggered them. By analysing authentication attempts, command executions, and system changes, McSkidy can begin piecing together the full story of how the attack unfolded.

If we go back to the alert's full details view, we can try clicking the **Events** from the **Evidence** section.

 ![Event Evidences from a single alert.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1761814527567.png)

From this view, we can definitely see the actual name of the kernel module installed in each machine and the time it was installed.

Diving deeper into this, we can try checking the raw events from a single host through a custom query. To do this, let's change the view into an editable KQL query and find all the events triggered from **app-02**.


1. Press the **Simple mode** dropdown from the upper-right corner and select KQL mode.
2. Modify the query with the following KQL query below. \n  \n `set query_now = datetime(2025-10-30T05:09:25.9886229Z);` \n `Syslog_CL` \n `| where host_s == 'app-02'` \n `| project _timestamp_t, host_s, Message` \n  \n 
3. Press the **Run** button and wait for the results.

 ![KQL query mode.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1761814527687.png)

After executing the query, it can be seen that multiple potentially suspicious events occurred around the installation of the kernel module.


1. Execution of the `cp` (copy) command to create a shadow file backup.
2. Addition of the user account Alice to the sudoers group.
3. Modification of the backupuser account by root.
4. Insertion of the malicious_mod.ko module.
5. Successful SSH authentication by the root user.

 ![System logs from the app-02 machine.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1761815790577.png)
Based on the surrounding events, including the execution of the `cp` command to create a shadow file backup, the addition of the user account Alice to the sudoers group, the modification of the backupuser account by root, and the successful SSH authentication by the root user, this activity appears highly unusual. The sequence suggests potential privilege escalation and persistence behaviour, indicating that the event may not be part of normal system operations and warrants further investigation.

Now that we have discussed the methodology for determining and reviewing alerts, let’s help McSkidy complete the assessment by examining the remaining alerts and answering the questions below.

 ![Investigating alerts with McSkidy.](https://tryhackme-images.s3.amazonaws.com/user-uploads/5dbea226085ab6182a2ee0f7/room-content/5dbea226085ab6182a2ee0f7-1763630714346.png)

Answer the questions below

What is the name of the kernel module installed in websrv-01?



What is the unusual command executed within websrv-01 by the ops user?



What is the source IP address of the first successful SSH login to storage-01?



What is the external source IP that successfully logged in as root to app-01?



Aside from the backup user, what is the name of the user added to the sudoers group inside app-01?



