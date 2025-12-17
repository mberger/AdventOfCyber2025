---
layout: post
title:  "Day 16 - Forensics - Registry Furensics"
date:   2025-12-16
categories: welcome
image: Day16-banner.png
---

# Registry Forensics - Reg the Halls

TryHackMe Room: https://tryhackme.com/room/registry-forensics-aoc2025-h6k9j2l5p8Advent of Cyber 2025 – Day 16

### Quick Overview

* **Difficulty**: Medium
* **Type**: Windows Registry forensics investigation CTF
* **Time**: \~50–65 minutes
* **Goal**: Dive into the Windows Registry hives from a compromised elf workstation to uncover persistence mechanisms, user activity, and Sir Carrotbane's backdoor traces

### The Story in a Nutshell

An elf developer machine has gone rogue—it's quietly phoning home to EASTMAS C2 and running suspicious processes at startup. Splunk caught the outbound traffic, but the malware's hiding in the Registry like a bad gift under the tree. McSkidy's rescue depends on solid forensics: you get a memory dump or exported hives from the infected Windows box. Time to dissect NTUSER.DAT, SOFTWARE, SYSTEM, and Amcache.hve to expose Carrotbane's autostart tricks, executed commands, and shimmy shenanigans before he registries the entire North Pole network.

### What You Actually Do (Task Flow)


1. Deploy the VM → download the provided Registry hives or mount the forensic image.
2. **Registry 101**: Learn the big five hives (SYSTEM, SOFTWARE, SAM, SECURITY, NTUSER.DAT) and where persistence loves to hide.
3. **Run Key Hunt**: Parse SOFTWARE and SYSTEM for classic Run/RunOnce keys with RegRipper or manual reg queries.
4. **UserAssist Magic**: Decode the ROT13-obfuscated UserAssist entries in NTUSER.DAT to reveal recently executed programs (hello, bunny_dropper.exe).
5. **Shimcache Sleuth**: Extract AppCompatCache (Shimcache) from SYSTEM to list every executable ever run—spot the anomalies.
6. **Amcache & Prefetch**: Dig into Amcache.hve for program execution timestamps and Prefetch for superfetch traces.
7. **Persistence Roundup**: Find scheduled tasks, services, or LNK shortcuts buried in registry values.
8. **Timeline Build**: Correlate timestamps across hives to reconstruct the infection chain.
9. Answer the flags: suspicious Run key path, decoded UserAssist exe, Shimcache oddity, final backdoor name.

### Key Commands/Tools You'll Actually Use and Remember

```bash
regripper/rip.pl -r NTUSER.DAT -p userassist   # decode executed GUIs
regripper/rip.pl -r SYSTEM -p shimcache        # AppCompatCache parser
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg save HKLM\SYSTEM system.hive
```

* **Tools**: RegRipper (the star), Registry Explorer, Autopsy (for mounting images), reg.exe (built-in queries), or offline registry viewers.
* **Core Concepts**: Autostart locations (Run keys, Services, AppInit_DLLs); execution artifacts (UserAssist, Shimcache, Amcache, BAM); ROT13 decoding; timeline analysis.

### Why It’s Awesome for Beginners (to Windows Forensics)

* Registry seems scary until you run one RegRipper plugin and watch the secrets spill out—instant "wow" factor.
* Builds perfectly on previous forensics days (logs, malware, web attacks) but shifts to the Windows persistence playground.
* No live compromise needed—just static hive analysis, safe and repeatable.
* Festive forensics: Keys named "EastmasUpdater" or values pointing to "carrot.exe"—makes hunting persistence feel like naughty-list auditing.

Spin up the VM, fire up RegRipper, and start regging those halls. By the end, you'll read Windows Registry artifacts like holiday letters, and Carrotbane's persistence will be ripped right out. Day 16 registered—backdoors banished! 🎄


