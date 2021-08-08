# Ultimate-Guide-to-Migrate-SXOS-SX-Core-to-Atmosphere

# Latest support: Firmware 12.1.0

# Migrate from SXOS 3.0.5 (SX Core hardware mod) to Atmosphere 0.19.3

I had been away from SXOS scene for more than a year, and recently just got back to power on my Switch to play some of the recent titles in 2021. I have prior experience in jailbreaking Nintendo 3DS, the new Nintendo 3DS, Sony Playstation PSP Slim, Sony Playstation Vita (phat), and Sony Playstation Vita (Slim). While the tools used differ for each console, they do share some similarities and I find it easier to understand and compare the process. 

Having said that, I decided to wrote this as a log to my attempts, and something that I could read back in the future. I wrote down each rationale for the steps so that you can better understand the process yourself. You may follow it if you have the same setup as I did! I think it is important to emphasize that is not a guide to compete with other awesome guides out there that covers all possible scenarios/console variants do achieve the same migration outcome. Rather this is simply written from my perspective in my scene experience with other console/handhelds, and on my scenario.

**NOTE: If you ever decide to follow my steps, please carefully read through all the steps, make sure all the release links are available, and understand what you are doing. And I do not hold responsible if you bricked your Switch console, which is very unlikely though.**

All the files I downloaded are the latest release that I could obtained as of May 29, 2021.

tldr: I successfully updated to Atmosphere from SXOS. Turned out that the error is not due to SXOS, but the XCI game file itself! But I'm glad I transitioned since SXOS is support is stagnant at firmware 11. **And I might update my sysNAND to latest firmware in the future to support online play for the first time.**


# TODO:
This is merely the first draft that I wrote, so I might keep on revising it if there are demands.
- [DONE] Put all the tools/payload into this repo so you could easily install and link with my steps. (if there is a request)
- Include more screenshot

# Common Traits for the Exploits
For historical purposes only.

Regardless on which exploit on which handhelds, the exploit process or outcome do share some similarities, in which they would:
- Replace/redirect the function of an official app to Homebrew Menu launcher, where you will then launch homebrew
- A title manager (package installer manager) to extract a game container into a destination (internal eMMC or external microSD card)
- The initial container format usually dominated or made popular by the first company that release the exploit publicly. In 3DS, Gateway was the first commercial company that did the exploit before a9lhax, and their Gateway cards support game container in `.3ds` format. With that format, you can simply mount it, then it would replace the official cartridge on the menu. You don't have to extract and install those using title manager. Similarly, TeamXcecutioner with their SXOS lineup was the first commercial company that promote `xci` format with their SXOS, with mounting capabilities without having to extract/install.
- The successor container (3ds: CIA / Switch: NSP) would require the use of a title manager (3ds: BigBlueMenu BBB / Switch: Tinfoil, Goldleaf, Awoo) to extract and install those titles. No mounting option is possible with this type of container. The pros is once installed, you can simply launch it directly from the home menu screen, without having to to to homebrew menu, select a game, go back to home menu, and select the first icon/cartridge. This is suitable for titles that you intend to play for long, and not simply testing it.


# Prelude on Why I update to Atmosphere from SXOS
- I first encountered the dreaded **Error 2002-4153** in the first few seconds when launching recent titles like New Pokemon Snap, Monster Hunter Rise, and Super Mario + Bowser's Fury. Many had pointed this error to a possible corrupted microSD card, so I format the microSD into FAT32 since some users reported that NS have a funky exFAT driver that sometimes caused corruption issues. With FAT32 I had to split my XCI into each files of 4GB using SAP tool. Even with it, I am still facing the same 20.xxx error when launching those recent titles.

After updating to Atmosphere, the same problem persist!
- I then suspect that it could be the Installer Manager that I used to install the NSP, which is Awoo-Installer at first. So I tried Tinfoil. Again the same error in the first few seconds on boot. To make sure my previous games that used to work in SXOS, I converted a couple of the XCI into NSP, and alas it works without freeze! So this could only means one thing...
- Turned out that the error is caused by the XCI that I downloaded from HBG using TotalCommander (FileManager) via rclone mount. To be safe, it is better to use `rclone copy ...` instead of a mounted file manager. I am still puzzled as to why this happen, and I am sure my HDD did not powered off unexpectedly the last time. I confirm that it is indeed the files because the error only occured on all the games I downloaded in the same day. Checking the checksums and I found that they are DIFFERENT! So my suspecious is correct! 
- **P/S: I was about to update the firmware on the emuNAND using Daydream as the last resort, glad I didn't do that and is able to pinpoint the issue!**


