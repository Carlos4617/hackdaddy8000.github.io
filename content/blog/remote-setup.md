---
title: Remote Development for Programming on the Go
date: 2022-09-08 16:43:00
tags: ["vscode", "hpc", "distributed systems"]
series: ["Tips"]
featured: false
---

The worst parts of developing on a laptop while unplugged are:
1. The battery life is eaten away if you try to run any program.
2. The laptop undervolts the processor in order to preserve what little battery life you have, making everything run excrutiatingly slow.

During my internship at Meta, I noticed that my laptop battery would last literally all day and everything was extremely fast. The linter and build process in my IDE were running nonstop since I save my code with every other key press. My experience developing on a laptop in Meta's ecosystem was frustration free in contrast to the misery I endure on my personal laptop. The key difference is that on my Meta laptop, I was connected to a remote virtual machine instead of developing locally.

In addition to this, I briefly worked at my university's supercomputer where I got hands-on experience with their job-creation software. In short, you SSH into a remote server. In your personal directory on that server, you FTP the code you want to run and create something like a docker file that contains instructions on how to run your code. Since this was a university supercomputer, researchers commonly created highly parallel programs. In that file, you could define how many compute nodes you need and what capabilities your compute nodes need (# of cores, GPU, RAM, etc).

I wanted to recreate these amazing features for my own personal use.

## Commercial Solutions

There are a few cloud solutions already

* Github Remote Development
* Jetbrains Space
* Using a node on AWS, Google Cloud, etc

I personally would rather use my own hardware because if I bought a $1400 PC for gaming, I'm going to get as much value out of it.

## My Setup

Will finish this later

Essentially I VPN into my network, SSH into a login node from which I can create jobs that run on my other hardware.