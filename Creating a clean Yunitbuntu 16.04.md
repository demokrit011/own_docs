## Creating a clean Yunitbuntu 16.04

##### Some links

* https://ubuntuforums.org/showthread.php?t=2372237&page=6&s=26e9dfcbc4f52294cff2f6b35f51dc03
* http://voices.canonical.com/alan.griffiths/2017/08/07/mir-related-ppas/
* https://launchpad.net/~ci-train-ppa-service/+archive/ubuntu/stable-phone-overlay
* https://launchpad.net/~ubports-developers/+archive/ubuntu/overlay
* https://launchpad.net/~khurshid-alam/+archive/ubuntu/yunit (however not for xenial, only later...)



### Goal and Reason

currently, there are some options to test Yunit on Debian/Ubuntu, however, there is no prebuilt iso out there that makes it easy for newbs (like me) to test it on real hardware... Since 16.04 is the current LTS and Ubports is targeting it as well, 16.04 seems the natural choice. What we want:

* Ubuntu 16.04.3 base with latest mainline (ubuntu patched) kernel
* no stuff not related to yunit/general subsystem
* latest yunit with working repos so update is possible and easy
* if possible, latest mir
* snaps/clicks from Ubports (Ubuntu Touch) and convergent stuff from KDE (kirigami, Plasma mobile)


#### Roadmap

1. Figure out what packages/configs are needed to run Ubuntu 16.04 + Yunit + phone apps
2. Build and Test top 20 apps + all core apps for x86_64
3. Repackage those as snaps
4. Test app-snaps on clean Ubuntu 16.04 VM + Ubuntu core 16 (RPi2)
5. Make plattform-snap and update app-snaps accordingly
6. Test this snap +  app-snaps on clean Ubuntu 16.04 VM + Ubuntu core 16 (RPi2)
7. Get minimal Ubuntu (server?) 16 and respin with additional needed packages 
8. Test this in VM and real hardware 
9. create CI script to automatically build from sources and configure corectly



### Comparing sources; Canonical vs. Ubports vs. Yunit repos

