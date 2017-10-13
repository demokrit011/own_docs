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