# Terminology
**Core Components**
- `Hekete`: The primary Bootloader that user (us) can access to launch a payload (next booting point) upon Tegra SOC power up. It allows root access and allow you to choose which environment to boot next, e.g. stock OFW from your sysMMC, CFW Atmosphere on your sysMMC or CFW Atmosphere on your emuMMC. The boot entries are defined in `hekate_ipl.ini` file (see the next one). Without this, the stock bootloader will always boot to SYSNAND (internal eMMC). IMO, calling it primary might be debatable to some embedded systems engineer, in the sense that the true primary or first stage bootloader is always internal chip ROM bootloader of Tegra SOC. 
- `hekate_ipl.ini`: Contains the boot entry to boot into different environment from Hekate (after initial payload injection via RCM or SXOS SX Gear). Boot entry are such as stock (OFW) on sysMMC (or sysNAND in 3DS scene), CFW (Atmosphere) on emuMMC (or emuNAND), CFW (Atmosphere) on sysMMC. As with all other .ini file, each boot option is in the `[option]` block, where you can define the boot flag, payload, icon for each booting option. All the possible settings can be found at the **Bootloader configuration** section in https://github.com/CTCaer/hekate. In Linux world, if you have dual boot Linux and Windows, you will know GRUB.
- `Payload`: A binary file to boot next from the first stage bootloader like `Hekete`. It could be another bootloader such as atmosphere `fusee primary`, or program like `Lockpick RCM` to dump your Switch console keys (full encryption/decryption).
- `RCM`: Recovery mode to inject payload
- `Chainload`: Loading payload by booting into Hekete bootloader first, i.e. you select `Payload` from `Hekete` menu option, instead of modifying the `payload.bin` file from your root microSD card.
- `fusee primary`: Payload (bootloader) to boot into atmosphere from injection (upon power on in RCM or SXOS). You can load this by modifying the SXOX `boot.ini` file. Make sure to use the right sigpatches in order to be able to install XCI file by package manager without error.
- `fusee secondary`: Payload (bootloader) to boot into atmosphere **ONLY FROM** Hekate AKA chainloading. You can **ONLY** load this by modifying the Hekate bootloader entry file `/bootloader/hekate_ipl.ini` file, particularly the `fss0` field. Make sure to use the different sigpatches made for Hekate+Atmosphere boot, and not using the one from Atmosphere. **Without the right sigpatches, you will 100% encounter XCI installation error when using package installer like Awoo!**
- `SX Gear`: Bootloader from SXOS developer Team Xecutor allowing us to boot into RCM mode directly to inject payload such as `Hekete` without having to use TegraRCM (jig to boot into recovery mode)
- `autoRCM`: Also known as auto Recovery Mode. This is a `soft brick` that always corrupt both `boot0` and `boot1` partitions so that the Switch never boot into Stock bootloader, which instead booting into Hekate or Atmosphere via payload injection. This effectively bypass fuse check, and prevent the fuse from being blown by stock booloader when it check its current fuse blown against the expected fuse to be blown for the booting firmware. **This is important because in the event for total battery drainage of the unpatched Switch or corruption, the system will boot into stock bootloader upon battery charging.** For Mariko patched Switch user however, this method is not applicable as Nintendo patched it, hence the `patched` switch. It is very likely that Mariko user would have SXOS Core SX hardware mod, which always ensure that it never got the chance to boot into stock bootlader, and it will always boot into the payload, thereby making this method unnecessary.


**Homebrew App**
- `Tinfoil, Goldleaf, Awoo`: Package manager to install game containers (NSP, NSZ, XCI). It is comparable this to BigBlueMenu (BBB) in 3DS scene
- `Edizon`, `jksv`, `Checkpoint`: Save Manager (for import/export/migration)


