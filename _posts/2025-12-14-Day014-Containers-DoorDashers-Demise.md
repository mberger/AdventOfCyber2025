---
layout: post
title:  "Day 14 - Containers - DoorDasher's Demise"
date:   2025-12-14
categories: welcome
image: Day14-banner.png
---

# Container Security - Containers Gone Wild


TryHackMe Room: https://tryhackme.com/room/container-security-aoc2025-z0x3v6n9m2Advent of Cyber 2025 – Day 14

### Quick Overview

* **Difficulty**: Medium
* **Type**: Docker container enumeration + escape/privilege escalation CTF
* **Time**: \~50–70 minutes
* **Goal**: Audit misconfigured containers on the North Pole's toy delivery platform, escape to the host, and stop Sir Carrotbane from hijacking the entire fleet

### The Story in a Nutshell

The elves modernized the sleigh dispatch system with shiny Docker containers—because who doesn't love microservices for gift routing? But Carrotbane's cracked the setup: he's running rogue containers, mounting sensitive host dirs, and plotting a full breakout to reroute every present to EASTMAS warehouses. McSkidy's latest smuggled intel points to the container host. You're the container cop: jump into the exposed services, enumerate the Docker environment, exploit bad configs, and privesc to root on the host before the bunnies containerize Christmas itself.

### What You Actually Do (Task Flow)


 1. Deploy the VM → scan and access the exposed web/app (usually a toy tracker on port 8080 or similar).
 2. **Container Basics**: Learn Docker risks—privileged mode, volume mounts, exposed sockets, capability drops.
 3. **Enumeration Inside**: From a low-priv container (maybe via a vuln web app), run `docker ps`, check mounts (`/proc/mounts` or `df -h`), spot the Docker socket `/var/run/docker.sock`.
 4. **Socket Abuse**: If mounted, curl or use a client to spawn new containers with host access (e.g., `--privileged -v /:/host`).
 5. **Cap Drop Fail**: Exploit retained caps like CAP_SYS_ADMIN to mount filesystems or chroot escapes.
 6. **Volume Shenanigans**: Write to host-mounted dirs (e.g., /etc for cron jobs or SSH keys).
 7. **Image Tricks**: Pull or run malicious images, or overwrite entrypoints.
 8. **Host Breakout**: Classic moves—mount host root, chroot, edit passwd, or drop a reverse shell as root.
 9. **Secure It Up**: Theory on best practices (least priv, no socket mounts, user namespaces).
10. Grab flags: container IDs, escaped shell proof, final root flag on host.

### Key Commands/Tools You'll Actually Use and Remember

```bash
docker ps                        # list running containers
docker exec -it <id> /bin/sh     # jump in (if socket access)
mount | grep host                 # spot bad mounts
cat /proc/1/mounts               # host view from container
curl -X POST --unix-socket /var/run/docker.sock http://localhost/containers/create ...
docker run --rm -it -v /:/host --privileged alpine chroot /host /bin/sh
```

* **Core Concepts**: Docker privilege escalation vectors (socket mount, privileged flag, capabilities, volume binds); container breakout techniques; secure config basics.

### Why It’s Awesome for Beginners (to Container Sec)

* That "I'm inside a container... now what?" moment turns into pure magic when you pop root on the host.
* Hands-on without needing your own Docker setup—everything's prepped in the VM.
* Builds on earlier privesc days but adds the modern cloud-native twist everyone's talking about.
* Festive flavor: Containers named "sleigh-router", "gift-db", with bunny droppings in the logs.

Boot the machine, find your container foothold, and start climbing out. By the end, you'll audit any Docker deploy like a pro, and Carrotbane's container empire will be sinking fast. Day 14 contained—host owned, Christmas shipping secured! 🎄


