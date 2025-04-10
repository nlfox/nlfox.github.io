---
title: SteamOS VisualNovel Guide
description: Setup SteamDeck for best handheld VN experience. 
image: https://cdn.cloudflare.steamstatic.com/steamdeck/images/video/overview_oled.jpg
date: 2024-10-12
categories:
    - Gadgets
    - VisualNovel
tags:
    - steamOS
    - linux
    - archlinux
    - wine
    - proton
---

## Common issue & solution

### Install Proton-GE

Proton GE is community maintained Proton, provide better compatibility than original proton.

https://www.rockpapershotgun.com/how-to-install-proton-ge-on-the-steam-deck

Note: ProtonGE >= 9.10 fixed a lot of video related issue, upgrade and enjoy!

### Define Locale for a Game

First you need to add required locale to SteamOS.
You can use this script:
https://gist.github.com/XargonWan/cc660daf92c224b7241cbf5a2bf12c47


It's a bit old so I recommend run the command line by line instead of all at once.
This way in case of issue you won't mess things up.
After this you can put  `LANG=ja_JP.UTF-8 %command%` to launch configurations.

Potential other regions beside `ja_JP`:
```
zh_CN.UTF-8
zh_TW.UTF-8
```


### Game crash on Fullscreen mode

Due to some ancient DX API, some game may refuse to run in fullscreen mode.

Start the game in desktop mode first, if it's possible, set it to `window` or `broaderless window` mode.

If the game is too old and don't have a window mode, one useful tool is `DxWnd`

https://kingdoms.catsboard.com/t1522-how-to-use-dxwnd




### Font missing or weird 
The font is limited with built-in Proton font library.

If you notice the font doesn't looks correct in the game, try to add required font to the folder 

```
/home/deck/.local/share/Steam/steamapps/compatdata/GAME_ID_HERE/pfx/drive_c/windows/Fonts
```


Or if you are not sure what font was missing, just copy over the whole font folder (`C:\Windows\Fonts`) from windows computer.



### Dual boot windows on SteamDeck

Some game simply can't be run in Linux due to anti-cheat. So still worth to install a windows on steamdeck sometimes.

This guide explains most of it:
https://www.makeuseof.com/how-to-dual-boot-steam-deck-windows/

#### Dual boot manager

**Clover Boot Manager**  works great.

#### Steamdeck windows drivers

Refer to this official page for drivers: https://help.steampowered.com/en/faqs/view/6121-ECCD-D643-BAA8

#### Steamdeck sidebar on Windows

For steamdeck like sidebar experience, I recommand https://github.com/Valkirie/HandheldCompanion.

#### SteamInput for Desktop mode

I personally do not recommand it's desktop mode control tho, instead I recommand the one built-in steam: Steam on windows can define desktop layout, you can check the option below:

```
Steam -> Settings -> Controller -> Desktop Configuration 
```

to set it up. It will provide more flexibility.

#### Share your SteamLibrary with Linux

Given VNs are not performance sensitive, we can put them in TF card so that we can share between Linux and Windows.

##### NTFS

https://github.com/scawp/Steam-Deck.Mount-External-Drive

Above script should help auto mount shared drive between Windows and SteamOS.

Afther that, if you move your game library to TF card, then it can be read from both Windows and SteamOS seamlessly.

##### BTRFS

Since windows have a good BTRFS driver now, BTRFS might be another good option. It will have better performance compared to NTFS/exFAT.

Driver: https://github.com/maharmstone/btrfs

In steamOS, you can use same script as NTFS to auto load BTRFS https://github.com/scawp/Steam-Deck.Mount-External-Drive



## How to Debug 

Sometimes common fix won't help with the issue you are facing, below is some useful resource/mechanism to do specific game debugging.

### Where to look for info online

Usually, you are not along. Try search the issue on internet. There are multiple keywords can help you find the right resources:

```
<your issue> + Proton
<your issue> + Wine
```

Since proton base on wine, you can find wine related solution helps as well.

*protontricks* is the equivalent to *winetricks*.




Useful websites:

https://www.protondb.com/ (Steam games)

https://appdb.winehq.org/ (non-Steam games)

https://wiki.archlinux.org/ (SteamOS based on Archlinux)



### Enable debug log

Add `PROTON_LOG=1` to your game launch options in Steam will generate debug log in home location (`/home/deck`).

Search for useful log and search on internet.

