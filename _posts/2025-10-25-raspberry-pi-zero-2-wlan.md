---
layout: post
title: "Fix for Raspberry Pi Zero 2 Wlan Issue"
date: 2025-10-25
lang: en    
---

I had the problem that the WLAN connection on my Raspberry Pi Zero 2 was very unstable and often dropped out.
This was on a fresh installation of Raspberry Pi OS Lite (64-bit), and I had the same problem on two different Raspberry Pi Zero 2s.

I found a forum post that mentioned a similar problem and that disabling power saving for the WLAN interface could help.

See also this [Raspberry Pi StackExchange thread](https://raspberrypi.stackexchange.com/questions/96606/make-iw-wlan0-set-power-save-off-permanent).

---

Check the status of power saving with the command:

```bash
iw dev wlan0 get power_save
```

What I did was to create a udev rule that permanently disables power saving for the WLAN interface. Might not be the best option for power saving, but if you want to rely on SSH access, etc. and you are hardwired on power anyway its probably fine.

Create a file `/etc/udev/rules.d/70-wifi-powersave.rules` with the following content:

```bash
ACTION=="add", SUBSYSTEM=="net", KERNEL=="wlan*", RUN+="/sbin/iw dev %k set power_save off"
```

Reboot.

Check again with:

```bash
iw dev wlan0 get power_save
```

It should now say `Power save: off`.


---


*– Anders*

