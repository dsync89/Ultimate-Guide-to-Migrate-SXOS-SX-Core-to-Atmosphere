# Ultimate-Guide-to-Migrate-SXOS-SX-Core-to-Atmosphere

# Migrate from SXOS 3.0.5 (SX Core hardware mod) to Atmosphere 0.19.3

I had been away from SXOS scene for more than a year, and recently just got back to power on my Switch to play some of the recent titles in 2021. I have prior experience in jailbreaking Nintendo 3DS, the new Nintendo 3DS, Sony Playstation PSP Slim, Sony Playstation Vita (phat), and Sony Playstation Vita (Slim). While the tools used differ for each console, they do share some similarities and I find it easier to understand and compare the process. 

Having said that, I decided to wrote this as a log to my attempts, and something that I could read back in the future. I wrote down each rationale for the steps so that you can better understand the process yourself. You may follow it if you have the same setup as I did! 

**NOTE: If you ever decide to follow my steps, please carefully read through all the steps, make sure all the release links are available, and understand what you are doing. And I do not hold responsible if you bricked your Switch console, which is very unlikely though.**

All the files I downloaded are the latest release that I could obtained as of May 29, 2021.

tldr: I successfully updated to Atmosphere from SXOS. Turned out that the error is not due to SXOS, but the XCI game file itself! But I'm glad I transitioned since SXOS is support is stagnant at firmware 11. **And I might update my sysNAND to latest firmware in the future to support online play for the first time.**

# TODO:
This is merely the first draft that I wrote, so I might keep on revising it if there are demands.
- Put all the tools/payload into this repo so you could easily install and link with my steps. (if there is a request)
- Include more screenshot


# Prelude on Why I update to Atmosphere from SXOS
- I first encountered the dreaded **Error 2002-4153** in the first few seconds when launching recent titles like New Pokemon Snap, Monster Hunter Rise, and Super Mario + Bowser's Fury. Many had pointed this error to a possible corrupted microSD card, so I format the microSD into FAT32 since some users reported that NS have a funky exFAT driver that sometimes caused corruption issues. With FAT32 I had to split my XCI into each files of 4GB using SAP tool. Even with it, I am still facing the same 20.xxx error when launching those recent titles.

After updating to Atmosphere, the same problem persist!
- I then suspect that it could be the Installer Manager that I used to install the NSP, which is Awoo-Installer at first. So I tried Tinfoil. Again the same error in the first few seconds on boot. To make sure my previous games that used to work in SXOS, I converted a couple of the XCI into NSP, and alas it works without freeze! So this could only means one thing...
- Turned out that the error is caused by the XCI that I downloaded from HBG using TotalCommander (FileManager) via rclone mount. To be safe, it is better to use `rclone copy ...` instead of a mounted file manager. I am still puzzled as to why this happen, and I am sure my HDD did not powered off unexpectedly the last time. I confirm that it is indeed the files because the error only occured on all the games I downloaded in the same day. Checking the checksums and I found that they are DIFFERENT! So my suspecious is correct! 
- **P/S: I was about to update the firmware on the emuNAND using Daydream as the last resort, glad I didn't do that and is able to pinpoint the issue!**


# Terminology
**Core Components**
`Hekete`: Bootloader to launch a payload. It allows root access and select which path to boot next, i.e. from your microSD card. Without this, the stock bootloader will always boot to SYSNAND (internal eMMC).
`Payload`:
`RCM`: Recovery mode to inject payload
`Chainload`: Loading payload by booting into Hekete bootloader first, i.e. you select `Payload` from `Hekete` menu option, instead of modifying the `payload.bin` file from your root microSD card.
`fusee primary`: Payload to boot into atmosphere. You can load this by modifying the SXOX `boot.ini` file or chainload from `Hekete`. 

**Homebrew App**
`Tinfoil`: Package manager
`Goldleaf`: Package manager
`Awoo`: Package manager
xxx: Save Manager


**File extensions**
`NRO`: Programs for Homebrew. These will appear on the Homebrew Menu without needing any further installation.
`NSP`: Container for games. Can hold the base, updates, and dlc
`NSZ`: A compressed format for NSP. NSZ files are typically 400MB less than NSP. It require installation via Installation Package Manager such as `Tinfoil`, `Goldbrew`, or `allos`.
`XCI`: Container for games. May be distributed as `trimmed XCI` which strip off all the empty/unused blocks in the storage. For example, for a flash cartridge with 32GB storage, if the game is only using 8GB, then the `trimmed XCI` will remove the remaining unused 24GB, and the final file size would be 8GB. This container is least compatible and difficult to work with to inject any updates or dlc. Recommend to use NSP instead. `XCI` can only be mounted as-is in SXOS (not Atmosphere), without needing further installation from the Switch via Installer Manager. 

