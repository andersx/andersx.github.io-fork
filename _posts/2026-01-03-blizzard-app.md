---
layout: post
title: "Installing Battle.net App on Ubuntu in 2026"
date: 2026-01-03
lang: en    
---

I've been playing Blizzard games on Linux since around 2012 first using PlayOnLinux, and later using Lutris.

Mostly it has been running flawlessly, with the major problems actually being with Nvidia drivers. But not this time...

Since the last update to the Battle.net app, I had a lot of trouble getting it to run on my Ubuntu 24.04. Funny thing is that StarCraft II would run fine, but the Battle.net app would not allow logging in. And eventually StarCraft II got patched requiring me to use the Battle.net app to launch it, which I could not do.

Turns out it was an outdated Wine version that was the problem.
So here is how I got it working again in early 2026 on Ubuntu 24.04 LTS.


---

### Step 1: Installing Lutris

The recommended way to install Lutris on Ubuntu is via `.deb` files off of Github.
Unfortunately, there is no PPA available at the moment that I could find.

So go to [https://github.com/lutris/lutris/releases](https://github.com/lutris/lutris/releases) and download the latest `.deb` file.

Now, install some dependencies:
```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install libgl1:i386 libgl1-mesa-dri:i386 libgnutls30:i386 mesa-vulkan-drivers:i386
```

Then install Lutris using `dpkg` (adjust the filename if a newer version is available):

```bash
sudo dpkg -i ~/Downloads/lutris_0.5.18_all.deb
```



### Step 2: Installing Battle.net via Lutris

Now go to the Lutris page for Battle.net app: [https://lutris.net/games/battlenet/](https://lutris.net/games/battlenet/)
Press the "Install" button, and Lutris should open up and guide you through the installation process.

- If prompted to install Mono, cancel it (it can freeze the process).
- Alternatively you can download the execuable from e.g [https://www.blizzard.com/en-us/apps/battle.net/desktop](https://www.blizzard.com/en-us/apps/battle.net/desktop) and point Lutris to that file during installation, in case the automatic install script does not work.

### Step 3: Customizing Wine Version

I was having a lot of problems logging in to the Battle.net app until I switched to a custom Wine version.
For this I found that I needed to install a different runner than the default wine runner that ships with Lutris.

Someone conveniently made a manager called `ProtonUp-Qt` that can install custom Wine/Proton versions for Lutris and Steam.
This manager is managed through yet another package manager `Flatpak`.
```bash
# Install protonup-qt via flatpak
flatpak install flathub net.davidotek.pupgui2

# Run protonup-qt with flatpak
flatpak run net.davidotek.pupgui2
```

In `ProtonUp-Qt`, select the `Lutris` tab, and then select `Add version` -> `Compatibility tool:` == `Wine Tgk` and `Version:` == `wine-tkg-valve-exp-bleeding-experimenta.bleeding.edge.10.0.28.0432.20251205` (was the latest version that I am using at the time of writing). Then press the Install button. 
It takes a while to download and install and the progress bar did not show up for me until it was done, so leave the window open for a while until it appears.

### Step 4: Configuring Lutris to use the custom Wine version

Start Lutris from the terminal with the debug flag to see what is going on:
```bash
lutris -d
```
I had the dependencies installed from Step 1 already, so I did not have to install any more dependencies. You'll see any missing dependencies for lutris show in the terminal output so go back and install them if needed.

Now right-click on the Battle.net app entry in Lutris and select `Configure`.
Go to the `Runner options` tab, and under `Wine version`, select the custom Wine version you installed with `ProtonUp-Qt` in Step 3. It should be named something like `wine-tkg-valve-exp-bleeding-experimental.edge.10.0.28.0432.20251205` in the dropdown menu.




*– Anders*

