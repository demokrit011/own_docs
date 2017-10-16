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
|                          |                                       |                                          |                                  |
|                          |                                       |                                          |                                  |
|                          |                                       |                                          |                                  |



### Testing what is needed for building & installing a phone app locally (usually as *.click but NOT here)

#### Ubuntu-Clock-App

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

#### Openstore

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