There are a few repos (some PPAs and deb repos) out there that provide the mir/ubuntu touc/unity8/yunit packages in different versions and the question is, which one to use... So where is the difference? As a reference, **all following infos are in respect to Xenial** therefore, the Yunit Test PPA by Khurshid Alam is not used but the [yunit deb-repos](https://yunit.io/yunit-packages-for-ubuntu-16-04-lts-xenial/) for 16.04 instead (see [package list here](http://archive.yunit.io/ubuntu/dists/xenial/main/source/Sources.gz))



|       Package Name       | Version in Yunit deb repos (130 pkgs) | Version in stable-phone overlay PPA (221 pkgs) | Version in ubports PPA (17 pkgs) |
| :----------------------: | :-----------------------------------: | :--------------------------------------: | :------------------------------: |
|      account-polld       |                                       |                                          |                                  |
| account-polld-plugins-go |                                       |                                          |                                  |
|       cmake-extras       |                                       |                                          |                                  |
|       debootstrap        |                                       |                                          |                                  |
|        dialer-app        |                                       |                                          |                                  |
|    gst-plugins-bad1.0    |                                       |                                          |                                  |
|  indicator-application   |                                       |                                          |                                  |
|                          |                                       |                                          |                                  |
|                          |                                       |                                          |                                  |
|                          |                                       |                                          |                                  |
|                          |                                       |                                          |                                  |
|                          |                                       |                                          |                                  |
|                          |                                       |                                          |                                  |
|                          |                                       |                                          |                                  |
|                          |                                       |                                          |                                  |
|                          |                                       |                                          |                                  |




### Testing what is needed for building & installing a phone app locally (usually as *.click but NOT here)

#### First Test System: Ubuntu 16.04.3 Unity7 + X11 based

##### Ubuntu-Clock-App

Starting with **Ubuntu 16.04.3 amd64** Unity 7, vanilla installation.... I think i read somewhere that i will need the [stable phone overlay](https://launchpad.net/~ci-train-ppa-service/+archive/ubuntu/stable-phone-overlay) and possibly the sdk so let's add both repos:

```
sudo add-apt-repository ppa:ubuntu-sdk-team/ppa
sudo add-apt-repository ppa:ci-train-ppa-service/stable-phone-overlay
sudo apt-get update
```

**Addendum 001: the first line is not needed, all necessary packages are inside the stable-phone-overlay repo!!**

We will now try to do one simple thing: get the sources from the calculator app, take the snapcraft.yaml and get all packages listed there as necessary for building and running and trying to build an run the app without snapping/click-packagin it.

First of all we need [the sources](http://bazaar.launchpad.net/~ubuntu-clock-dev/ubuntu-clock-app/trunk/files). For all needed packages, without using the [ubuntu-app-platform](https://launchpad.net/ubuntu-app-platform) snap, we have to look at the [snapcraft.yaml from before revision 497](http://bazaar.launchpad.net/~ubuntu-clock-dev/ubuntu-clock-app/trunk/view/496/snapcraft.yaml). This shows the following packages:

```
build-packages:
      - build-essential
      - cmake
      - gettext
      - intltool
      - ubuntu-touch-sounds
      - suru-icon-theme
      - qml-module-qttest
      - qml-module-qtsysteminfo
      - qml-module-qt-labs-settings
      - qtdeclarative5-u1db1.0
      - qtdeclarative5-qtmultimedia-plugin
      - qtdeclarative5-qtpositioning-plugin
      - qtdeclarative5-ubuntu-content1
      - qt5-default
      - qtbase5-dev
      - qtdeclarative5-dev
      - qtdeclarative5-dev-tools
      - qtdeclarative5-folderlistmodel-plugin
      - qtdeclarative5-ubuntu-ui-toolkit-plugin
      - xvfb
stage-packages:
      - ubuntu-sdk-libs
      - qtubuntu-desktop
      - qml-module-qtsysteminfo
```



Let's try this...

```
sudo apt install build-essential cmake gettext intltool ubuntu-touch-sounds suru-icon-theme qml-module-qttest qml-module-qtsysteminfo qml-module-qt-labs-settings qtdeclarative5-u1db1.0 qtdeclarative5-qtmultimedia-plugin qtdeclarative5-qtpositioning-plugin qtdeclarative5-ubuntu-content1 qt5-default qtbase5-dev qtdeclarative5-dev qtdeclarative5-dev-tools qtdeclarative5-folderlistmodel-plugin qtdeclarative5-ubuntu-ui-toolkit-plugin xvfb
```

this download all the build packages which should be necessary (~102 MB) now we only need to get the *stage-packages* (although qml-module-qtsysteminfo was already in buld-packages so only two left...) namely the [ubuntu-sdk-libs](https://packages.ubuntu.com/search?keywords=ubuntu-sdk-libs) and [qtubuntu-desktop](https://packages.ubuntu.com/search?suite=default&section=all&arch=any&keywords=qtubuntu-desktop&searchon=names).

```
sudo apt install ubuntu-sdk-libs qtubuntu-desktop
```

That's another 52.3 MB... and afterwards we **reboot**

Now let's move to our home directory and get the [ubuntu-clock-app sources](https://code.launchpad.net/~ubuntu-clock-dev/ubuntu-clock-app/trunk). Funny thing is bazaar is not installed by default on Ubuntu so we need to get this first...

```
cd ~
sudo apt install bzr
bzr branch lp:ubuntu-clock-app
```

we now move to the thusly created folder and invoke cmake with the parameters stated in the snapcraft.yaml

```
cd ubuntu-clock-app
cmake -DCMAKE_INSTALL_PREFIX=/usr
make -j4
make install
```

okay, this seemed to work, right until the end when it failed... Analysing some other time...

So first guess was that for **installing we need some privileges (sudo)** and this worked!

```
sudo make install
```

HOWEVER, i don't know how to start it -.- since the app does not appear in ```/usr/local/bin/``` nor can i just invoke ```ubuntu-clock-app``` form the terminal.

**Addendum 002: Upon doing this on a fresh VM with only the stable-phone-overlay PPA and using the exact cmake parameters from the snapcraft.yaml, everything works as follows**

```
cd ubuntu-clock-app
cmake -DCMAKE_INSTALL_PREFIX=/usr -DCLICK_MODE=off
make -j4
sudo make install
```

I can open the App via the *Dash* as "clock" or via the terminal using 

```
qmlscene $@ -I /usr/lib/x86_64-linux-gnu/qt5/qml/ClockApp  /usr/share/ubuntu-clock-app/ubuntu-clock-app.qml
```

which provides us with live-logging in our terminal. (I learned this by looking into the .desktop file of the clock app which you can do with ```less /usr/share/applications/ubuntu-clock-app.desktop``` and looking for the statement in the line beginning with "exec")

##### Openstore

We will be using the same VM as for the clock-app above with the exact same stuff installed as a first attempt.

This is normally utilizing **qmake instead of cmake** ([german reference article](http://www.linux-magazin.de/Ausgaben/2013/05/Qmake-vs.-Cmake)) and it is hosted on github instead of launchpad, which is why we need to install git (instead of bazaar aka bzr) and clone our repo

```
cd ~
sudo apt install git
git clone https://github.com/UbuntuOpenStore/openstore-app.git
cd openstore-app/
```

from some earlier tinkering and help by Brian Douglass i knwo we need a few other things:

```
sudo apt install libclick-0.4-dev ubuntu-sdk-qmake-extras
```

afterwards, we can just execute qmake:

```
qmake openstore.pro
make -j4
sudo make install
```

the ```make -j4``` will take a while and some *warnings* will occur, never mind that for now...

We can now start the Openstore:

```
cd /lib/x86_64-linux-gnu/bin/
./openstore
```

It works...**Hurray**

##### Guitar tools

Testing something totally different... Starting from same VM with packages installed from previous Ubuntu-Clock-App and OpenStore sections.

```
cd ~
bzr branch lp:guitar-tools
cd guitar-tools/
qmake guitar-tools.pro
make -j4
sudo make install
```

**Fail** in the "end" it exits with following error

> Project ERROR: Unknown module(s) in QT: multimedia
>
> Makefile:42: recipe for target 'sub-guitar-tools-make_first' failed

which indicates that some QT package concerning multimedia is missing. Doing a quick ```apt-cache search multimedia``` reveals (as one of many hits):

> libqt5multimedia5 - Qt 5 Multimedia module
>
> ....
>
> qtmultimedia5-dev - APIs for multimedia functionality - developement files

i guess it's one of those, HOWEVER; the first is already installed, maybe the second one?

```
sudo apt install qtmultimedia5-dev
sudo make install
```

This takes us to the next level, but still there are some errors... Will investigate some other time. The full logs is inside the logs folder...



##### InstantFX

Continuing with the exact same VM as in previous 3 attempts...

```
cd ~
bzr branch lp:instantfx
cd instantfx/
```

since we see no *.pro file but a CMakeLists.txt we can assume we need to use cmake again and thus i will follow the procedure as for the calculator:

```
cmake -DCMAKE_INSTALL_PREFIX=/usr -DCLICK_MODE=off
make -j4
sudo make install
```

This works as far as i can see, since there is one line stating

> -- Installing: /usr/bin/instantfx

i can just type in the terminal

```
instantfx
```

This works great, however it says there are currently no apps installed that can provide media to the app but it starts without errors and navigating around works like a charm.... However in the dash it also has no icon.

##### uNav

Continuing with the exact same VM as in previous 3 attempts... This is again hosted on launchpad

```
bzr branch lp:unav
cd unav
```

There is none of the previous files that indicate how to build (CMake or .pro) but there is a *Makefile* which probably means we can directly run *make*.

```
make install
```
This does not work and neither is 

```
make
```

but at least the latter does something (search for the set language which is the only thing happening)

Moving on as a first test this suffices for me noob ;-P...

##### UT Tweak Tool

Directly using the VM from the failed attempt building uNav, we'll clone UT Tweak Tool from launchpad and move into it's folder...

```
bzr branch lp:ubuntu-touch-tweak-tool/trunk
cd trunk
```

here we see a CMakeLists.txt so we'll try the cmake route. Inside the provided README, i states

> ### Dependencies
>
> Install development files:
> sudo apt-get install libpam0g-dev qtdeclarative5-gsettings1.0

so we install those previously to trying...

```
sudo apt install libpam0g-dev qtdeclarative5-gsettings1.0
```

these were actually not installed before so now everything should work...

```
cmake -DCMAKE_INSTALL_PREFIX=/usr -DCLICK_MODE=off
make -j4
sudo make install
```

the cmake console output told as that Click_Mode is not used by this app so that is not needed next time...

The Console output from ```make install``` on the other hand among other things stated that the .desktop file has been written to "//ut-tweak-tool.desktop" which is directly our root directroy.(This is by the way a mess and cannot be intentionally by the author, must have forgotten a variable... Ooops) By looking into this (```less /ut-tweak-tool.desktop```) we can see that the correct command to start the tool should be:

```
qmlscene $@ /share/qml/ut-tweak-tool/main.qml
```

(I inculded another "/" before the share which was not in the desktop file but is needed for this to "work")

This fails because some needed modules, namely

* StoreManager
* com.ubuntu.PamAuthentication

are missing... moving on for now

##### uMatrix

Taking the VM from above we next try uMAtrix as out-of-the-box experience... It tells a little on how you can build and deploy it on the phone but i will only take a few lines from that stuff which is that we need to use qmake

```
cd ~
git clone https://github.com/LarreaMikel/uMatriks.git
cd uMatriks
qmake uMatriks.pro
make -j4
```

This has an **fatal error** dum dum duuuummmm... moving on for now.

##### Gallery

This is another one that already has a snap file so chances are high that it works. I'll post the complete packages list and the thorough reader may notice that there are some that were not installed for the e.g. the clock-app so we may need a few of those...

```
build-packages:
      - cmake
      - gettext
      - intltool
      - python3
      - pkg-config
      - libexiv2-dev
      - libmediainfo-dev
      - qtbase5-dev
      - qtdeclarative5-dev
      - libexpat1-dev
      - zlib1g-dev
    stage-packages:
      - xdg-user-dirs
      - libmediainfo0v5
      - libzen0v5
      - libmms0
- libtinyxml2-2v5
```

The new ones in respect to the clock app are (directly including the command to install them):

```
sudo apt install pkg-config libexiv2-dev libmediainfo-dev libexpat1-dev zlib1g-dev
```

plus all of the stage-packages:

```
sudo apt install xdg-user-dirs libmediainfo0v5 libzen0v5 libmms0 libtinyxml2-2v5
```

Afterwards we can start cloning, building and deploying...

```
cd ~
git clone https://github.com/ubports/gallery-app.git
cd gallery-app
cmake -DCMAKE_INSTALL_PREFIX=/usr -DCLICK_MODE=off
make -j4
```

This fails at 55% at line 138 of the Makfile -.-



##### Beru

Last but not least for today: Beru from github, no snap yet but isn't as internally linked as some of the above so may work and uses cmake...

```
cd ~
git clone https://github.com/rschroll/beru.git
cd beru
cmake -DCMAKE_INSTALL_PREFIX=/usr -DCLICK_MODE=off
sudo make install
```

This fails and says **fatal error:** poppler/qt5/poppler-qt5.h: No such file or directory

So maybe we can install poppler?

```
sudo apt-cache search poppler
```

shows some results, even some with reference to qt5:

> libpoppler-qt5-1
>
> libpoppler-qt5-dev
>
> ...
>
> python3-poppler-qt5
>
> ...
>
> qtdeclarative5-poppler1.0

since the last is about the poppler QML plugin and UT is som much about QML we shall start with that...

```
sudo apt install qtdeclarative5-poppler1.0
```

this is already installed... moving on

```
sudo apt install libpoppler-qt5-dev
sudo make install
```

**This was it!!** hurray.. let's try to start this thing...

```
cd ..
beru
```

This tries to start it, however, beru seems to be built atop oxide (something like a browser container of chrome/ium AFAIK) which is not found -.- This might work better with Yunit since this should have oxide in it's repos...

##### Resumee

So we saw that a few apps can be build if we know which dependencies they have which is easy for all already snapped apps. For all others, we need to figure out a way to find out what is needed. Also we do not know whether the content-hub and everything like camera and else will work with these locally build apps. For now, we will switch to a Yunit based system to see what we can learn from there. For the future, we should do the following: 

1. create a table with all Apps we want to test (more or less done in Click2Snap docu)
2. insert URL to sources in that table (done)
3. insert existing snapcraft.yaml into that table (should make it an xml with links not a stupid .md table)
4. insert dependencies we found out by other means
5. insert manifest from clicks
6. try to automatise all this and make a script that does all our work

Concerning finding out dependencies: what i did in the above was very dirty, a cleaner method would be to create a clean Ubuntu 16.04.3 with Unity7, X11 and acivated Overlay PPA and then create a snapshot of this. Afterwards, each app should be checked what dependencies are really needed to make it build and in a second step to make it work. A list of those should be written to a file, the log should be saved and the VM should return to the snapshot and try the next app. Also this might be done the exact other way round, installing all dependencies from all currently available already snapped mobile apps, see what apps build and then remove dependencies one by one and see which apps fail at what point. However both is the brute-force method, best would be some way to know from the code what is needed...

#### Ubuntu 16.04 server image + Yunit + mir

##### Setting up the VM

** Step 01: Pristine Yunit + Mir + Ubuntu 16.04**

So this time we get the latest 16.04.3-server 64-bits image, create a VM with that, activate our Yunit repos, download Yunit and reboot...
```
wget http://releases.ubuntu.com/16.04/ubuntu-16.04.3-server-amd64.iso
sha256sum ubuntu-16.04.3-server-amd64.iso
```
You may compare the sha256sum with the correct one [here](http://releases.ubuntu.com/16.04/SHA256SUMS) and move on if correct, otherwise repeat the download.

Create a VM (i use dynamically allocated VDI disk with 30 GB maximum space) and putting the downloaded iso into the drives... we will install Ubuntu server with the HWE kernel, enable automatic security updates and leave everything else to default. (maybe including OpenSSH server might be usefull but we can install it later on if needed)

Okay, i am writing this just as i am doing it but i wanted to install yunit next however copy-paste between host and guest VM does not work very well so we will install and set up openssh first ;-P

**Inside the VM**

```
sudo apt install openssh-server
```

shutdown the VM...

**Outside of the VM**

we go to the settings of the VM (in Virtualbox that is), navigate to network -> NAT (adapter 1), open the additional feature and go to "port forwarding" and add a new rule:

| Name | Protocoll | HOST-IP | HOST-port | GUEST-IP | GUEST-port |
| ---- | --------- | ------- | --------- | -------- | ---------- |
| ssh  | TCP       |         | 3022      |          | 22         |

Now we start the VM again and open a terminal in the host machine. Only the login via ssh (so the next command) is *purely done on the host terminal* everything else is the same if we do it in or outside. (my user is named yunit)

```
ssh -p 3022 yunit@127.0.0.1 
```

*Sidenote: if you have done this with already, it may tell you that the Host identification has changed and provide you with the solution how to move on, i.e. the ssh-keygen... command*

Now we can set up Yunit

```
wget -qO - https://archive.yunit.io/yunit.gpg.key | sudo apt-key add
echo 'deb [arch=amd64] http://archive.yunit.io/ubuntu/ xenial main' | sudo tee /etc/apt/sources.list.d/yunit.list
echo 'deb-src http://archive.yunit.io/ubuntu/ xenial main' | sudo tee --append /etc/apt/sources.list.d/yunit.list
sudo apt update
sudo apt upgrade
sudo apt install yunit-desktop
```

Then we will restart the VM and see whether yunit will come up... If so, we have our first snapshot "Yunitbuntu 16.04.3_amd64 pristine" (since we did not add the Overlay PPA yet which we will do in the next step before our first work-with-snapshot, up to this part is the first log of this session)

Nice, it started! Be aware though, that **it will change keyboard layout to english** (for those that have a QWERTZ or other layout!)

A quick note what is there: not much, Terminal, Settings, Browser, Music and Videos Scope and some utilities. Vim seems to be preinstalled but does not start from the dash. Since i have no wish to actually run it i will move on. Also sound is not working...

Shutting down the VM and creating a snapshot. (logs/yunitbuntu_16-04-3_amd64_setup_pristine.log)

**Step 02: Adding OVerlay PPA, making working Snapshot**

see (logs/yunitbuntu_16-04-3_amd64_adding_overlay-PPA.log) for procedure...

shutdown VM and create snapshot

##### Intermezzo: Dirty Quickshot
**this is logged as FREAKrun = install ALL the packages and test core apps + x THEN return to snapshot**

so we will install
1. all packages inside the ubuntu-plattform-snap
2. all packages from previous snapcraft.yamls

So let's begin with the [ubuntu-platform-snap](https://git.launchpad.net/ubuntu-app-platform/tree/snapcraft.yaml) which states the following:
```
stage-packages:
        - fontconfig
        - libc6
        - libcups2
        - libdbus-1-3
        - libdrm2
        - libegl1-mesa
        - libfontconfig1
        - libfreetype6
        - libgbm1
        - libgcc1
        - libgl1-mesa-dev
        - libgl1-mesa-glx
        - libgles2-mesa-dev
        - libglib2.0-0
        - libglu1-mesa-dev
        - libharfbuzz0b
        - libice6
        - libicu55 ##
        - libinput10
        - libjpeg8
        - libmtdev1
        - libpcre16-3
        - libpng16-16
        - libproxy1v5
        - libsm6
        - libsqlite3-0
        - libstdc++6
        - libudev1
        - libx11-6
        - libx11-xcb1
        - libxcb1
        - libxcb-glx0
        - libxcb-icccm4
        - libxcb-image0
        - libxcb-keysyms1
        - libxcb-randr0
        - libxcb-render0
        - libxcb-render-util0
        - libxcb-shape0
        - libxcb-shm0
        - libxcb-sync1
        - libxcb-xfixes0
        - libxcb-xkb1
        - libxext-dev
        - libxi6
        - libxkbcommon0
        - libxkbcommon-x11-0
        - libxrender1
        - perl
        - zlib1g
        # workaround for LP: #1576282
        - locales-all
        # qtmultimedia
        - libpulse0
        # ubuntu-sdk-libs
        - ubuntu-sdk-libs
        # Extra UI components
        - qtdeclarative5-ubuntu-ui-extras0.2
        # Calendar deps
        - qtcontact5-galera
        - qtorganizer5-eds
        # Web apps
        - liboxideqt-qmlplugin
        - webapp-container
        # Allow non-memory GSettings backend
        - dconf-gsettings-backend
        # Mir QPA
        - qtubuntu-desktop
        # Additional deps
        - qml-module-ubuntu-thumbnailer0.1
        # Needed by address-book-app. See bug 1643660
        - qtdeclarative5-gsettings1.0
        - qtdeclarative5-ofono0.2
        - qtdeclarative5-ubuntu-keyboard-extensions0.1
        - qtdeclarative5-ubuntu-telephony-phonenumber0.1
        - qtdeclarative5-buteo-syncfw0.1
        - qml-module-qtcontacts # See bug 1643659
        # Needed by telegram.
        - qml-module-ubuntu-connectivity
        - qtdeclarative5-ubuntu-contacts0.1
        # Add a droid derived Sans-Serif style CJK font
        - fonts-wqy-microhei
        # Unity7 desktop integration (menus and indicators with snap support)
        - appmenu-qt5
        # OSK
        - ubuntu-keyboard-arabic
        - ubuntu-keyboard-autopilot
        - ubuntu-keyboard-azerbaijani
        - ubuntu-keyboard-bosnian
        - ubuntu-keyboard-catalan
        - ubuntu-keyboard-chinese-chewing
        - ubuntu-keyboard-chinese-pinyin
        - ubuntu-keyboard-croatian
        - ubuntu-keyboard-czech
        - ubuntu-keyboard-danish
        - ubuntu-keyboard-dev
        - ubuntu-keyboard-dutch
        - ubuntu-keyboard-emoji
        - ubuntu-keyboard-english
        - ubuntu-keyboard-esperanto
        - ubuntu-keyboard-finnish
        - ubuntu-keyboard-french
        - ubuntu-keyboard-german
        - ubuntu-keyboard-greek
        - ubuntu-keyboard-hebrew
        - ubuntu-keyboard-hungarian
        - ubuntu-keyboard-icelandic
        - ubuntu-keyboard-italian
        - ubuntu-keyboard-japanese
        - ubuntu-keyboard-korean
        - ubuntu-keyboard-latvian
        - ubuntu-keyboard-norwegian-bokmal
        - ubuntu-keyboard-polish
        - ubuntu-keyboard-portuguese
        - ubuntu-keyboard-romanian
        - ubuntu-keyboard-russian
        - ubuntu-keyboard-scottish-gaelic
        - ubuntu-keyboard-serbian
        - ubuntu-keyboard-slovenian
        - ubuntu-keyboard-spanish
        - ubuntu-keyboard-swedish
        - ubuntu-keyboard-ukrainian
```

so to make this into a command after having started the VM:

```
sudo apt install fontconfig libc6 libcups2 libdbus-1-3 libdrm2 libegl1-mesa libfontconfig1 libfreetype6 libgbm1 libgcc1 libgl1-mesa-dev libgl1-mesa-glx libgles2-mesa-dev libglib2.0-0 libglu1-mesa-dev libharfbuzz0b libice6 libicu55 ## libinput10 libjpeg8 libmtdev1 libpcre16-3 libpng16-16 libproxy1v5 libsm6 libsqlite3-0 libstdc++6 libudev1 libx11-6 libx11-xcb1 libxcb1 libxcb-glx0 libxcb-icccm4 libxcb-image0 libxcb-keysyms1 libxcb-randr0 libxcb-render0 libxcb-render-util0 libxcb-shape0 libxcb-shm0 libxcb-sync1 libxcb-xfixes0 libxcb-xkb1 libxext-dev libxi6 libxkbcommon0 libxkbcommon-x11-0 libxrender1 perl zlib1g locales-all libpulse0 ubuntu-sdk-libs qtdeclarative5-ubuntu-ui-extras0.2 qtcontact5-galera qtorganizer5-eds liboxideqt-qmlplugin webapp-container dconf-gsettings-backend qtubuntu-desktop qml-module-ubuntu-thumbnailer0.1 qtdeclarative5-gsettings1.0 qtdeclarative5-ofono0.2 qtdeclarative5-ubuntu-keyboard-extensions0.1 qtdeclarative5-ubuntu-telephony-phonenumber0.1 qtdeclarative5-buteo-syncfw0.1 qml-module-qtcontacts # See bug 1643659 qml-module-ubuntu-connectivity qtdeclarative5-ubuntu-contacts0.1 fonts-wqy-microhei appmenu-qt5 ubuntu-keyboard-arabic ubuntu-keyboard-autopilot ubuntu-keyboard-azerbaijani ubuntu-keyboard-bosnian ubuntu-keyboard-catalan ubuntu-keyboard-chinese-chewing ubuntu-keyboard-chinese-pinyin ubuntu-keyboard-croatian ubuntu-keyboard-czech ubuntu-keyboard-danish ubuntu-keyboard-dev ubuntu-keyboard-dutch ubuntu-keyboard-emoji ubuntu-keyboard-english ubuntu-keyboard-esperanto ubuntu-keyboard-finnish ubuntu-keyboard-french ubuntu-keyboard-german ubuntu-keyboard-greek ubuntu-keyboard-hebrew ubuntu-keyboard-hungarian ubuntu-keyboard-icelandic ubuntu-keyboard-italian ubuntu-keyboard-japanese ubuntu-keyboard-korean ubuntu-keyboard-latvian ubuntu-keyboard-norwegian-bokmal ubuntu-keyboard-polish ubuntu-keyboard-portuguese ubuntu-keyboard-romanian ubuntu-keyboard-russian ubuntu-keyboard-scottish-gaelic ubuntu-keyboard-serbian ubuntu-keyboard-slovenian ubuntu-keyboard-spanish ubuntu-keyboard-swedish ubuntu-keyboard-ukrainian
```

and from all previous attempts we include the following


```
sudo apt install build-essential cmake gettext intltool ubuntu-touch-sounds suru-icon-theme qml-module-qttest qml-module-qtsysteminfo qml-module-qt-labs-settings qtdeclarative5-u1db1.0 qtdeclarative5-qtmultimedia-plugin qtdeclarative5-qtpositioning-plugin qtdeclarative5-ubuntu-content1 qt5-default qtbase5-dev qtdeclarative5-dev qtdeclarative5-dev-tools qtdeclarative5-folderlistmodel-plugin qtdeclarative5-ubuntu-ui-toolkit-plugin xvfb ubuntu-sdk-libs qtubuntu-desktop bzr git ubuntu-sdk-qmake-extras libclick-0.4-dev qtmultimedia5-dev pkg-config libexiv2-dev libmediainfo-dev libexpat1-dev zlib1g-dev xdg-user-dirs libmediainfo0v5 libzen0v5 libmms0 libtinyxml2-2v5 libpoppler-qt5-dev
```

okay, if this starts up after reboot we will try to build some apps, this is crazy but cool xD

...

so i am a little bit astonished, it still works xD

From here on, we will now clone [every app that has the Tag "core app"](https://github.com/search?q=topic%3Acore-app+org%3Aubports+fork%3Atrue) on ubports (19 in total) and see whether they build. We'll at least create a "builddir" to work in so it doesn't completely get out of hand (the chaos)

```
mkdir builddir
cd builddir
git clone https://github.com/ubports/telegram-app.git
git clone https://github.com/ubports/camera-app.git
git clone https://github.com/ubports/filemanager-app.git
# git clone https://github.com/ubports/terminal-app.git
git clone https://github.com/ubports/docviewer-app.git
git clone https://github.com/ubports/ubports-app.git
git clone https://github.com/ubports/address-book-app.git
git clone https://github.com/ubports/clock-app.git
git clone https://github.com/ubports/dialer-app.git
# git clone https://github.com/ubports/wheather-app.git
git clone https://github.com/ubports/calculator-app.git
git clone https://github.com/ubports/gallery-app.git
git clone https://github.com/ubports/mediaplayer-app.git
git clone https://github.com/ubports/sudoku-app.git
git clone https://github.com/ubports/messaging-app.git
# git clone https://github.com/ubports/ubuntu-printing-app.git
git clone https://github.com/ubports/notes-app.git
# git clone https://github.com/ubports/webbrowser-app.git
git clone https://github.com/ubports/calendar-app.git
```

out of these, only Terminal and Webbrowser are preinstalled, not shure about ubuntu-printing though... (there is something like that and i thought i saw it upon setup, not so important now...) we'll leave them out for now, also for the wheather -app it wants to know the github account so we'll skip that one too. 

Since we are at it, before going any further we will also clone a few more apps, archive all of them and copy them to the host so we don't have to do this anytime in the future we want to repeat something similar... but we will do this in a subdirectory

```
mkdir community-apps
cd community-apps
git clone https://github.com/LarreaMikel/uMatriks.git
git clone https://github.com/rschroll/beru.git
bzr branch lp:ubuntu-touch-tweak-tool/trunk
bzr branch lp:unav
bzr branch lp:instantfx
bzr branch lp:guitar-tools
git clone https://github.com/UbuntuOpenStore/openstore-app.git
git clone https://github.com/bhdouglass/falcon.git
git clone https://github.com/mzanetti/kodimote.git
git clone https://github.com/loqui/im.git
git clone https://github.com/scummvm/scummvm.git
git clone https://github.com/doniks/shorter.git
bzr branch lp:tagger 
git clone https://github.com/syncthing/syncthing.git
bzr branch lp:timer 
bzr branch lp:~mzanetti/+junk/ubtd 
```

So we also use ssh to copy those files to our host machine, adapting [this documentation](https://help.ubuntu.com/community/SSH/TransferFiles) to our needs.

*In host terminal, not logged into VM* (this will copy recursively the folder builddir to the current folder in the host terminal beware the Capital -P for scp instead of minor -p for ssh)

```
scp -P 3022 -r yunit@127.0.0.1:~/builddir/ .
```

So now we have (if we take into account that we left out the preinstalled) **30 apps to test with** i think this should suffice...

**Testing the ... out of it**

*Clock*

```
cd ..
cd clock-app
cmake -DCMAKE_INSTALL_PREFIX=/usr -DCLICK_MODE=off
make -j4
sudo make install
```

builds...

```
qmlscene $@ -I /usr/lib/x86_64-linux-gnu/qt5/qml/ClockApp  /usr/share/ubuntu-clock-app/ubuntu-clock-app.qml
```

won't work:

> QXcbConnection: Could not connect to display 
> Aborted (core dumped)

From Brian Douglass i heard that the following might work 

```
QT_QPA_PLATFORM=mirserver
# and/or
MIR_SOCKET=/path/to/something
```

the path is found by ```sudo find / -name 'mir_socket'``` 

> /run/mir_socket
> /run/user/1000/mir_socket

 

And indeed if we run

```
QT_QPA_PLATFORM=mirserver qmlscene $@ -I /usr/lib/x86_64-linux-gnu/qt5/qml/ClockApp  /usr/share/ubuntu-clock-app/ubuntu-clock-app.qml
```

we get a different error:

> [2017-11-16 22:51:45.230081] mirplatform: Found graphics driver: mir:mesa-kms (version 0.26.3)
> [2017-11-16 22:51:45.230567] mirplatform: Found graphics driver: mir:mesa-x11 (version 0.26.3)
> [2017-11-16 22:51:45.231708] mirserver: Starting
> [2017-11-16 22:51:45.232281] mircommon: Loading modules from: /usr/lib/x86_64-linux-gnu/mir/server-platform
> [2017-11-16 22:51:45.232585] mircommon: Loading module: /usr/lib/x86_64-linux-gnu/mir/server-platform/graphics-mesa-kms.so.12
> [2017-11-16 22:51:45.232865] mircommon: Loading module: /usr/lib/x86_64-linux-gnu/mir/server-platform/server-mesa-x11.so.12
> [2017-11-16 22:51:45.233890] mircommon: Loading module: /usr/lib/x86_64-linux-gnu/mir/server-platform/input-evdev.so.6
> [2017-11-16 22:51:45.236290] mirplatform: Found graphics driver: mir:mesa-kms (version 0.26.3)
> [2017-11-16 22:51:45.236786] mirplatform: Found graphics driver: mir:mesa-x11 (version 0.26.3)
> [2017-11-16 22:51:45.237314] mirserver: Selected driver: mir:mesa-kms (version 0.26.3)
> Exception while creating graphics platform
> ERROR: /build/mir-O8_xaj/mir-0.26.3+16.04.20170605/src/platforms/mesa/server/kms/linux_virtual_terminal.cpp(220): Throw in function int mir::graphics::mesa::LinuxVirtualTerminal::find_active_vt_number()
> Dynamic exception type: boost::exception_detail::clone_impl<boost::exception_detail::error_info_injector<std::runtime_error> >
> std::exception::what: Failed to find the current VT

We'll have to ask someone else or search the web. However i have to do some real life stuff now... So just to test whether this works with another app, let's say the ubports-app...



*ubports-app*

```
cd ~/builddir/ubports-app
cmake -DCMAKE_INSTALL_PREFIX=/usr -DCLICK_MODE=off
make -j4
sudo make install
```

works... going to desktop file and searching for command gives... executing has QXcbCOnnection error...trying with

```
QT_QPA_PLATFORM=mirserver qmlscene /qml/Main.qml
```

same error as for the clock



Okay, so finally, we will try this for a community app and end it for today.



*beru*

```
cd ~/builddir/community-apps/beru/
cmake -DCMAKE_INSTALL_PREFIX=/usr -DCLICK_MODE=off
make -j4
sudo make install
```

works...

```
beru
# error connecting..
QT_QPA_PLATFORM=mirserver beru
```

same error as allways...

