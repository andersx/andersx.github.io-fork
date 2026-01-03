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

### Step 1: Install a custom Wine version using ProtonUp-Qt

I was having a lot of problems logging in to the Battle.net app until I switched to a custom Wine version.
For this I found that I needed to install a different runner than the default wine runner that ships with Lutris.

This step is a prerequisite for the next step.

Someone conveniently made a manager called `ProtonUp-Qt` that can install custom Wine/Proton versions for Lutris and Steam.
This manager is managed through yet another package manager `Flatpak`.
```bash
# Install protonup-qt via flatpak
flatpak install flathub net.davidotek.pupgui2

# Run protonup-qt with flatpak
flatpak run net.davidotek.pupgui2
```

In `ProtonUp-Qt`, select `Lutris (/path/to/lutris)` drop down menu `Install For`, and then select `Add version` -> `Compatibility tool:` == `Wine Tgk (Vale Wine Bleeding Edge)` and `Version:` == `20028xxxx` whatever the top of the list number in the list is (at the time of writing it was 10.0.28.0432).

Then press the Install button. 
After a while you should see something like: `wine-tkg-valve-exp-bleeding-experimenta.bleeding.edge.10.0.28.0432.20251205` in the list of install tools (was the latest version that I am using at the time of writing).


### Step 2: Installing Lutris

The recommended way to install Lutris on Ubuntu is via `.deb` files off of Github.
Unfortunately, there is no PPA available for ubuntu 24.04 LTS at the moment that I could find. I think it exists for 22.04, so you can probably use the PPA if you are still on 22.04.

So go to [https://github.com/lutris/lutris/releases](https://github.com/lutris/lutris/releases) and download the latest `.deb` file.

Now, install some dependencies:
```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install libgl1:i386 libgl1-mesa-dri:i386 libgnutls30:i386 mesa-vulkan-drivers:i386 python3-protobuf protobuf-compiler wine
```

Then install Lutris using `dpkg` (adjust the filename if a newer version is available):

```bash
sudo dpkg -i ~/Downloads/lutris_0.5.18_all.deb
```
If you get complaints about missing dependencies, run:
```bash
sudo apt --fix-broken install
```
You can also use `apt` to install the `.deb` file, which will automatically resolve dependencies:
```bash
sudo apt update
sudo apt install ~/Downloads/lutris_0.5.18_all.deb  # Note the absolute path 
```

##### Step 2.5: Setting up Lutris
Now start Lutris from your application menu or by running `lutris -d` in a terminal.
A bunch of addons will be installed the first time you run it (lower left corner). Wait for these to finish.

If you see any missing dependencies warnings, install the missing dependencies (I believe I have most of them installed from the previous step, though).

**Important:**
- Make sure you have the latest Wine version installed via `ProtonUp-Qt` as described in Step 1 
- **DO THIS BEFORE PROCEEDING TO STEP 3:** Now, click the Wine gear icon in the bottom left in lutris and click on the dropdown menu for `Wine version` and select the bleeding edge version you installed with `ProtonUp-Qt`. This makes sure that Lutris uses the correct Wine version when installing Battle.net.

Ok, with Lutris set up and the correct Wine version selected, we can now proceed to install Battle.net.

### Step 3: Installing Battle.net via Lutris

Now go to the Lutris page for Battle.net app: [https://lutris.net/games/battlenet/](https://lutris.net/games/battlenet/)
Press the "Install" button, and Lutris should open up and guide you through the installation process.

- **DO NOT INSTALL MONO** If prompted to install Mono, press cancel (it can freeze the process).
- You can also download the execuable from e.g [https://www.blizzard.com/en-us/apps/battle.net/desktop](https://www.blizzard.com/en-us/apps/battle.net/desktop) and point Lutris to that file during installation, in case the automatic install script does not work. Both worked for me this time, but I've had problems with the automated downloader in the past.

Quick command to download the Battle.net installer:
```bash
wget "https://www.battle.net/download/getInstallerForGame?os=win&version=LIVE&gameProgram=BATTLENET_APP" -O Battle.net-Setup.exe
```

This can take some times (a few minutes) before you'll get to the the dialog boxes to log in to your Battle.net account etc.
Once you have logged in exit the Battle.net app from the menu in the top left corner.

### Step 4: Running Battle.net

Start Lutris again from the terminal `lutris -d`, and double click on Battle.net in your library to start it up. Then you can install your games as usual.

I tried this on two different Ubuntu 24.04 LTS machines, one with an Nvidia GPU and one with an Intex iGPU and was successful on both instances. I tested StarCraft II on both and they worked, I am not sure about other games.


*– Anders*

