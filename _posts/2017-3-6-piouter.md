---
layout: post
title: Piouter
---

## The Problem
Using my chromecast with my apartment's wifi is a game of chance. The wifi is provided by the apartment, which is nice, and it works across the whole apartment, not just in your unit. The problem is that this means when you connect to the wifi you're connecting to the nearest router - which is fine unless you and your chromecast end up on different routers. Somehow, my roommate and I managed to place the TV such that the chromecast is right between two routers, so you never know which one it's connected to, and which one *you're* connected to. Trying to play music through it is a game of going from device to device hoping that one of them is on the same network your chromecast is. It's infuriating.

## The Solution
Enter the Raspberry Pi I won two years ago at a hackathon. I kept it around just in case I ever wanted to do a hardware project, and now it was finally ready for it's prime time. Technically my wifi contract forbids me from using a router, but I wanted to be able to use my chromecast in a sane manner, and I wanted to be able to actually allow my friends to get on a wifi network. So away with rules.

I don't actually really know anything about setting up a router. Most of what I did was google around and follow instructions I found on sketchy looking forums. Fortunately, the Pi I won at the hackathon came equipped with a little wifi dongle. Unfortunately, it ended up causing me a world of pain.

I pretty much entirely followed [this](https://learn.adafruit.com/setting-up-a-raspberry-pi-as-a-wifi-access-point?view=all) tutorial, but the hostapd configuration gave me some trouble. I was using the Rasbian OS (as the Pi came with an SD card that had it preinstalled) but the installed hostapd version is 0.7. Unfortunately, the wifi dongle that I was using required the `nl80221` driver, which isn't supported in hostapd until version 2.0.0. I had to [download](https://w1.fi/hostapd/) the most recent package from the hostapd website and manually install.

```
cd /tmp
wget https://w1.fi/releases/hostapd-2.6.tar.gz
tar -xvzf hostapd-2.6.tar.gz
cd hostapd-2.6/hostapd
cp defconfig .config
make
sudo make install
```
Note:  Adafruit also has it's own explanation of how to download and custom install the newer hostapd (at the bottom of the page), but I figured it out on my own so I wanted to show off my solution.

## The Roommate
So, three days in, it finally appeared that everything worked. At this point  my roommate informed me that he actually had a *real* router. After yelling at him for not telling me this earlier, I decided to go ahead and use mine because I hadn't done all that work for nothing. I plugged in the ethernet, plugged in the power, booted it up, and connected with my phone. My phone proceeded to inform me that I had no internet. Sighing, I plugged in the piouter to a monitor, and tried to figure out what was wrong. After finding nothing wrong I reconnected with my phone, and it worked. So maybe it just needed a little more time (???).

## The Aftermath
I'm very happy to now have a wifi that I can actually let guests use so that they can go on facebook and ignore me while I serve them tea and attempt to talk to them.

## The AfterAftermath
So, several days later, I accidentally hit a button in my kitchen that reset the electrical outlet the piouter was plugged into, causing it to reboot, and fail to work like normal. After several days of being annoyed at my inability to use the chromecast, I finally plugged the box back into the TV and tried to figure out what went wrong. After spending a fun hour shutting down and putting back up network services in various different orders, I found the correct order to start the processes in. Because I was too lazy to figure out how to configure the boot sequence to bring the services up in the correct order, I created a bash script, `fix_this_dumb_thing.sh` (source at bottom), which took all the daemons down and brought them back up in the correct order. I left it in the home directory and patted myself on the back.

On a separate note, for my ethical hacking class I had to run a vulnerability scan against a unix computer, so I decided to do it against the piouter. I used [nessus](https://www.tenable.com/products/nessus-vulnerability-scanner) which revealed, two medium vulnerabilities and two low vulnerabilities. Upon more in depth investigation, it appeared that all of the vulnerabilities had to do with the ssh server accepting a slightly weaker encryption algorithm. I'm not gonna devote time to messing with that.

## Future improvements
* Fix vulnerabilities revealed by nessus (but really, who cares)
* ~~Set up wireshark and capture all traffic to get the passwords of everyone who uses the network~~

### Notes
`fix_this_dumb_thing.sh`:

```bash
service hostapd stop
service isc-dhcpd-server stop
ifdown wlan0
ifup wlan0
service hostapd start
service isc-dhcpd-server start
```
