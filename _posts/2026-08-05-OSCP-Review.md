---
title: "OSCP Review: My Journey to Becoming an Offensive Security Certified Professional"
date: 2026-08-05
categories:
  - certifications
tags:
  - oscp
  - offsec
  - pentesting
  - infrastructure-hacking
  - review
header:
  teaser: /assets/images/certs/oscp+.png
toc: true
toc_label: "Contents" 
toc_icon: "bug" 
excerpt: "My honest review of the OSCP exam: preparation, exam experience, lessons learned, and tips for future candidates."
---

---
description: >-
  In this review, I'll share everything I wish I had known before taking
  this certification exam, along with tips and my personal opinions about it.
icon: magnifying-glass
---

# Review

<figure><img src="/assets/images/certs/oscp+.png" alt=""><figcaption></figcaption></figure>

### 🧠 Experience before the exam

When I decided to prepare for the **OSCP+**, I wasn't starting from scratch. Over the previous years, I had been building a solid foundation both practically and theoretically through different certifications and, above all, thanks to my experience conducting security audits in a professional environment.

Before taking on this challenge, I had already obtained certifications such as **eJPT**, **eCPPT**, and **OSWP**, each of which gave me different knowledge within the field of offensive security. While some reinforced my methodology during the reconnaissance, exploitation, and post-exploitation phases, others allowed me to dive deeper into more specific areas such as wireless network security.

However, I believe the experience gained working as a pentester was one of the aspects that helped me the most during preparation. In day-to-day work, you learn that an audit isn't just about finding vulnerabilities, but about following a methodology, properly documenting each step, and maintaining an analytical mindset when things don't go as expected.

Even so, the **OSCP+** is a different kind of challenge. Regardless of your prior experience, it requires many hours of practice, consistency, and the ability to work autonomously when facing problems that don't always have an obvious solution.

***

### 📚 How I prepared

My preparation was based mainly on practice. From the start, I was clear that understanding the concepts wasn't enough; I needed to face environments as close as possible to a real audit.

I started working through the official **PEN-200** material and labs, which helped me reinforce the methodology and understand OffSec's approach. From there, I expanded my practice with different platforms and community-created labs.

Among the resources that helped me the most, I would highlight:

* **Medtech**, for practicing Active Directory and pivoting.
* **Relia**, with a similar approach but with increased environment complexity.
* **Skylark**, a much larger Active Directory network where you need to chain different techniques together and maintain a solid methodology.
* The **OSCP A**, **OSCP B**, and **OSCP C** labs, designed to simulate a full certification scenario and test time management and the ability to solve problems under pressure.

I also solved numerous individual machines to reinforce specific techniques and get used to facing unknown targets.

One of the lists I used the most was **LainKusanagi's**, since it compiles a large number of machines recommended by the community for OSCP preparation. I found it especially useful for organizing my practice and making sure I covered different difficulty levels and technologies.

Beyond just completing machines, I tried to make each one an opportunity to improve my methodology, document my procedures, and understand the reasoning behind each technique used.

***

### 🧪 Exam environment

Without going into details about the exam content itself, I can talk about how I experienced it from the perspective of the working environment.

The exam is taken by connecting to the remote lab via **VPN**, so you work from your own machine. In my opinion, this is a big advantage, since you can use the same environment you've used throughout your entire preparation.

My recommendation is to set up a **Kali Linux** virtual machine in advance (or **Parrot Security**, if that's the distro you're more comfortable with) and use it throughout your preparation. Arriving at the exam with an environment you already know, with your tools organized, your aliases, scripts, notes, and usual configurations, will let you focus solely on solving the problems instead of wasting time setting up the system.

I also recommend checking a few days beforehand that everything works correctly: the virtual machine, the connection, snapshots, available disk space, and any tools you use frequently. Exam day is not the time to discover that a dependency is missing or that a configuration isn't working as expected.

Beyond the technical side, I believe preparing the environment in advance brings a lot of peace of mind. The fewer concerns you have about your setup, the easier it will be to stay focused throughout the entire test.

***

### 🛠️ Tools I used

The truth is there's no magic list. What matters is knowing the tools you use well and knowing when to use them.

These were the ones I used the most during my preparation:

| xfreerdp     | bloodhound   | wpscan        | burpsuite   |
| ------------ | ------------ | ------------- | ----------- |
| Metasploit   | mimikatz     | hydra         | john        |
| hashcat      | msfvenom     | powerview.ps1 | powerup.ps1 |
| evil-winrm   | searchsploit | netcat        | python3     |
| crackmapexec | kerbrute     | Impacket      | nmap        |

***

### ✔️ Honest advice for anyone considering it

* Don't rush to sign up. It's better to go in with confidence than to do it just to meet a deadline.
* Practice until your methodology becomes second nature.
* Learn to document everything you do from day one.
* If something isn't working, avoid falling into a "rabbit hole." Knowing when to change strategy is also part of the exam.
* Don't obsess over comparing your progress to other people's. Everyone learns at a different pace.
* Resting is also part of the preparation. Mental clarity makes a huge difference when you've spent many hours solving problems.