**File extensions**
- `NRO`: Programs for Homebrew. These will appear on the Homebrew Menu without needing any further installation.
- `NSP`: Container for games. Can hold the base, updates, and dlc. It has to be extracted and installe by title manager like Tinfoil. This is similar to the CIA format in 3DS, which you would then install using title manager like BigBlueMenu (BBB) 
- `NSZ`: A compressed format for NSP. NSZ files are typically 400MB less than NSP. It require installation via Installation Package Manager such as `Tinfoil`, `Goldbrew`, or `allos`.
- `XCI`: Container for games. May be distributed as `trimmed XCI` which strip off all the empty/unused blocks in the storage. For example, for a flash cartridge with 32GB storage, if the game is only using 8GB, then the `trimmed XCI` will remove the remaining unused 24GB, and the final file size would be 8GB. This container is least compatible and difficult to work with to inject any updates or dlc. Recommend to use NSP instead. `XCI` can only be mounted as-is in SXOS (not Atmosphere), without needing further installation from the Switch via Installer Manager. 

**Other**
- `Horizon OS`: Official Nintendo stock bootloader. Booting via this bootloader WILL always check for fuse count, and blow it to match the installed firmware. This is why unpatched Switch user need to make sure autoRCM is ON. 
- autoRCM:

## Comparing with 3DS Scene

Credits to `github/Electric1447` and `u/deleted` for the corrections.
| Aspect | Switch | 3DS |
| -------- | -------- | -------- | 
Game Cartridge Dump | .XCI/.XCZ (trimmed)     | .3DS      |
eShop Dump| .NSP/.NSZ (trimmed)    | .CIA
Homebrew Apps | .NRO/.NSP |.3DSX/CIA
Title Manager |FBI, BigBlueMenu (BBB), DevMenu | Tinfoil,Goldleaf,Awoo
First Commercial Company|TeamXcecutioner|Gateway|

# My setup
- NS Switch `Mariko` (a codename for NS switch sold with RED packaging box, with the longer battery life released after 2019), with soldered hardware mods SXOS Core chip.
- Both my sysNAND and emuNAND was on firmare 10.0.2
- 512GB microSD card
- Backup sysNAND and emuNAND
- Never connected online in both emuNAND and sysNAND
- Only booted to sysNAND less than ten times to play my legit game. Again I never online in sysNAND or update the stock firmware.

Note:
- I don't have to force my NS Switch into recovery mode using TegraRCM nor connect any USB cable from my Switch to PC, I don't think Mariko (patched or v2) Switch support that anyways. This is only because I have the SXOS Core hardware mod soldered. You might have to do it if yours are not, e.g. you are using dongle from SXOS Pro or jig to boot into SXOS.

# What You Will Have At the End of This Guide
By the end of going through **Section 1 to 3** this you would have
- An emuMMC (emuNAND) as a FILE. Partition is another option but I find it easy to backup a file that you can list without using Partition tool or Linux `dd` command.
- A choice to always boot into Hekate upon power on, then select the possible booting option (OFW Stock on sysMMC or CFW Atmosphere on emuMMC), or always boot into Atmosphere CFW upon power on.
- A full dump of the `prod.keys` and `partialaes.keys` that you can use to convert between different game container, or use it in Yuzu Switch emulator!
- A peace of mind that no sensitive information about your Switch will be sent to Nintendo Server should you ever connect to the Internet when in Atmosphere CFW. It will block any Internet connection to any of Nintendo servers via `exosphere`.

---

**!!! BEFORE YOU CONTINUE WITH MY STEPS, MAKE SURE TO CLONE MY ENTIRE GITHUB REPO https://github.com/dsync89/Ultimate-Guide-to-Migrate-SXOS-SX-Core-to-Atmosphere, THEN RUN THE CHECKSUM.SFV TO MAKE SURE ALL THE FILES ARE CORRECT AND NOT CORRUPTED !!!**

---

# About My Package
All the folder name are are linked to each of the section below, so 1-1 would means all the files you need to copy to your microSD root when doing **Step 1.1 Meet Hekate, your friendly Bootloader** and so on


