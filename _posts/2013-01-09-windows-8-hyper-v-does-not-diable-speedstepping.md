---
layout: post
title: Windows 8 + Hyper-V does not diable speedstepping
created: 1357756880
---
After enabling [Hyper-V](http://en.wikipedia.org/wiki/Hyper-V) on my laptop running Windows 8, I noticed that *Task Manager* and *Resource Monitor* always showed the processor as running at full speed, as if [Speedstepping](http://en.wikipedia.org/wiki/SpeedStep) was suddenly disabled. This is however not the case. What is really happening is that Hyper-V is virtualizing the CPU for the host OS (Windows 8) as well. Opening up [CPU-Z](http://www.cpuid.com/softwares/cpu-z.html) reveals the real clock speed and multiplier of  the CPU.

Hope this helps.
