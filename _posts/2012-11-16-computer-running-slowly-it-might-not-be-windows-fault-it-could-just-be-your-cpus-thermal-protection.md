---
layout: post
title: Computer running slowly – it might not be Windows’ fault – it could just be
  your CPUs thermal protection
created: 1353072808
redirect_from:
  - /2012/11/16/computer-running-slowly-it-might-not-be-windows-fault-it-could-just-your-cpus-thermal/
  - /2012/11/16/computer-running-slowly-it-might-not-be-windows-fault-it-could-just-be-your-cpus-thermal/
---
Conventional wisdom says that when a PC starts to run slowly and angry screaming at the computer screen ensues, it is time to reinstall Windows or remove some of the crap-ware that's been building up on the PC. However, it could be a hardware issue; it could be the computers cooling system that is unable to dissipate heat from the processor probably, causing the processors to lower its speed to 50% or more (see [Intel's description of this processor feature for more details](http://www.intel.com/cd/ids/developer/asmo-na/eng/downloads/54118.htm)).

<!--break-->

## Background
This happened to me last summer with my then Dell Precision M2400, a "workstation laptop", about a year old at the time. If I put the CPU under a relatively heavy load, i.e. working with Visual Studio, Photoshop, while watching a YouTube video, the system would after a little while become very sluggish, ending up in a usable state, where even scrolling was laggy. I started playing around and figured out I could get this to happen within minutes by just using [7-Zip](http://www.7-zip.org/) to compress a 6GB virtual machine. Usually, it would take complete shutdown of the computer for a 10-20 minutes before it would run normally again. After consulting Dell Support and trying various solutions, it was clear that this was not caused by any software. Even disabling [SpeedStep](http://en.wikipedia.org/wiki/SpeedStep) in the bios did not help.

{% include figure.html src="motherboard-cooling-system.jpg" caption="Photo of the cooling system in my Dell Precision M2400. It looks pretty clean, but this was still not clean enough to keep the CPU happy." %}

It turned out that the only thing that solved the problem was to replace the entire cooling system in the laptop. However, now that I was aware of the symptoms, I quickly noticed when the problem started to reappear as the cooling system slowly started to degrade in performance. In the end Dell replaced my laptop entirely with a different model, Dell Latitude E6420, which has not yet had the issue.

## How to detect the issue
If you think your experiencing this problem, the easiest way to verify it I have come across is by launching the **Resource Monitor** that comes with Windows Vista, Windows 7 and Windows 8 (just type "Resource Monitor" in the Start Menu/Start Screen). If you see the "CPU Usage" is higher than the "Maximum Frequency", you have a problem. The screenshot below is from a colleague's laptop (Dell Latitude E6400) which has the problem.

{% include figure.html src="resource-monitor-screenshot.png" caption="Screenshot of Resource Monitor and Core Temp running on a Dell Latitude E6400. Resource monitor shows the CPU Usage at 100% with the Maximum Frequency at 24%, and the graph to the right shows that the maximum frequency has been below the CPU usage for a while. Core Temp shows that the CPU is indeed running at 798.97 MHz." %}

Normally, Windows always keeps the CPU running as fast as it needs to keep up with the workload, i.e. keep the maximum frequency above the CPU usage, and run it slower when there is no need for the extra CPU cycles to save on battery life (or if power management options enforces it), but when the CPU is protecting itself from overheating, the OS cannot override the speed of the CPU and you are stuck with a 2.8 GHz CPU running at 800 MHz.

## Conclusion
The question is if this is just Dell pushing the limits of their cooling systems or if other manufactures have this issue as well. In the future, CPUs will certainly get less hot but cooling systems will also shrink in size to reduce the size of laptops, so if you system starts running slowly, it might just be the CPUs thermal protection kicking in.

I hope this can save others from a lot of frustration.