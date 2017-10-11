# Creating a "UBports Experience" on x86 hardware

We won't need any Android parts when running on x86 hardware. This means we share the rootfs with the "normal" UBports images, however i don't think OTA updates apply. What needs to be done to create the same experience?

- [ ] Make Mir + Unity 8 (/Yunit) the default session
- [ ] Enable On-Screen-Keyboard as default
- [ ] Set up libertine for Xmir applications
- [ ] Enable Click-Packages? -> Enable Openstore by default?

The last point has some question marks as i don't know whether clicks will be the future once the move to 16.04 is done or whether snaps will then be the new packaging solution.



## Which rootfs to build upon?

There are **two options**: 

* 16.04 (+: same base as Ubports-next, LTS supported till April 2021 [source](https://wiki.ubuntu.com/Releases)) ( - : less up-to-date)
* 17.04 (+: up-to-date) ( - : only supported till April 2018, different base than UBports)

Why no 17.10? Because Unity 8 and it's successor Yunit don't ship with it anymore/yet so sticking to an LTS that's [being worked on supporting Yunit](https://yunit.io/yunit-packages-for-ubuntu-16-04-lts-xenial/) and that is also getting the [newest Mir stuff via PPA](http://voices.canonical.com/alan.griffiths/2017/08/07/mir-related-ppas/) might be the best choice >>>> **16.04 it is!**



## Downloading Tools and Preparing Image

<!-- ATTENTION: Yunit is currently only being built for 64-bit architectures so other probably won't work! -->

### Fetching the tools we need

1. Download the latest rootfs as .iso for our architecture @ [releases.ubuntu.com/16.04](http://releases.ubuntu.com/16.04/)

2. Head over to [linuxiumcomau.blogspot.de](http://linuxiumcomau.blogspot.de/) and get the great **isorespin** tool
3. Make isorespin executable with `chmod +x isorespin.sh`

### Integrate latest Mir + Yunit in iso
We want the latest and greatest so we will respin our ISO to include everything we need "out-of-the-box". This might be especially cool if someone doesn#t have a keyboard to attach to the tablet and still wants to login from the beginning.

#### Enable mir-staging PPA

```
sudo add-apt-repository ppa:mir-team/staging
sudo apt-get update
sudo apt-get upgrade

```
I'm not 100% shure we need MirAL too but if we do this also needs to be included:
```
sudo add-apt-repository ppa:alan-griffiths/miral-release
sudo apt-get update
sudo apt-get upgrade
```

#### Enable Yunit Source

```
wget -qO - https://archive.yunit.io/yunit.gpg.key | sudo apt-key add
echo 'deb [arch=amd64] http://archive.yunit.io/ubuntu/ xenial main' | sudo tee /etc/apt/sources.list.d/yunit.list
echo 'deb-src http://archive.yunit.io/ubuntu/ xenial main' | sudo tee --append /etc/apt/sources.list.d/yunit.list
sudo apt update
sudo apt upgrade
sudo apt install yunit-desktop
```

#### Enable Stable Phone Overlay PPA
I'm not shure but this seems to be some valuable things that might get the phone and desktop further into one line. Also Popescu Sorin recommended it on a [video showcasing unity 8 + mir on 16.04 (Nov 29, 2016)](https://www.youtube.com/watch?v=Dwxx2yQs_Ig).

```
sudo add-apt-repository ppa:ci-train-ppa-service/stable-phone-overlay
sudo apt-get update
```

**This needs to be changed to UBports Phone Overlay PPA** or whatever the UBports PPA is/will be called.

### Make yunit + mir the default desktop session

According to [this helpful askubuntu thread](https://askubuntu.com/questions/62833/how-do-i-change-the-default-session-for-when-using-auto-logins) we have to change the displaymanagers config (this is currently lightdm, after 17.04 it will be gdm!)

`sudo nano /etc/lightdm/lightdm.conf`

and change
`user-session=ubuntu`
to
`user-session=yunit`
(i guess?!)
also possible is something like
`user-session=unity8-desktop-session-mir`

For testing purposes, i will not change this now, only document how it would be done and do the switch manually when the iso is working via

`sudo /usr/lib/lightdm/lightdm-set-defaults -s <session-name>`

## Writing Image and Test on device
Okay, so we have created a fully pre-configured Ubuntu 16.04 image. To flash this onto our device we create a bootable USB medium, for example by [downloading Etcher](https://etcher.io/), extracting and running etcher-x.x.x.AppImage in the terminal. After choosing the linuxium-ubuntu-16.04-desktop-amd64.iso and our USB stick we execute the flashing. 
# Docu what i'm doing --- delete this later on

so  i downloaded 16.04.3 (http://releases.ubuntu.com/16.04/ubuntu-16.04.3-desktop-amd64.iso), and isorespin on August 13th, 2017
chmoded isorespin.sh

## First
run, went through GUI to execute it with options: 
" -i /home/demokrit/Downloads/making/ubuntu-17.04-desktop-amd64.iso --atom -r "ppa:mir-team/staging" -r "ppa:alan-griffiths/miral-release" -p "unity8-desktop-session-mir""

Okay, this crashed, cause looking into isorespin, "Adding repo" means a deb repo, not ppa. Therefore, will just run this as commands instead...

## Second
starting GUI, this time with different repos...

" -i /home/demokrit/Downloads/making/ubuntu-17.04-desktop-amd64.iso --atom -r "http://archive.yunit.io/ubuntu/ xenial main" -p "yunit-desktop" -c "sudo add-apt-repository ppa:mir-team/staging""

### Third

" -i /home/demokrit/Downloads/making/ubuntu-17.04-desktop-amd64.iso -r "deb [arch=amd64] http://archive.yunit.io/ubuntu/ xenial main" -p "yunit-desktop""

### Fourth 
this time we will not enadble the ppa nor yunit but just install the "old" unity8-desktop-session-mir

" -i /home/demokrit/Downloads/making/ubuntu-17.04-desktop-amd64.iso --atom -p "unity8-desktop-session-mir""

**This one worked**, unfortunately i accidentaly used 17.04 but doesn't matter right now.

A few points to note:
* Working on Chuwi Hi10 pro tablet with somewhat defunctional boot behavour (have to enter bios, disable Intel ME for next boot, then only i can boot anything)
* PROBLEM: sometimes the tablet has to be charged 100% to be able to enter BIOS
* 17.04 worked with unity 7 and unity 8 (for the latter i had to log off, click on the ubuntu icon and select "unity 8" and log back in with user "ubuntu", no passowrd in live session)
* PROBLEM: touchscreen of dock and screen are both in "horizontal mode" where to fix?
* PROBLEM: wifi is not working
* PROBLEM: battery status not working
* PROBLEM: touchscreen is not working

**Next up:**
Step 1:
* Testing with 16.04.3
* Enable Intel Cerry/Baytrail enhancemments from linuxium

Step 2:
* Enable new kernel

Step 3:
* Do all the things i wanted to do initially...

** Some links**
* [Rotating Screen with xrandr (CLI)](https://www.faqforge.com/linux/rotating-screen-in-ubuntu-and-linux-mint/)
* [Rotate Screen, Indicator Applet 16.04](http://www.omgubuntu.co.uk/2016/08/rotate-screen-ubuntu-16-04-indicator-applet)
* [How to install ubuntu on win10 tablet @ techradar, 21.04.2016](http://www.techradar.com/how-to/computing/how-to-install-ubuntu-onto-a-windows-tablet-1319489)
* [xInput Ubuntu Wiki Page](https://wiki.ubuntu.com/X/Config/Input) for identifiying Input devices

### Fifth aka Step 1

" -i /home/demokrit/Downloads/making/ubuntu-16.04.3-desktop-amd64.iso --atom -p "unity8-desktop-session-mir""

worked... flashing onto USB drive...

Okay, so

* It boots
* Unity 8 is not working out-of-the-box (when selecting unity 8 and trying to log in it just doesn't do anything after having "typed in the password", this by the way also happens on my regular laptop using unity 8 directly from the main repos)
* PROBLEM: Still no Wifi
* PROBLEM: Still no touchscreen


So i think most problems might be overcome with a more recent kernel (although 16.04.3 ships with 4.10!!) therefore we will move on and respin our iso, this time with the additional kernel upgrade...



### Sixth aka Step 2

" -i /home/demokrit/Downloads/making/ubuntu-16.04.3-desktop-amd64.iso --atom -u -p "unity8-desktop-session-mir""

worked...flashing onto USB drive...

* boots into lightdm or unitygreeter or whatever
* PROBLEM: doesn't take input except Power button
* FIX: Wifi is at least shown to work, couldn't try connecting since no input device...

**Next up**
* Delete unity8 from respun ISO and add it if unity7 on x comes up by using all the additional PPAs



### Seventh - a

" -i /home/demokrit/Downloads/making/ubuntu-16.04.3-desktop-amd64.iso --atom -u"

worked... flashed ... boots straight into unity 7 without login prompt .... changing screen rotation (clockwise) under settings>Displays

* PROBLEM: Touch not working
* FIX: Wifi is working
* FIX: Bluetooth is working

However, at this moment my USB drive seems to have had an interruption so my live session broke, little REISUB later...

### Seventh - b

trying the same image again, this time trying not to disconnect USB accidentaly...

```
sudo add-apt-repository ppa:mir-team/staging
sudo add-apt-repository ppa:alan-griffiths/miral-release
sudo add-apt-repository ppa:ci-train-ppa-service/stable-phone-overlay
sudo apt-get update
```
Encountering first error "(Appstreamcli:3700): CRITICAL Error while moving old database out of the way" so trying from [this soltion](https://askubuntu.com/questions/761592/unable-to-apt-get-dist-upgrade-on-a-persistent-ubuntu-16-04-usb)
`sudo chmod -R a+rX,u+w /var/cache/app-info/xapian/default`

it worked 
```
sudo apt-get update
sudo apt-get upgrade
```

This however fails again. Perhaps i need to add persistancy to the live-medium otherwise it will not be able to install those packages... *sigh*

### Eigth

" -i /home/demokrit/Downloads/making/ubuntu-16.04.3-desktop-amd64.iso --atom -u -s 2048MB"

worked...flashing to USB....okay, first of all there was now an strange own bootloader (refined bootloader or something), waiting 20 s or pressing Enter goes straight to ubuntu + unity7
```
sudo add-apt-repository ppa:mir-team/staging
sudo add-apt-repository ppa:alan-griffiths/miral-release
sudo add-apt-repository ppa:ci-train-ppa-service/stable-phone-overlay
sudo apt-get update
```
Again the appstream chache couldn't get updated....
```
sudo chmod -R a+rX,u+w /var/cache/app-info/xapian/default
sudo apt-get update
sudo apt-get upgrade
```
Again many errors occuring all something like 
```
dpkg: error processing archive xyz_new-version.deb (-unpack) unable to securely remove xyz_old-version.dpkg-temp: stale file handle
```

Some stupid repition gets a little further...
```
sudo dpkg --configure -a
sudo apt-get upgrade
```

I think most of these problems are caused because **this is a live cd** > Trying to install to a second USB drive attached to the dock right now.
* PROBLEM: installer crashed... don't have time for this now, giving up for the moment




### Alpha (Detour)

Okay, now i'm just trying out what happens on my old PC (amd64, ubuntu 16.04.1) which is a vanilla system besides all packages installed needed for porting ubports according to the wiki (+shutter screenshot tool).
```
sudo add-apt-repository ppa:mir-team/staging
sudo add-apt-repository ppa:alan-griffiths/miral-release
sudo add-apt-repository ppa:ci-train-ppa-service/stable-phone-overlay
sudo apt-get update
sudo apt-get upgrade
```

well this worked whch seems to support my hypothesis that errors were caused by the live-system.

```
sudo apt install unity8-desktop-session-mir
```

REBOOTING...

cannot login, let's me insert user & password then just does nothing

```
wget -qO - https://archive.yunit.io/yunit.gpg.key | sudo apt-key add
echo 'deb [arch=amd64] http://archive.yunit.io/ubuntu/ xenial main' | sudo tee /etc/apt/sources.list.d/yunit.list
echo 'deb-src http://archive.yunit.io/ubuntu/ xenial main' | sudo tee --append /etc/apt/sources.list.d/yunit.list
sudo apt update
sudo apt upgrade
sudo apt install yunit-desktop
```

doesn't work cause unmet dependencies... However logoff now has "yunit" instead of "unity8" as second option. Giving up for now...



### Beta (Detour 2)

So maybe we wll just respin for the old PC with yunit repos initialised plus yunit-desktop package included and nothing more...



" -i /home/demokrit/Downloads/making/ubuntu-16.04.3-desktop-amd64.iso -r "deb [arch=amd64] http://archive.yunit.io/ubuntu/ xenial main" -p "yunit-desktop""

(the deb-src is left out...)

failed cause the authentication (first step in yunit install where wget gets the key) is not performed in the script...



### Gamma (Detour 3)

Trying to just execute everything for adding yunit as "Add command" under "advanced" in isoresp.sh

" -i /home/demokrit/Downloads/making/ubuntu-16.04.3-desktop-amd64.iso -c "wget -qO - https://archive.yunit.io/yunit.gpg.key | sudo apt-key add" -c "echo 'deb [arch=amd64] http://archive.yunit.io/ubuntu/ xenial main' | sudo tee /etc/apt/sources.list.d/yunit.list" -c "echo 'deb-src http://archive.yunit.io/ubuntu/ xenial main' | sudo tee --append /etc/apt/sources.list.d/yunit.list" -c "sudo apt update" -c "sudo apt upgrade" -c "sudo apt install yunit-desktop""

isorespin seems to work... flashing to USB.... booting on PC....

So the yunit.list is inside /etc/apt/sources.list.d/ however running 

```
dpkg -s yunit-desktop
```

Says it is not installed nor is information available. It also seems to not have done the

```
sudo apt update
sudo apt upgrade
```

Since there are still all the yunit packages to install (and update has again Appstreamcli error)

```
sudo dpkg --configure -a
sudo chmod -R a+rX,u+w /var/cache/app-info/xapian/default
sudo apt update
sudo apt upgrade
```

This tells me that lot of things have been kept back... tacking that list (<list>) and coying it into

```
sudo apt install <list>
```

doesn't work, unmet dependencies... Now i'm really stopping :-(



### Epsilon (Detour 4)

Okay, this time i will just boot a vanilla ubuntu 16.04 amd64 on ye ol PC and do the yunit stuff, let's see what happens...

```
wget -qO - https://archive.yunit.io/yunit.gpg.key | sudo apt-key add
echo 'deb [arch=amd64] http://archive.yunit.io/ubuntu/ xenial main' | sudo tee /etc/apt/sources.list.d/yunit.list
echo 'deb-src http://archive.yunit.io/ubuntu/ xenial main' | sudo tee --append /etc/apt/sources.list.d/yunit.list
sudo apt update
sudo apt upgrade
sudo apt install yunit-desktop
```