## Task 1 - Introduction

 ![Task banner for day 16](https://tryhackme-images.s3.amazonaws.com/user-uploads/6645aa8c024f7893371eb7ac/room-content/6645aa8c024f7893371eb7ac-1763317996813.svg)


TBFC is under attack. Systems are exhibiting weird behavior, and the company is now feeling the absence of its lead defender, McSkidy. However, McSkidy made sure the legacy continues.


McSkidy’s team, determined and well-trained, is fully confident in securing all the systems and regaining control before the big event, SOCMAS.


They have now decided to conduct a detailed forensic analysis on one of the most critical systems of TBFC, `dispatch-srv01`. The `dispatch-srv01` coordinates the drone-based gifts delivery during SOCMAS. However, recently it was compromised by King Malhare’s bandits of bunnies.


TBFC’s defenders have decided to split into specialized teams to uncover the attack on this system through detailed forensics. While some of the other team members investigate logs, memory dumps, file systems, and other artefacts, you will work to investigate the registry of this compromised system.


 ![](https://tryhackme-images.s3.amazonaws.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1763737444784.png)


## Learning Objectives


* Understand what the Windows Registry is and what it contains.
* Dive deep into Registry Hives and Root Keys.
* Analyze Registry Hives through the built-in Registry Editor tool.
* Learn Registry Forensics and investigate through the Registry Explorer tool.


## Connecting to the Machine


Before moving forward, review the questions in the connection card shown below:


 ![](https://tryhackme-images.s3.amazonaws.com/user-uploads/6645aa8c024f7893371eb7ac/room-content/6645aa8c024f7893371eb7ac-1761551274374.png)


Start your target machine by clicking the **Start Machine** button below. The machine will open in split view and will need about 2 minutes to fully boot. In case you can not see it, click the **Show Split View** button at the top of the page.


### Set up your virtual environment

To successfully complete this room, you'll need to set up your virtual environment. This involves starting the Target Machine, ensuring you're equipped with the necessary tools and access to tackle the challenges ahead.



Answer the questions below

I successfully have access to my target machine!


## Task 2 - Investigate the Gifts Delivery Malfunctioning

## Windows Registry


Your brain stores all the information that you need to function effectively. This includes:


* How should you behave?
* What would be the first thing you would do after waking up?
* How would you dress yourself?
* What are your habits?
* What happened in the recent past?


These are just a few things. Your brain knows pretty much everything about you. It's just like a database storing the human configuration.


Windows OS is not a human, but it also needs a brain to store all its configurations. This brain is known as the **Windows Registry**. The registry contains all the information that the Windows OS needs for its functioning. 


Now, this Windows brain (Registry) is not stored in one single place, unlike a human brain, which is situated in one single place inside the head. It is made up of several separate files, each storing information on different configuration settings. These files are known as **Hives**.


 ![](https://tryhackme-images.s3.amazonaws.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1763737444859.png)


Let's take a look at all these hives in the table below. The first column contains the hive names, the second column contains the type of configuration settings that each hive stores, and the third column contains the location of each hive on the disk. 



| Hive Name | Contains | Location |
|----|----|----|
| SYSTEM | Services Mounted Devices Boot Configuration Drivers Hardware | `C:\Windows\System32\config\SYSTEM` |
| SECURITY | Local Security Policies Audit Policy Settings | `C:\Windows\System32\config\SECURITY` |
| SOFTWARE | Installed Programs OS Version and other info Autostarts Program Settings | `C:\Windows\System32\config\SOFTWARE` |
| SAM | Usernames and their Metadata Password Hashes Group Memberships Account Statuses | `C:\Windows\System32\config\SAM` |
| NTUSER.DAT | Recent Files User Preferences User-specific Autostarts | `C:\Users\username\NTUSER.DAT` |
| USRCLASS.DAT | Shellbags Jump Lists | `C:\Users\username\AppData\Local\Microsoft\Windows\USRCLASS.DAT` |


**Note:** The configuration settings stored in each hive listed above are just a few examples. Each hive stores more than these.


Now that you understand where these Registry Hives are stored, you may be tempted to double-click and open these files from their respective locations to view the data they contain. But here's the twist. These Registry Hives contain binary data that cannot be opened directly from the file. So, double-clicking them would only display things you won't ever understand. Then how can we possibly view the registry data?


The Windows OS has a built-in tool known as the **Registry Editor**, which allows you to view all the registry data available in these hives. You can go ahead and open this tool by simply typing Registry Editor in your search bar.


 ![](https://tryhackme-images.s3.amazonaws.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1763584779702.png)


As you can see in the Registry Editor screenshot above, there are some folders named `HKEY_LOCAL_MACHINE`, `HKEY_CURRENT_USER`, and more. But didn't we expect `SYSTEM`, `SECURITY`, `SOFTWARE`, etc., to be seen so we can view their data? Don't worry, Windows organizes all the Registry Hives into these structured **Root Keys**. Instead of seeing the Registry Hives, you would always get these registry oot keys whenever you open the registry. Now, which registry key contains which registry hive's data? To answer this question, we have a table below that maps the registry keys with their respective Registry Hives.



| Hive on Disk | Where You See It in Registry Editor |
|----|----|
| SYSTEM | `HKEY_LOCAL_MACHINE\SYSTEM` |
| SECURITY | `HKEY_LOCAL_MACHINE\SECURITY` |
| SOFTWARE | `HKEY_LOCAL_MACHINE\SOFTWARE` |
| SAM | `HKEY_LOCAL_MACHINE\SAM` |
| NTUSER.DAT | `HKEY_USERS\<SID> and HKEY_CURRENT_USER` |
| USRCLASS.DAT | `HKEY_USERS\<SID>\Software\Classes` |


In the table above, you can see that most of the Registry Hives are located under the `HKEY_LOCAL_MACHINE (HKLM)` key. We can verify this by clicking on the little toggle arrow at the on the left side of the `HKLM` key in the Registry Editor, as shown in the screenshot below:


 ![](https://tryhackme-images.s3.amazonaws.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1763584206508.png)


As you can see, the `SYSTEM`, `SOFTWARE`, `SECURITY`, and `SAM` hives are under the `HKLM` key. `NTUSER.DAT` and `USRCLASS.DAT` are located under `HKEY_USERS (HKU)` and `HKEY_CURRENT_USER (HKCU)`. 


**Note:** The other two keys (`HKEY_CLASSES_ROOT (HKCR)` and `HKEY_CURRENT_CONFIG (HKCC)`) are not part of any separate hive files. They are dynamically populated when Windows is running.


So far, we have learned what the registry is, where it is located (in separate Registry Hives), and how to view the registry through the Registry Editor, which displays the registry keys backed by these Registry Hives. 


Now, let's see how we can actually extract information from the registry. Think of it like navigating to different files in the file explorer. If you know the path where the file is stored, you can go directly to that location to find the file. Let's take a look at a few examples:


## Example 1: View Connected USB Devices


**Note:** The registry key contents explained in this example are not available in the attached VM.


The registry stores information on the USB devices that have been connected to the system. This information is present in the `SYSTEM` hive. To view it:



1. Open the Registry Editor.
2. Navigate to the following path: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`.
3. Here you will see the USB devices' information (make, model, and device ID).
4. Each device will have the following:

   
   1. A main subkey that is the identification of the type and manufacturer of the USB device.
   2. A subkey under the above (for example) that represents the unique devices under this model.


 ![Registry Editor screenshot showing the resgitry key that has the information of the USBs that have been attached.](https://tryhackme-images.s3.amazonaws.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1761292172298.png)


## Example 2: View Programs Run by the User


The registry stores information on the programs that the user ran using the Run dialog `Win + R`. This information is present in the `NTUSER.DAT` hive. To view it:



1. Open the Registry Editor.
2. Navigate to the following path: `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU`.
3. Here you will see the list of commands typed by the user in the Run dialog to run applications.


 ![Registry Editor screenshot showing the resgitry key that has the information of the commands typed by the user in the Run Dialog.](https://tryhackme-images.s3.amazonaws.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1761292172317.png)


## Registry Forensics


Since the registry contains a wide range of data about the Windows system, it plays a crucial role in forensic investigations. Registry forensics is the process of extracting and analyzing evidence from the registry. In Windows digital forensic investigations, investigators analyze registry, event logs, file system data, memory data, and other relevant data to construct the whole incident timeline. 


The table below lists some registry keys that are particularly useful during forensic investigations.



| Registry Key | Importance |
|----|----|
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist` | It stores information on recently accessed applications launched via the GUI. |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths` | It stores all the paths and locations typed by the user inside the Explorer address bar. |
| `HKLM\Software\Microsoft\Windows\CurrentVersion\App Paths` | It stores the path of the applications. |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\WordWheelQuery` | It stores all the search terms typed by the user in the Explorer search bar. |
| `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` | It stores information on the programs that are set to automatically start (startup programs) when the users logs in. |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` | It stores information on the files that the user has recently accessed. |
| `HKLM\SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName` | It stores the computer's name (hostname). |
| `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall` | It stores information on the installed programs. |


Numerous other registry keys can be used for extracting important evidence from a Windows system during an incident investigation. The investigation of these registry keys during forensics cannot be done via the built-in Registry Editor tool. It is because the Registry analysis cannot be done on the system under investigation (due to the chance of modification), so we collect the Registry Hives and open them offline into our forensic workstation. However, the Registry Editor does not allow opening offline hives. The Register editor also displays some of the key values in binary which are not readable.


To solve this problem, there are some tools built for registry forensics. In this task you will use the **[Registry Explorer](https://ericzimmerman.github.io/)** tool which is a registry forensics tool. It is open source and can parse the binary data out of the registry, and we can analyze it without the fear of modification.


## Practical


In this practical example, we will use the Registry Explorer tool to analyze the Registry Hives from the compromised system, `dispatch-srv01`. The Registry Hives have been collected and are available in the folder `C:\Users\Administrator\Desktop\Registry Hives` on the machine attached to this task.


**Step 1: Launch Registry Explorer**


Click on the Registry Explorer icon pinned to the taskbar of the target machine to launch it.


** ![Windows Desktop screenshot highlighting the location of Registry Explorer tool in the taskbar.](https://tryhackme-images.s3.amazonaws.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1761291226158.png)**


**Step 2: Load the Registry Hives**


Once Registry Explorer opens with an empty interface, follow these steps to load the hives:



1. Click the **File** option from the top menu
2. Select **Load hive** from the dropdown


 ![Registry Explorer screenshot with the option of loading offline hives highlighted. ](https://tryhackme-images.s3.amazonaws.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1761291226447.png)


**Step 3: Handling Dirty Hives**


While loading Registry Hives, it is important to know that these Registry Hives can sometimes be "dirty" when collected from live systems, meaning they may have incomplete transactions. To ensure clean loading:


1\.On the **Load hives** pop-up, navigate to `C:\Users\Administrator\Desktop\Registry Hives` \n 2. Select the desired hive file (e.g., SYSTEM) \n **3. Hold SHIFT**, then press **Open** to load associated transaction log files. This ensures you get a clean, consistent hive state for analysis.


 ![Screenshot showing the process of loading offline registry hives in the registry explorer tool.](https://tryhackme-images.s3.amazonaws.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1761291226324.png)


4\. You'll be prompted with a message indicating successful replay for transaction logs \n 5. Repeat the same process for all the other hives you want to load


 ![Registry Explorer screenshot showing the "transcation logs replayed" prompt.](https://tryhackme-images.s3.amazonaws.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1761291226402.png)


**Step 4: Investigating Registry Keys**


After loading the `SYSTEM` hive, you can navigate to specific registry keys for investigation. Let's practice by finding the computer name:


* Navigate to: `ROOT\ControlSet001\Control\ComputerName\ComputerName`. Or you can also just type "ComputerName" in the search bar to quickly locate the key, as shown below.


 ![Registry Editor screenshot showing the navigation to the resgitry key that has the information of the computer's name.](https://tryhackme-images.s3.amazonaws.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1761291226611.png)


* Alternatively, you can click the **Available Bookmarks** tab and navigate to the **ComputerName** key from there.
* Examine the values to identify the system's hostname. Under the **Data** value, you'll find `DISPATCH-SRV01`.


 ![Registry Editor screenshot showing the resgitry key that has the information of the computer's name.](https://tryhackme-images.s3.amazonaws.com/user-uploads/68d2c1e7ab94268f6271de1d/room-content/68d2c1e7ab94268f6271de1d-1761291226675.png)


Now that you understand how to load hives and navigate in Registry Explorer, you're ready to begin your forensic investigation and uncover evidence of the TBFC intrusion on the Dispatch server!


**Note:** The abnormal activity on the `dispatch-srv01` started on 21st October, 2025.


**Tip:** The table given in the Registry Forensics explanation is going to be your friend.

Answer the questions below

What application was installed on the `dispatch-srv01` before the abnormal activity started?
DroneManager EastmasUpdater

What is the full path where the user launched the application (found in question 1) from?
C:\Users\dispatch.admin\Downloads\DroneManager_Setup.exe

Which value was added by the application to maintain persistence on startup?
C:\Program Files\DroneManager\dronehelper.exe" --background

If you enjoyed today's room, feel free to check out the [Expediting Registry Analysis](https://tryhackme.com/room/expregistryforensics) room.