## Task 1 - Introduction

 ![Task banner for day 1](https://tryhackme-images.s3.amazonaws.com/user-uploads/6228f0d4ca8e57005149c3e3/room-content/6228f0d4ca8e57005149c3e3-1763378735412.png)


It seemed as the sun rose this morning, it had already been decided that today would be another day of chaos in Wareville. At least that’s the feeling all the folks at “DoorDasher” got. DoorDasher is Warevilles local food delivery site, a favourite of the workers in The Best Festival Company, especially on long days when they get home from work and just can’t bring themself to make dinner. We’ve all been there, I’m sure.


Well, one Wareville resident was feeling particularly tired this morning and so decided to order breakfast. Only to find King Malhare and his bandit bunny battalions had seized control of yet another festive favourite. DoorDasher had been completely rebranded as Hopperoo. All of the ware’s favourite dishes had been changed as well. Reports started flooding into the DoorDasher call centre. And not just from customers. The health and safety food org was on the line too, utterly panicked. Apparently, multiple Wareville residents were choking on what turned out to be fragments of Santa’s beard. Wareville authorities were left tangled in confusion today as Hopperoo faced mounting backlash over reports of “culinary impersonation.” Customers across the region claim to have been served what appears to be authentic strands of Santa’s beard in place of traditional noodles.


A spokesperson for the Health & Safety Food Bureau confirmed that several diners required “gentle untangling” and one incident involved a customer “achieving accidental facial hair synchronisation.”


 ![Beardgate news coverage](https://tryhackme-images.s3.amazonaws.com/user-uploads/6228f0d4ca8e57005149c3e3/room-content/6228f0d4ca8e57005149c3e3-1763634056631.png)


Immediately, one of the security engineers managed to log on and make a script that would restore DoorDasher to its original state, but just before he was able to run it, Sir CarrotBaine caught wind of his attempt and locked him out of the system. All was lost, until the SOC team realised they still had access to the system via their monitoring pod, an uptime checker for the site. Your job? As a SOC team member of DoorDasher, can you escape the container and escalate your privileges so you can finish what your team started and save the site!


## Learning Objectives


* Learn how containers and Docker work, including images, layers, and the container engine
* Explore Docker runtime concepts (sockets, daemon API) and common container escape/privilege-escalation vectors
* Apply these skills to investigate image layers, escape a container, escalate privileges, and restore the DoorDasher service
* DO NOT order “Santa's Beard Pasta”


## Connecting to the Machine


Before moving forward, review the questions in the connection card shown below:


 ![Connection Card](https://tryhackme-images.s3.amazonaws.com/user-uploads/61a7523c029d1c004fac97b3/room-content/61a7523c029d1c004fac97b3-1764847103823.png)


Start the lab by clicking the **Start Machine** button below. The machine will start in split view and will take about two minutes to load. In case the machine is not visible, click the **Show Split View** button at the top of the page. Once the machine has loaded, you should be given access to the `mrbombastic` user. You will be given commands to run on this virtual machine in the next task. Additionally, start the AttackBox by pressing **Start AttackBox** down below. 


### Set up your virtual environment

To successfully complete this room, you'll need to set up your virtual environment. This involves starting both your AttackBox (if you're not using your VPN) and Target Machines, ensuring you're equipped with the necessary tools and access to tackle the challenges ahead.

 ![](https://tryhackme.com/static/svg/attack-box-to-target-machine.e30b7a02.svg)

Attacker machine ![Machine info](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTciIGhlaWdodD0iMTYiIHZpZXdCb3g9IjAgMCAxNyAxNiIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHBhdGggZD0iTTguODI1MiAwQzQuNDA2NDUgMCAwLjgyNTE5NSAzLjU4MTI1IDAuODI1MTk1IDhDMC44MjUxOTUgMTIuNDE4NyA0LjQwNjQ1IDE2IDguODI1MiAxNkMxMy4yNDM5IDE2IDE2LjgyNTIgMTIuNDE4NyAxNi44MjUyIDhDMTYuODI1MiAzLjU4MTI1IDEzLjI0MzkgMCA4LjgyNTIgMFpNOC44MjUyIDRDOS4zNzczOCA0IDkuODI1MiA0LjQ0NzgxIDkuODI1MiA1QzkuODI1MiA1LjU1MjE5IDkuMzc3MzggNiA4LjgyNTIgNkM4LjI3MzAxIDYgNy44MjUyIDUuNTUzMTIgNy44MjUyIDVDNy44MjUyIDQuNDQ2ODggOC4yNzIwNyA0IDguODI1MiA0Wk0xMC4wNzUyIDEySDcuNTc1MkM3LjE2MjcgMTIgNi44MjUyIDExLjY2NTYgNi44MjUyIDExLjI1QzYuODI1MiAxMC44MzQ0IDcuMTYxMTMgMTAuNSA3LjU3NTIgMTAuNUg4LjA3NTJWOC41SDcuODI1MkM3LjQxMTEzIDguNSA3LjA3NTIgOC4xNjQwNiA3LjA3NTIgNy43NUM3LjA3NTIgNy4zMzU5NCA3LjQxMjcgNyA3LjgyNTIgN0g4LjgyNTJDOS4yMzkyNiA3IDkuNTc1MiA3LjMzNTk0IDkuNTc1MiA3Ljc1VjEwLjVIMTAuMDc1MkMxMC40ODkzIDEwLjUgMTAuODI1MiAxMC44MzU5IDEwLjgyNTIgMTEuMjVDMTAuODI1MiAxMS42NjQxIDEwLjQ5MDggMTIgMTAuMDc1MiAxMloiIGZpbGw9IiM4NzhGQTIiLz4KPC9zdmc+Cg==)Status:Off

Target machine ![Machine info](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTciIGhlaWdodD0iMTYiIHZpZXdCb3g9IjAgMCAxNyAxNiIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHBhdGggZD0iTTguODI1MiAwQzQuNDA2NDUgMCAwLjgyNTE5NSAzLjU4MTI1IDAuODI1MTk1IDhDMC44MjUxOTUgMTIuNDE4NyA0LjQwNjQ1IDE2IDguODI1MiAxNkMxMy4yNDM5IDE2IDE2LjgyNTIgMTIuNDE4NyAxNi44MjUyIDhDMTYuODI1MiAzLjU4MTI1IDEzLjI0MzkgMCA4LjgyNTIgMFpNOC44MjUyIDRDOS4zNzczOCA0IDkuODI1MiA0LjQ0NzgxIDkuODI1MiA1QzkuODI1MiA1LjU1MjE5IDkuMzc3MzggNiA4LjgyNTIgNkM4LjI3MzAxIDYgNy44MjUyIDUuNTUzMTIgNy44MjUyIDVDNy44MjUyIDQuNDQ2ODggOC4yNzIwNyA0IDguODI1MiA0Wk0xMC4wNzUyIDEySDcuNTc1MkM3LjE2MjcgMTIgNi44MjUyIDExLjY2NTYgNi44MjUyIDExLjI1QzYuODI1MiAxMC44MzQ0IDcuMTYxMTMgMTAuNSA3LjU3NTIgMTAuNUg4LjA3NTJWOC41SDcuODI1MkM3LjQxMTEzIDguNSA3LjA3NTIgOC4xNjQwNiA3LjA3NTIgNy43NUM3LjA3NTIgNy4zMzU5NCA3LjQxMjcgNyA3LjgyNTIgN0g4LjgyNTJDOS4yMzkyNiA3IDkuNTc1MiA3LjMzNTk0IDkuNTc1MiA3Ljc1VjEwLjVIMTAuMDc1MkMxMC40ODkzIDEwLjUgMTAuODI1MiAxMC44MzU5IDEwLjgyNTIgMTEuMjVDMTAuODI1MiAxMS42NjQxIDEwLjQ5MDggMTIgMTAuMDc1MiAxMloiIGZpbGw9IiM4NzhGQTIiLz4KPC9zdmc+Cg==)Status:Off

**Note:** It’s recommended to open both machines in full-screen view using the small icon on the far left in the screenshot below; otherwise, you might get kicked out of the Docker container when switching tabs in split view. If you still prefer to use split view, you can switch between the target machine and the AttackBox using the bottom tabs, as shown below:


 ![](https://tryhackme-images.s3.amazonaws.com/user-uploads/5f9c7574e201fe31dad228fc/room-content/5f9c7574e201fe31dad228fc-1765727030820.png)

Answer the questions below

Tell me more!


## Task 2 - Container Security

## What Are Containers?


To understand what a container is, we first need to understand the problem it fixes. Put plainly, modern applications can be quite complex:


* **Installation**: Depending on the environment the application is being installed in, it’s not uncommon to run into “*configuration quirks*” which make the process time-consuming and frustrating. 
* **Troubleshooting**: When an application stops working, a lot of time can be wasted determining if it is a problem with the application itself or a problem with the environment it is running in.
* **Conflicts**: Sometimes multiple versions of an application need to be run, or perhaps multiple applications which need (for example) different versions of Python to be installed. This can sometimes lead to conflicts, complicating the process further.


If reading these problems, your brain conjured up the word “isolation” as a solution, well, you’re onto the right track. Containerisation solves these problems by packing applications, along with their dependencies, in one isolated environment. This package is known as a container. In addition to solving all the above problems, containerisation also offers further benefits. One key benefit is how lightweight they are. To understand this, we will now take a (brief) look at container architecture.


**Containers vs VMs**


To illustrate container architecture, let's examine another form of virtualisation: Virtual Machines. Virtual Machines, like the ones you have been booting up on this platform throughout Advent of Cyber.


 ![Virtualisation diagram](https://tryhackme-images.s3.amazonaws.com/user-uploads/6228f0d4ca8e57005149c3e3/room-content/6228f0d4ca8e57005149c3e3-1763634135270.png)


A virtual machine runs on a hypervisor (software that emulates and manages multiple operating systems on one physical host). It includes a full guest OS, making it heavier but fully isolated. Containers share the host OS kernel, isolating only applications and their dependencies, which makes them lightweight and fast to start. Virtual machines are ideal for running multiple different operating systems or legacy applications, while containers excel at deploying scalable, portable micro-services.


**Applications at Scale**


Microservices are a switch in the style of application architecture, where before it was a lot more common to deploy apps in a monolithic fashion (built as a single unit, a single code base, usually as a single executable ), more and more companies are choosing to break down their application into parts based on business function. This way, if a specific part of their application receives high traffic loads, they can scale that part without having to scale the entire application. This is where the lightweight nature of containers comes into play, making them incredibly easy to scale to meet increasing demands. They became the go-to choice for this (increasingly popular) architecture, and this is why, over the last 10 years, you have started seeing the term more and more. 


You may have noticed a layer labelled “Container Engine” in the diagram. A container engine is software that builds, runs, and manages containers by leveraging the host system’s OS kernel features like namespaces and cgroups. One example of a container engine is Docker, which we will examine in more detail. Docker is a popular container engine that uses Dockerfiles, simple text scripts defining app environments and dependencies, to build, package, and run applications consistently across different systems. Docker is the container engine of choice at “DoorDasher” and so is what we will be interacting with in our interactive lab. 


**Docker**


Docker is an open-source platform for developers to build, deploy, and manage containers. Containers are executable units of software which package and manage the software and components to run a service. They are pretty lightweight because they isolate the application and use the host OS kernel.


 ![Present Container](https://tryhackme-images.s3.amazonaws.com/user-uploads/6228f0d4ca8e57005149c3e3/room-content/6228f0d4ca8e57005149c3e3-1763634251845.png)


**Escape Attack & Sockets**


A container escape is a technique that enables code running inside a container to obtain rights or execute on the host kernel (or other containers) beyond its isolated environment (escaping). For example, creating a privileged container with access to the public internet from a test container with no internet access. 


Containers use a client-server setup on the host. The CLI tools act as the client, sending requests to the container daemon, which handles the actual container management and execution. The runtime exposes an API server via Unix sockets (runtime sockets) to handle CLI and daemon traffic. If an attacker can communicate with that socket from inside the container, they can exploit the runtime (this is how we would create the privileged container with internet access, as mentioned in the previous example).


## Challenge


Your goal is to investigate the Docker layers and restore the defaced Hopperoo website to its original service: DoorDasher. We are going to walk through the steps to beat this challenge together! 


**Access Points**


Now it's time to start using the machine you booted up in task 1. Let's check which services are running with Docker. Run the following command:


`docker ps`


You should see a few services running:


 ![docker ps command output](https://tryhackme-images.s3.amazonaws.com/user-uploads/61a7523c029d1c004fac97b3/room-content/61a7523c029d1c004fac97b3-1761169045152.png)


The main service runs at `http://MACHINE_IP:5001`. Explore the web application, and notice the defaced system “Hopperoo”. We know DoorDasher is a food delivery service. Let's explore `uptime-checker`. Sounds interesting.


Run the following command to access the `uptime-checker` container:


`docker exec -it uptime-checker sh`


After logging in, check the socket access by running: `ls -la /var/run/docker.sock`


The Docker documentation mentions that by default, there is a setting called “Enhanced Container Isolation” which blocks containers from mounting the Docker socket to prevent malicious access to the Docker Engine. In some cases, like when running test containers, they need Docker socket access. The socket provides a means to access containers via the API directly. Let's see if we can. Your output should be something like:


 ![socket permissions](https://tryhackme-images.s3.amazonaws.com/user-uploads/61a7523c029d1c004fac97b3/room-content/61a7523c029d1c004fac97b3-1761169453775.png)


I wonder what happens if we try to run Docker commands inside the container. By running `docker ps` again, we can confirm we can perform Docker commands and interact with the API; in other words, we can perform a Docker Escape attack! 


Let's instead try to access the `deployer` container, which is a privileged container. Run the following command:


`docker exec -it deployer bash`


Type `whoami` and check which user we are currently logged in as. Explore the container and find the flag.


Okay, we made it! We have managed to make it to the container where the script to restore the site to its former version is! Save the day, run the recovery_script in the root directory to recover the app using sudo:


`sudo /recovery_script.sh`


With that run, you can see for yourself by refreshing the site (`http://MACHINE_IP:5001`)! As a reward, you should be able to find a flag in the same directory as the script (`/`), which you can read using the `cat` command.

Answer the questions below

What exact command lists running Docker containers?
* docker ps

What file is used to define the instructions for building a Docker image?
* dockerfile

What's the flag?
* THM{DOCKER_ESCAPE_SUCCESS}

Bonus Question: There is a secret code contained within the news site running on port `5002`; this code also happens to be the password for the deployer user! They should definitely change their password. Can you find it?
* DeployMaster2025!

Liked the content? We have plenty more where this came from! Try our [Container Vulnerabilities](https://tryhackme.com/room/containervulnerabilitiesDG) room.


