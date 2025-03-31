+++
date = '2025-03-30T11:56:15-04:00'
draft = false
title = 'Installing (Upgrading) Ollama'
+++

This is an urgent note about how I resolved a pressing issue installing Ollama on my Raspberry Pi. 

A few months ago, I successfully installed Ollama on my Raspberry Pi, and it performed flawlessly. I made some configuration tweaks to ensure everything worked smoothly at the time, but I couldn’t recall the changes I had implemented. Recently, after upgrading the Ollama server, disaster struck—all the applications reliant on this Raspberry Pi began to fail. I wrote this blog to prevent a repeat of this frustrating experience during future upgrades, documenting how I resolved the issue and restored functionality.

## What happened?

I’ve worked with models like Pi3 and Gemma3 on my Raspberry Pi, and while they performed adequately, I discovered a newer version of Gemma3 was available—an upgrade I couldn’t resist. At the time, I was running Ollama version 0.3.x, but the latest Gemma3 model required Ollama version 0.6.0 or higher. As someone prioritizing staying up-to-date with the newest software, I promptly upgraded my installation. This upgrade occurred on a Friday, and as fate would have it, I overlooked the change and didn’t use the applications reliant on my local Ollama installation for weeks. When I finally reconnected, every application hit me with exceptions, indicating that the upgrade needed further attention.

The challenge I faced was rooted in confusion—I couldn’t pinpoint the exact cause of the problem. Was the issue introduced by the Ollama server upgrade itself, or was it connected to the change in my network setup? Around the same time, I had replaced my router, switching from one that primarily used IPV4 to a newer model that emphasized IPV6. This shift introduced the possibility of a firewall misconfiguration or connectivity hiccups that I hadn’t anticipated. Alternatively, could the problem have stemmed from the Windows update that occurred in parallel with these changes? The timing of these events made troubleshooting even more complex, as I had to consider multiple variables and their potential interactions.

On the surface, each factor seemed plausible, but I lacked the clarity to determine which one—or combination of them—was to blame. The Ollama upgrade introduced compatibility concerns with my applications, while the router change could have affected network communication protocols. Simultaneously, the Windows update may have brought unexpected changes to system settings, firewall rules, or application permissions. The overlapping nature of these modifications created a perfect storm of uncertainty, leaving me scrambling to identify the root cause of the failure.

This experience served as a stark reminder of the importance of thorough testing and documentation when implementing multiple changes to a system. It underscored how seemingly unrelated upgrades and modifications could combine to create unforeseen issues, leaving critical applications in limbo. As I delved deeper into the problem, I realized that unraveling this tangled web of possibilities would require methodical investigation and careful analysis of each component involved. My goal became not just resolving the immediate issue but ensuring that I could learn from this situation and mitigate similar risks in the future.

## Fix
To expose Ollama on your network, you must change the default bind address from 127.0.0.1 to 0.0.0.0. This allows Ollama to accept connections from any IP address and makes it accessible from other devices on the same network. The bind address can be modified using the OLLAMA_HOST environment variable.

I run Raspberry Pi OS, which is Debian-based, on my Raspberry Pi. The required change needs to be done in ```ollama.service``` file, and the way one updates this file is as follows:
```bash
systemctl edit ollama.service
```
In the editor, add the following line under the ```[Service]``` section:
```bash
[Service] 
Environment="OLLAMA_HOST=0.0.0.0:11434"
```
Save and exit the editor, and reload systemd configuration and restart Ollama as follows:
```bash
systemctl daemon-reload 
systemctl restart ollama
```

Ollama should be accessible from a remote machine.  **If not, update this document after you resolve the issue 😔.**