# 1. Setting Up for Atmosphere
Watch the amazing video by Sthetix on **MIGRATING FROM THE SX OS TO ATMOSPHERE** (https://www.sthetix.info/migrating-from-the-sx-os-to-atmosphere/) to get familiarize with the steps. Then follow my steps below starting from 1.1!

My case is **Case#4 are for Patched Switch (Regular/Lite) with Emunand**. Simply jump to time **12:42** in that video and follow them EXACTLY till the end.

> TODO: I might put a screenshots for the steps in the video for reference. But as of the date of the writing, all his tools and steps are valid.

All the following steps and softwares are packaged, CRC checked, and adapted by me after watching Sthetix videos, and various articles from r/SwitchPirates and GBAtemp.

## 1.1 Meet Hekate, your friendly Bootloader
1. Remove microSD card from Switch console and insert to USB microSD Card reader to your PC
2. Copy and replace all files in `1-1 Meet Hekate, your friendly Bootloader` folder to the root of your microSD card.
3. Reinsert microSD card to your console
4. Boot it up and you should see Hekate boot screen.

## 1.2 Migrate your existing emuNAND (SXOS) to emuMMC (Optional)
> Note: Only do this if you want to carry your emuNAND in SXOS to Atmosphere, without starting from scratch and recreating one. Skip this if you want to start fresh. I would like to resume mine so I continue.
1. Skip the date/time if you want. It is only used to display the time in Nyx (the skin/theme you see in Hekate). It will not change your Switch system time.
2. Select `emuMMC` icon
3. Select `Migrate emuMMC` icon at the top right.
4. Select `emuNAND` button
5. Select `Continue` button
6. Select `OK` button when it is done
7. Select `Close` button at the top right
8. Select `Power Off` button at the bottom right.

## 1.3 Install Atmosphere to your microSD
1. Insert microSD card to your PC.
2. Copy and replace all files in `1-3 Install Atmosphere to your microSD` folder to the root of your microSD card.
3. Choose either **2.1a** or **2.1b** in the next section depending on how you would like to boot into.

# 2. Launching into Atmosphere

There are two ways to boot into Atmosphere, and I chose the **second one**. Some claimed that booting via chainload (Hekate) is faster, and I verified it. From my experience, it take 30-60 seconds just to boot into Atmosphere before you can see the Homescreen using Method 2.1a. It took less than 10 seconds using Method 2.1b.

> Edit on May 31, 2021: I used to choose method 1, but the waiting time is too long, and I want the option to boot into Stock OFW easily without having to press `Volume -` button every time I want.

Regardless on which method you use, you will know you are booting into Atmosphere CFW if you see three logos in succession before seeing the homescreen.
1. Indigo background with atmosphere logo
2. atmosphere triangle shaped logo
3. Official Nintendo Switch logo

## 2.1a Auto boot into Atmosphere, bypass Hekate
**Booting Flow: SXOS payload injection -> fusee_primary.bin (payload.bin) -> profit!**

> Note: This is the method used by Sthetix in his video on **MIGRATING FROM THE SX OS TO ATMOSPHERE** (https://www.sthetix.info/migrating-from-the-sx-os-to-atmosphere/)

Choose this method if you always wanted to boot into Atmosphere upon power on. You can still boot into Hekate, but would need to hold the `Volume -` button for 3 seconds when power the Switch on.

1. Copy and replace all files in `2-1a Auto boot into Atmosphere, bypass Hekate` folder to the root of your microSD card.
2. Insert microSD card back to Switch and power it on.
3. Go to toilet and get a cup of coffee. You deserve it! It will take at least 30 seconds to boot into the Nintendo home screen. 


## 2.1b Boot into Hekate first, then select boot mode AKA "Chain Load"
**Booting Flow: SXOS bootloader -> hekete.bin (payload.bin) -> Your boot entry (common entries are OFW Stock sysMMC, CFW on sysMMC or CFW on emuMMC)**

This is the most flexible method and you can always choose where to boot next, most common options are
- OFW Stock on sysMMC
- CFW on sysMMC
- CFW on emuNAND

I never intend to put CFW stuffs into my sysMMC since I would like to have a clean sysMMC to play my legit game online, with minimal chance for banning. So I only created two entries in the bootloader options.

1. Copy and replace all files in `2-1b Boot into Hekate first, then select boot mode` folder to the root of your microSD card.
2. Insert microSD card back to Switch and power it on.
3. Select `Launch` icon at the top left, and choose either one to boot next, OFW (sysMMC) or CFW Atmosphere (emuMMC).

# 3. Post Install
Do the following to prevent possible ban by connecting your Switch to the Internet while in CFW. They will help you to hide all your Switch info or block Internet connection to any of Nintendo servers, if you ever accidentally connect to the Internet while in CFW.

## 3.1 Blank prodinfo (prevent ban) using exosphere (if you use CFW on emuMMC)

## 3.2 Dumping prod.keys (Full Encryption/Decryption Keys on Your Switch)
Before you can start to convert between XCI<->NSP using SAK, you have to dump the **FULL** set of keys `prod.keys` from your actual Switch console, using Lockpick_RCM ONLY. Lockpick NRO can only dump full key if you firmware is below 7.0.0, which is very unlikely, and mine is 10.0.2. 

I was using the `prod.keys` dumped from Lockpick NRO, and converting in SAK always shown `check your keys`. There is a YouTube video that claimed that the solution to this is to uncheck the **Read only** attribute of the file, but mine was already unchecked! So the culprit IS INDEED the invalid partial keys.

Note that Lockpick NRO (homebrew app) will only dump a subset of the keys. I verified these after comparing the `prod.keys` from Lockpick_RCM and Lockpick NRO, and the keys from Lockpick_RCM has more keys, and most importantly it has the `masterkey_00` to `masterkey_0a`, and not just the **derrived keys** dumped by Lockpick NRO.

See the following for comparison. The left pane is the `prod.keys` dumped from Lockpick NRO, the right pane is the `prod.keys` dumped from Lockpick_RCM. Notice how the right pane has more keys (highlighted in RED) that is missing from the left.

![](https://i.imgur.com/odnqywY.png)

![](https://i.imgur.com/rozXfBK.png)

To start dumping the keys,

1. Copy and replace all files in `3-2 Dumping prod.keys (Full Encryption Decryption Keys on Your Switch)` folder to the root of your microSD card.
2. Insert microSD card back to Switch and power it on.
3. Select `Payload` icon from the center menu, and choose `Lockpick_RCM`.
4. Hold your switch vertically. For navigation, use volume + or - to go up and down, and Power button to enter.
5. Press `Power` button to start dumping all keys from sysMMC
6. Power Switch off
7. Reinsert microSD card to your PC
8. Copy and backup the full encryption/decryption keys found in `switch/partialaes.keys` and `switch/prod.keys`.

> **NOTE: I would highly suggest you to backup these two keys and put it to your sysNAND FULL BACKUP!**




Steps
1. Copy and replace all files in `3-1 Blank prodinfo (prevent ban) using exosphere (if you use CFW on emuMMC)` folder to the root of your microSD card.
2. Insert microSD card back to Switch and power it on.
3. All the stuffs are done under the hood when you connect to the Internet on CFW.

> Note: I never connect my Switch to Internet in both sysMMC or emuMMC, so this is just for extra peace of mind. I did this after reading a comment by `u/igromanru` on Reddit.


## 3.3 Incognito Mode
According to https://rentry.co/SettingUpIncognito this is not supported in "Mariko" switch, so I didn't do it.


# 4. Prepping Games
Since atmosphere cannot mount XCI directly like SXOS could, I had to convert the XCI from HBG (a paid membership access store with all the dumped games). In HBG, you'll find that not all titles are available in NSP format. A game is usually in one of the container, rather than both. So I have to convert all XCI into NSP.

Edit: Some Title Installers like `awoo` and `Tinfoil` can already extract and install XCI files, so you can use them instead of having to convert to NSP. I did this because I didn't know at first, but it is a good thing to learn.

## 4.1 Switch Army Knife (SAK)
I used **Switch Army Knife (SAK)** (https://github.com/dezem/SAK) that has a very good GUI with all the function I need to convert between XCI to NSP or vice versa. It basically bundle all the command line tools for different conversion into a program. Without this, I had to use script like **NSZ** (https://github.com/nicoboss/nsz) to convert. 

![](https://i.imgur.com/nmUWQ1P.png)


Before you can start to convert from any of the container, you would need to put your Switch `prod.keys` to the `bin` folder. Refer to **Section 3.2 Dumping prod.keys (Full Encryption/Decryption Keys on Your Switch)** to see how you can dump the key.

> Note: DO NOT USE any prod.keys you found circulating from the Internet. There's no risk there (need further confirmation) but why do that if you already hack your Switch and have full access?

Once you put the `prod.keys` to `bin` folder, you can then click any of the `x to y` button

> Note: Splitting NSP/XCI to FAT32 max file size limit (4GB) does not require `prod.keys`.


# 5. Install Games
Recall that Atmosphere cannot mount the XCI and won't detect them from Homebrew menu, so you have to use Installer to extract NSP to either your internal storage (Switch eMMC) or SD card. **Always choose SD card for the destination since the internal storage is supposed to only store files for your cleaned sysNAND and leave no trace of CFW!**

There are three popular installer to choose from:
- Tinfoil (https://tinfoil.io/Download)
- Goldleaf (https://github.com/XorTroll/Goldleaf)
- Awoo (https://github.com/Huntereb/Awoo-Installer)

Before installing your package manager, create a folder called **[NSP]** in your root microSD card where you will put all the NSP container games. 

> NOTE: You don't have to put all the NSP at the root microSD card unlike XCI mount in SXOS. Recall in SXOS you have to put all XCI games at the root microSD, otherwise the "Album" app would not detect the game. This is because the Title Installer will extract the NSP contents (which could have base, dlc, updates) into the emuMMC. So that should keep your root microSD card clean!

## 5.1 Package Manager 
### 5.1.1 Tinfoil
I used **Tinfoil v12 NRO**, not NSP!

Steps
1. Open download.txt from `5-1-1 Tinfoil Package Manager`. 
2. Paste the link to your browser and download it
3. Extract to your root microSD card. The `switch` folder should appear at the top most level.
4. Launch Tinfoil from atmosphere CFW. If this is the first time launching it, I suggest launching Homebrew menu by launching any title then hold the R button, from there select Tinfoil. After that the Tinfoil icon should appear at your home menu. The next time you can just click it to launch directly.
5. Get familiarize with Tinfoil menu UI, then select File Browser > sdcard > NSP to see your NSP games!

## 5.2 Transfer via PC (Optional)
### 5.2.1 TinNUT
**Edit @ 2021-June-01: Since I chose to boot to Atmosphere via Hekate and NOT using `fusee-primary` as the payload, Tinfoil will just show blank screen when launching via Homebrew Menu (with or without Applet) are the same. Of course without Tinfoil TinNUT would not work. It seems like it will only launch if you boot using `fusee-primary`. So for now I will choose Awoo and then transfer games to it via NS-USBloader v5.0 (see 5.2.2 for details).**

To simplify the transfer and not having to take out the microSD card from my Switch, I downloaded Tin NUT v2.7 (https://github.com/blawar/nut), which is a server program that you run on your PC. Once it is running, you can then launch Tinfoil, and select `File Browser -> usbfs:` to see all the drives on your PC! 

1. Copy all the files from `5-2-2 TinNUT (Server for Transferring Games from PC to Switch) to Tinfoil` to your PC
2. Run `tinfoil_driver.exe` to install a modified USB driver to communicate with Switch when you plug in USB3 cable.
3. Double click `tinfoil.reg` to extend the MTP transfer timeout. This will add entry to your Windows registry, I've checked the content and it is harmless.
4. Run `nut.exe`, then paste the path on the top left field that contains all your NSP files! This will open a web server that you can access from Tinfoilon your switch.

![](https://i.imgur.com/zCkd2nW.png)

Next, on your Switch,
5. Launch Tinfoil homebrew app.
6. Select [File Browser], then select `usbfs:`. You should see all your PC drives.
7. Select a folder containing all the games, then press A to start transfer. The download progress bar will show on the top right.

On my PC, each transfer is averaging 30-40MBps even when I am connected to USB 3.1 and using USB3 cable. I was expecting 80MBps but at least it works!

### 5.2.2 NS-USBloader v5.0
I also tried out NS-USBloader v5.0 (require Java runtime) (https://github.com/developersu/ns-usbloader), and it works flawlessly! 

You just have to select USB when launching `Awoo`, then launch `NS-USBloader`! I chose Awoo because Tinfoil failed to launch since I boot to Atmosphere via Hekate chainload, and not using `fusee-primary`.

![](https://i.imgur.com/wp82v1x.png)


# 6. Post Install
## 6.1 Transfer saves in emuNAND into sysNAND for legit game
WIP
Basically the gist is
1. Full backup sysMMC before connecting Online for the first time
2. Launch OFW on sysMMC (no trace of CFW ever) then connect to Internet
3. Reboot and do another full backup of the sysMMC. This is because some additional files will be written to sysMMC when you connect for the first tiem, possibly some files downloaded from Nintendo server. You want to always restore from this full backup in the future, not the first one, whenever you want to launch a clean OFW with your history of Internet Connection. If you restore from the full backup in step 1, there will be mismatch for the record when Nintendo matches the data. No one knows how and what files Nintendo server is checking, since their implementation is close source.

## 6.2 Update sysNAND to latest firmware for online play
WIP

# 7. Issues
**Blank screen when trying to wake up from idle screen (sleep mode).**
No longer happening.

**microSD card corrupted**
Many had pointed the culprit to the exFAT file system on the microSD card. I once forgot to power the unit off and unplug the microSD card. When booting again, my games become corrupted. So I had to format to FAT32 and will stick to it from now on and see if the problem reoccur. Fortunately I don't have to recreate my emuNAND, instead I just have to reinstall those NSPs via Awoo + USB!

**Lesson learnt: ALWAYS POWER YOUR SWITCH OFF BEFORE REMOVING THE MICROSD CARD!**

# 8. FAQ (Technical)
**Q: What is Fuse and should I care?**
A fuse is a one time programmable (write) hardware registers. The **one-time only** function is exactly the same as the fuse you will find in ALL power adapter, extension cord (power strip) or Power Supply Unit (PSU) on your PC. The lead inside the fuse will be shorted or break once in the event of very high surge of voltage/current flowing through. In other words, the fuse is the last line of defense that protect the surged volage to go through and damage any connected electronics.

The same logic applies here. Think of it as a storage somewhere in the powerful Tegra X1 SOC in Switch that store either '0' or '1'. Once that is written AKA blown/fused/burned, it will stay there permanently, hence the term **one-time**. However it can be read as many times as possible. It is very infeasible to modify the fused registers even for professional engineer.

Nintendo store a table somewhere in the firmware that contains the **expected** number of fuses to be burnt for each firmware version. The detail can be found here https://gist.github.com/jonluca/0d7ce7da7c84de5163be0b49b3e319cc. Though it is slightly outdated and only covers until firmware 10.x.

The fuse is only checked or burn by the stock Nintendo bootloader when you power on the console. It will never be burn once bootloader booted into the OS, either OFW (Horizon OS) or CFW (Atmosphere).

**Power on**
Assuming that you never hacked/mod your Switch and power it on, the stock (official Nintendo) bootloader will first read the number of fused burnt in the Tegra X1 SOC, then it will compare that number to the expected number of fuses to be burnt for the firmware to be loaded. There are **THREE** possible cases for the matching results:

> NOTE: Booting via payload injection such as hekate or atmosphere `fusee-primary` will **ALWAYS DISABLE FUSE CHECK**.

**Case 1:** If the number of burnt fuses is **LESS THAN** the expected number of fuse for the booted firmware, it will burn the remaining fuses until the total matches the expected number of fuses to be burnt for that firmware version. For example, if you are on Firmware 8.1.0, by default your unit would already have 10 fuses burnt (For retail unit). If you are previously updating the OFW firmware to 10.0.2 and this is the reboot,  the system will automatically burnt all the remaining 3 fuses, and this brings the total of burnt fuses to 10+3 = 13. 

**!!! IMPORTANT IMPORTANT IMPORTANT!!! IF YOU DONT WANT TO EVER ACCEIDENTALLY BURNT THE FUSE, BE SURE TO INJECT PAYLOAD TO HEKATE OR ATMOSPHERE FUSEE-PRIMARY (AND ENABLE AUTORCM IF YOU ARE USING UNPATCHED V1 CONSOLE, TO PREVENT THE SYSTEM TO EVER BOOT INTO STOCK BOOTLOADER)**

**Case 2:** If the number of burnt fuses is **EQUAL** to the expected number of burnt fuses of the booting firmware, it will not burn any fuse and proceed to boot. For example, if you are on firmware 10.0.2 and the number of burnt fuses is 13, it matches with the expected number for that firmware, i.e. 13. So it will not do anything.

**Case 3.** If the number of burnt fuses is **GREATER** to the expected number of burnt fuses of the firmware, it will refuse to proceed. The only way to boot into the system is using Hekate payload or Atmosphere `fusee-primary` payload that automatically bypass the fuse check. This case would only happen if you are trying to **DOWNGRADE** the firmware by booting into sysMMC with CFW installed, since you have to use Daybreak homebrew to downgrade and you would not be able to boot into OFW Horizon OS. For example, you are on firmware 10.0.2 (current burnt fuses number is 13), and are trying to downgrade to firmware 8.1.0 (expected burnt fuses number is 10).

---

Bootloader such as Hekate will bypass the fuse check, which is why you could use homebrew app like Daydream to downgrade/upgrade your firmware anytime you want.

There are only two ways that the fuse will be burnt when you boot up your Switch.

1) Via the stock/unmodded (never hacked) Nintendo Switch
2) Via the **Genuine Boot** option in SXOS Core.

Now the golden question, should you care about the burnt fuse?
You don't actually, **ONLY IF you DO NOT have plan to DOWNGRADE your system firmware on sysMMC.** Or for those OCD that wanted to keep their switch at their fresh retail state.


I'm still waiting for clarification from veteran whether booting Stock OFW from Hekate with the following bootentry (on `hekate_ipl.ini`) will bypass the fuse check when I boot into stock. So far I haven't boot into stock using that entry since I update to Atmosphere.
```
[OFW Stock (sysMMC)]
emummc_force_disable=1
fss0=atmosphere/fusee-secondary.bin
icon=bootloader/res/icon_switch.bmp
stock=1
```

---

**Q. What is `fusee-primary` and `fusee-secondary` in `atmosphere`?**
They are essentially payload that contains the bootloader code to boot into Atmosphere CFW.

You use `fusee-primary` whenever you want to automatically boot into Atmosphere. If you are on SXOS like me, you would get `SX Gear` and put both `boot.dat` and `boot.ini` in the root microSD. Then modify `boot.ini` payload to `fusee-primary.bin`, or simply rename `fusee-primary.bin` to `payload.bin`.

You use `fusee-secondary` **ONLY** in the first stage bootloader such as Hekate to boot into Atmosphere CFW. For example, if you are on SXOS like me, you would get `SX Gear` and put both `boot.dat` and `boot.ini` in the root microSD. Then modify `boot.ini` payload to `hekate_ctcaer_x.x.x.bin`, or simply rename `hekate_ctcaer_x.x.x.bin` to `payload.bin`. This will make sure your Switch boot into Hekate bootloader upon power up. You would then have boot entry or `/bootloader/hekate_ipl.ini` that have the `fusee-secondary.bin` as the fss0 path so that you could boot into Atmosphere from Hekate. For example:
```
[config]
autoboot=0
autoboot_list=0
bootwait=3
backlight=100
autohosoff=0
autonogc=1
updater2p=0
bootprotect=0

[CFW Atmosphere (emuMMC)]
emummcforce=1
fss0=atmosphere/fusee-secondary.bin
icon=bootloader/res/icon_payload.bmp

[OFW Stock (sysMMC)]
emummc_force_disable=1
fss0=atmosphere/fusee-secondary.bin
icon=bootloader/res/icon_switch.bmp
stock=1
```

---

**Q. Which package manager to use?**
**Case 1:
If you are using Atmosphere CFW:**

Tinfoil and Awoo are among the most popular ones. Based on my experience, I was getting `Corrupted data is found` when installing one container format over another. 

Following are my findings when installing different containers to Awoo via `NS-USBLoader`. The container are converted/merged using `SAK`. 


| Package Installer | NSZ (Base Game) | NSZ (Update/DLC ONLY) | NSP (Base Game) | NSP (Merged) | XCI (Base Game) | XCI (Merged) |
|-------------------|-----------------|-----------------------|-----------------|--------------|-----------------|--------------|
| Awoo              | Corrupted       | OK                    | OK              | Corrupted    | *E1             | Corrupted    |
| Tinfoil           |                 |                       |                 |              |                 |              |
- **Merged** above means `Base` + `Update` and/or `DLC`
- **E1**: `Partially installed contents can be removed ... OpenFileSystemWithId:124: Failed to open filesystem. Make sure your signature patches are up to date and set up properly. Error code: 0x00234c02`


**IMPORTANT**: If you boot to atmosphere via Hekate (i.e. using `fusee-secondary` as the payload), make sure you use the sigpatches downloaded from here (https://gbatemp.net/threads/sigpatches-for-atmosphere-hekate-fss0-fusee-primary-fusee-secondary.571543/). Otherwise error E1 **WILL** occur at the very early beginning when you try to install XCI game cartridge game dump via Awoo. The reason is it require different sigpatches (need clarification). It will install normally if you boot into atmosphere directly (i.e. using `fusee-primary` as the payload), since it has the right sigpatches that you previously installed.

> **NOTE: From my experience, the BEST and SAFEST way to install games via `Awoo` is to always install the base game NSP first. Then install the updates/dlc separately (it doesn't matter which container format, and I've tried NSZ directly and it works), instead of merging them into one container using SAK.**

**Case 2:
If you are using SXOS CFW:**

All XCI (base game only) and (base game + updates + dlc) can be mounted without problem on SXOS. However, some game that have a higher System Firmware requirement than the system current Firmware will prompt `Checking if software can be played...` and then asking to connect to the Internet. This is because using packaging installer like Tinfoil and Awoo will bypass the check. I never connected to the Internet so I am not sure whether connecting will help. Installing via Tinfoil might work too (not verified by me).