# My setup
- NS Switch `Mariko` (a codename for NS switch sold with RED packaging box, with the longer battery life released after 2019), with soldered hardware mods SXOS Core chip.
- 512GB microSD card
- Backup sysNAND and emuNAND
- Never connected online in both emuNAND and sysNAND
- Only booted to sysNAND less than ten times to play my legit game. Again I never online in sysNAND or update the stock firmware.

Note:
- I don't have to force my NS Switch into recovery mode using TegraRCM nor connect any USB cable from my Switch to PC. This is only because I have the SXOS Core hardware mod soldered. You might have to do it if yours are not, e.g. you are using dongle from SXOS Pro or jig to boot into SXOS.


# 1. Setting Up for Atmosphere
Watch the amazing video by Sthetix on **MIGRATING FROM THE SX OS TO ATMOSPHERE** (https://www.sthetix.info/migrating-from-the-sx-os-to-atmosphere/)

My case is **Case#4 are for Patched Switch (Regular/Lite) with Emunand**. Simply jump to time **12:42** in that video and follow them EXACTLY till the end.

> TODO: I might put a screenshots for the steps in the video for reference. But as of the date of the writing, all his tools and steps are valid.


# 2. Launching into Atmosphere

There are two ways to boot into Atmosphere, and I chose the first one. Some claimed that booting via chainload (Hekate) is faster. From my experience, it take 30-60 seconds just to boot into Atmosphere and see the Homescreen using Method 1.

1. Directly from SXOS bootloader
**SXOS bootloader -> fusee_primary.bin (payload.bin) -> profit!**

2. Chainload via Hekate.
**SXOS bootloader -> hekete.bin (payload.bin) -> chainload (select fusee_primary.bin) from `/bootloader/payload`**

You should see three logos in succession before seeing the homescreen.
1. Indigo background with atmosphere logo
2. atmosphere triangle shaped logo
3. Official Nintendo Switch logo


# 3. Prepping Games
Since atmosphere only support NSP container, not XCI. I had to convert . In HBG, you'll find that not all titles are available in NSP format. A game is usually in one of the container, rather than both. So I have to convert all XCI into NSP.

## 3.1 Switch Army Knife (SAK)
I used **Switch Army Knife (SAK)** (https://github.com/dezem/SAK) that has a very good GUI with all the function I need to convert between XCI to NSP or vice versa. It basically bundle all the command line tools for different conversion into a program. Without this, I had to use script like **NSZ** (https://github.com/nicoboss/nsz) to convert. 

### 3.2 Dumping prod.keys
Before you can start to convert between XCI<->NSP using SAK, you have to dump the **FULL** set of keys `prod.keys` from your actual Switch console, using Lockpick_RCM ONLY. Lockpick NRO can only dump full key if you firmware is below 7.0.0, which is very unlikely, and mine is 10.0.2. 

I was using the `prod.keys` dumped from Lockpick NRO, and converting in SAK always shown `check your keys`. There is a YouTube video that claimed that the solution to this is to uncheck the **Read only** attribute of the file, but mine was already unchecked! So the culprit IS INDEED the invalid partial keys.

Note that Lockpick NRO (homebrew app) will only dump a subset of the keys. I verified these after comparing the `prod.keys` from Lockpick_RCM and Lockpick NRO, and the keys from Lockpick_RCM has more keys, and most importantly it has the `masterkey_00` to `masterkey_0a`, and not just the **derrived keys** dumped by Lockpick NRO.

See the following for comparison. The left pane is the `prod.keys` dumped from Lockpick NRO, the right pane is the `prod.keys` dumped from Lockpick_RCM. Notice how the right pane has more keys (highlighted in RED) that is missing from the left.

![](https://i.imgur.com/odnqywY.png)

![](https://i.imgur.com/rozXfBK.png)

To start dumping the keys,
1. Download `Lockpick_RCM.bin` (https://github.com/shchmue/Lockpick_RCM/releases/download/v1.9.2/Lockpick_RCM.bin) and put it to the root of your microSD card.

2. Open `boot.ini` at the root of your microSD card to the following.
```
[payload]
file=Lockpick_RCM.bin
```

You can also simply keep `boot.ini` content as is and rename `Lockpick_RCM.bin` to `payload.bin`. Both will work the same.

3. Insert back the microSD to your switch and power it on.
4. It should boot into Lockpick menu.
5. Hold your switch vertically. For navigation, use volume + or - to go up and down, and Power button to enter.
6. Select the first option, dump keys from sysNAND, then press Power button to execute.
7. It should display all keys dumped.
8. Power off, unplug the microSD and reinsert it to your PC.
9. Copy the `switch/prod.keys` to your `SAK\bin` folder. You will also find `partialaes.keys` there, but it is not used in SAK. **I would highly suggest you to backup these two keys and put it to your sysNAND FULL BACKUP!**
10. Launch SAK and start converting!


# 4. Install Games
Recall that Atmosphere cannot mount the XCI and won't detect them from Homebrew menu, so you have to use Installer to extract NSP to either your internal storage (Switch eMMC) or SD card. **Always choose SD card for the destination since the internal storage is supposed to only store files for your cleaned sysNAND and leave no trace of CFW!**

There are three popular installer to choose from:
- Tinfoil (https://tinfoil.io/Download)
- Goldleaf (https://github.com/XorTroll/Goldleaf)
- Awoo (https://github.com/Huntereb/Awoo-Installer)

Before installing your package manager, create a folder called **[NSP]** in your root microSD card where you will put all the NSP container games. 

> NOTE: You don't have to put all the NSP at the root microSD card unlike XCI mount in SXOS. So that should keep your root microSD card clean!

## 4.1 Package Manager 
### 4.1.1 Tinfoil
I used **Tinfoil v12 NRO**, not NSP!

Steps
1. Download the latest tinfoil from Tinfoil homepage (https://tinfoil.io/Home/Bounce/?url=https%3A%2F%2Ftinfoil.media%2Frepo%2Ftinfoil.latest.zip)
2. Copy the extracted folder to your microSD card. The `tinfoil` folder should be located in your `switch` folder.
3. Launch Tinfoil from atmosphere. If this is the first time launching it, I suggest launching Homebrew menu by launching any title then hold the R button, from there select Tinfoil. After that the Tinfoil icon should appear at your home menu. The next time you can just click it to launch directly.
4. Get familiarize with Tinfoil menu UI, then select File Browser > sdcard > NSP to see your NSP games!


## 4.2 Transfer via PC (Optional)
### 4.2.1 TinNUT
To simplify the transfer and not having to take out the microSD card from my Switch, I downloaded Tin NUT v2.7 (https://github.com/blawar/nut), which is a server program that you run on your PC. Once it is running, you can then launch Tinfoil, and select `File Browser -> usbfs:` to see all the drives on your PC! 

1. Download `nut.exe`, `tinfoil.reg` and `tinfoil_driver.exe`
2. Run `tinfoil_driver.exe` to install a modified USB driver to communicate with Switch when you plug in USB3 cable.
3. Double click `tinfoil.reg` to extend the MTP transfer timeout. This will add entry to your Windows registry, I've checked the content and it is harmless.
4. Run `nut.exe`, then paste the path on the top left field that contains all your NSP files!

![](https://i.imgur.com/zCkd2nW.png)


On my PC, each transfer is averaging 30-40MBps even when I am connected to USB 3.1 and using USB3 cable. I was expecting 80MBps but at least it works!

### 4.2.2 NS-USBloader v5.0
I also tried out NS-USBloader v5.0 (require Java runtime) (https://github.com/developersu/ns-usbloader), but unfortunately it doesn't work on Tinfoil. Clicking the `Upload` button always failed. I didn't investigate further since TinNUT works out of the box.

![](https://i.imgur.com/wp82v1x.png)


# 5. Post Install
## 5.1 Transfer saves in emuNAND into sysNAND for legit game
WIP

## 5.2 Update sysNAND to latest firmware for online play
WIP

# 6. Issues
**Blank screen when trying to wake up from idle screen (sleep mode).**
I had to force shutdown by long pressing the power button for more than 15 seconds to reboot.

Might try to chainload atmosphere from Hekate since some user reported that it work. Will update later.
