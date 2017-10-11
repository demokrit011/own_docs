# Simulating a "Ubports" 16.04 on x86 hardware (or VM)

What does this mean? Well there are some tablets out there with x86 hardware which desperately need to get hold of the UBports experience, but how to do this?

## Idea

* **Goal**: Have a Ubuntu 16.04 with Yunit (atop mir) running snaps & (phone) clicks
* **Path**
  *  Install Ubuntu 16.04
  *  Set up Yunit
  *  Set up sth. like 15.04 (armhf) container to run phone apps

Why is this usefull? This is **not really Ubports**, however it is somehow what i personally think we can explore and try out to show where the project can head in the mid-term. (In the long term it is essential to migrate the UITK to QQC2-suru or Kirigami or whatever and update the ubuntu-sdk-framework from 15.04.6 which is not available in 16.04 aka migrate the 15.04 click apps to whatever we will be using afterwards). **If this works, the pressure behind updating the underlying Ubuntu version is lowered** (at least i believe so)



## Roadmap

1. Set up 16.04 (x86) with Yunit and test basics - ~100% done
   1. Setup 16.04.3 in VM [download 16.04 here](http://releases.ubuntu.com/16.04/) - works
   2. Install Yunit [as described on their blog](https://yunit.io/yunit-packages-for-ubuntu-16-04-lts-xenial/) - works
   3. Test apt install method! - works
   4. Test snaps! - works (some do not and for dash integration of new/removed packages, reboot required)
   5. Test clicks! - fails (cause missing 15.04.6 framework and all clicks fro armhf only AFAIK)
   6. Installing things from software center - works (also gtk+ apps)
2. Set up 15.04 and test clicks - ~0% done
   1. Setup 15.04 in VM [download 15.04 here](http://old-releases.ubuntu.com/releases/15.04/) - todo (which version would be best?! PROBLEM: armhf images not available for download so testing this is hard to compare to LXC (armhf) container)
   2. Try to get clicks running (openstore etc.) - todo (maybe stable-phone-overlay ppa needed?!)
3. Set up 15.04 (armhf) LXC container on 16.04.3 (x64) and connect both - ~10% done
   1. Use 16.04 with yunit as described in (1) and set up LXD [how-to here](https://insights.ubuntu.com/2015/04/28/getting-started-with-lxd-the-container-lightervisor/) (update: this way of installing  won't work directly since there are no vivid containers @ linuxcontainers.org however there are the sdk images @ https://sdk-images.canonical.com/releases/ ) see also additional ressources at the end of this document - work-in-progress... PROBLEM: cross-architecture container don't work that well, some possible work-arounds i see:
      1. Try qemu ("real" virtualization) in seamless mode to emulate an armhf environment (downside: ressource intensive)
      2. Compile the clicks from the openstore for x86 architectures (32-bit binaries work on 64-bit but not vice versa so 32-bit would suffice) and perhaps use a 15.04-amd64/i386 container instead of armhf
   2. Launch an Ubuntu 15.04 (armhf) container - todo (maybe impossible, might work with sth. like qemu-user-static but has downsides)
   3. Install dependencies and configure the container (internally at first) to enable (phone) clicks as done in (2) - todo
   4. Test if installing clicks (headless) works - todo
   5. Make host hardware and files accessible for Container (some usefull links: [1](https://bitsandslices.wordpress.com/2015/12/08/creating-an-lxd-container-for-graphics-applications/), [2](https://blog.simos.info/how-to-run-graphics-accelerated-gui-apps-in-lxd-containers-on-your-ubuntu-desktop/), [3](https://askubuntu.com/questions/871092/failed-to-connect-to-mir-failed-to-connect-to-server-socket-no-such-file-or-di/880724), [4](https://askubuntu.com/questions/766269/remote-desktop-remote-display-viewer-for-ubuntu-touch-devices) a quick and dirty thing would be to use something like mirscreencast just as a proof-of-concept) - todo
   6. Test graphical clicks! - todo
   7. Test audio (in/out) clicks! - todo
   8. Test networking - todo
   9. Play around with phone apps! - todo
4. Polishing - ~1% done
   1. Write the docu! (best if that gets done while testing) - wip
   2. Gather what is still missing - done, almost everything ;-P
   3. auto-start container on host bootup -todo
   4. write script which does all the installing/configuring on vanilla 16.04 - todo
   5. See how the appstore can be mde to automatically use the container to install clicks (if necessary...) - todo
   6. Make cool videos of this! - todo

## Docu

This is 

(A) for people to reproduce and work further on the project and 

(B) so we can write a bash script or something to do all necessary steps on a vanilla 16.04 system

The ways to install Virtualbox/Qemu or anything like this are not explained for now since some may want to try this on an actual device. On an actual device, i would also recommend using **isorespin.sh** from Ian Morrison (http://linuxiumcomau.blogspot.de/) to respin the iso with the most recent kernel. This often helps with hardware problems from my experience!

### Setting up 16.04.3 (64-bit)
For doing this maybe it's usefull to move inside a directory, however i'm leaving this out for now since i am not your janitor.
```
wget http://releases.ubuntu.com/16.04/ubuntu-16.04.3-desktop-amd64.iso
# in the script we should check sha256sum or something afterwards, i always do this manually (sorry, i'm a noob ;-P)
wget http://releases.ubuntu.com/16.04/SHA256SUMS
sha256sum ubuntu-16.04.3-desktop-amd64.iso
# compare with what is listed in the SHA256SUMS file
less SHA256SUMS
```

Now you can either Respin this iso or directly move to your VM/tablet and install Ubuntu as usual...

### Installing yunit
As described on the yunit blog:
```
wget -qO - https://archive.yunit.io/yunit.gpg.key | sudo apt-key add
echo 'deb [arch=amd64] http://archive.yunit.io/ubuntu/ xenial main' | sudo tee /etc/apt/sources.list.d/yunit.list
echo 'deb-src http://archive.yunit.io/ubuntu/ xenial main' | sudo tee --append /etc/apt/sources.list.d/yunit.list
sudo apt update
sudo apt upgrade
sudo apt install yunit-desktop
```

### Setting up LXD and 15.04 (armhf) container - failed attempt

**ATTENTION: This does not work as described in the whole section and is just an "extended log" for all who are interested**

```
sudo apt install lxd
newgrp lxd
lxc remote add images images.linuxcontainers.org
# there are no vivid images @ linuxcontainers.org, need to do this manually
lxc image list images:
lxc launch images:ubuntu/vivid/armhf ubuntu-15.04-armhf
```
**TODO** If this works, the container has to be auto-started on host bootup! **TODO**

```
# Testing on the ubuntu 16.04.3
sudo apt install lxd
# says already newest version so ommiting creating a new group
lxc remote add ubuntu-sdk-images sdk-images.canonical.com
lxc image list ubuntu-sdk-images
## Okay this does not work, i wanted to avoid installing more of the sdk than needed on
## the host but let's try it as proposed in the second link under additional ressources
sudo add-apt-repository ppa:ubuntu-sdk-team/tools-development
sudo apt-get update
sudo apt install ubuntu-sdk-tools
# Now listing the available remotes:
lxc remote list
# damn, the sdk images have been deleted, so we have to download them manually and install
# as discussed by stephane graber in the 4th link under additional ressources...
```

We have 2 tarballs for every container inside (https://sdk-images.canonical.com/releases/vivid/) with the following combinations:

* amd64 - amd64
* amd64 - armhf
* armhf - armhf
* i386 - armhf
* i386 - i386

And i am assuming the first means the host, the latter the container architecture. Since trying this on an amd64 host, i choose amd64 - armhf to download and according to stephane, we can install the metadata tarball (the tiny one) and rootfs tarball (the bigger one) with the following command

```lxc image import <metadata tarball> <rootfs tarball>```

```
wget https://sdk-images.canonical.com/releases/vivid/ubuntu-sdk-15.04-amd64-armhf-lxd.tar.xz
wget https://sdk-images.canonical.com/releases/vivid/ubuntu-sdk-15.04-amd64-armhf-root.tar.xz
lxc image import ubuntu-sdk-15.04-amd64-armhf-lxd.tar.gz ubuntu-sdk-15.04-amd64-armhf-rootfs.tar.gz
# This should state "Transferring image: xx%" and then "image imported with fingerprint: xyz"
```

Now we can list our image and then try launching it

``` lxc image list```

we see that currently, no alias is set and since i am lazy and don't want to type the fingerprint all the time, we edit the image

```lxc image edit <fingerprint>```

and add the line 

```
  aliases: 15.04
```

but this does not seem to work right now... (it doesn't update the aliases) strangely, it states the architecture of the container to be x86_64 so this may be the wrong container...

To prevent this problem, i deleted the container again and imported again attaching the --alias=15.04 flag

```
lxc image delete <fingerprint>
lxc image import ubuntu-sdk-15.04-amd64-armhf-lxd.tar.gz ubuntu-sdk-15.04-amd64-armhf-rootfs.tar.gz --alias=15.04
```

This works and now we can just use this alias for referring. Let's test it with simply opening a bash shell in the next step...

It seems, we have to launch the lxc container from the image we just imported:

```
lxc launch 15.04 ubuntu-15.04-armhf
```

However, trying the "armhf-armhf" image will give the error:

```
error: The image architecture is incompatible with the target server
```

Damn, this seems to indicate that the guest container has to have the same architecture as the host :-(

(alhough i was able to run a i386 image on 64-bits but perhaps) Testing this thesis on Ubuntu 16.04.3 (x64, unity7, x11, qemu-user-static installed) with a xenial-armhf (fails, but with a more complex error code), testing with artful-amd64 (works) testing with artful-armhf (fails)

#### Trying again

Assuming we already installed LXC, now only getting the correct (hopefully?!) images from the old sdk and setting up the static qemu-binaries to emulate armhf...

```
sudo apt install qemu-user-static
wget https://sdk-images.canonical.com/releases/vivid/ubuntu-sdk-15.04-armhf-armhf-lxd.tar.xz
wget https://sdk-images.canonical.com/releases/vivid/ubuntu-sdk-15.04-armhf-armhf-root.tar.xz
lxc image import ubuntu-sdk-15.04-armhf-armhf-lxd.tar.gz ubuntu-sdk-15.04-armhf-armhf-root.tar.gz --alias=15.04-armhf
```

However, qemu-user-static was already installed in the VM system and again we get the error...

```
error: The image architecture is incompatible with the target server
```



#### Theoretically moving to container to apply changes needed for click

```
lxc exec 15.04 /bin/bash
```
This does however not work currently :-(



**TODO** find out what is needed to get this to work. **TODO**

### Setting up LXD and 15.04 (amd64) container with recompilation of clicks

#### LXD

```
sudo apt install lxd	
# instead of apt you can also use snap on snap enabled systems
newgrp lxd
```

The remote for the sdk-images seems to have been removed from the LXD config so we get the image manually...

```
wget https://sdk-images.canonical.com/releases/vivid/ubuntu-sdk-15.04-amd64-armhf-lxd.tar.xz
wget https://sdk-images.canonical.com/releases/vivid/ubuntu-sdk-15.04-amd64-armhf-root.tar.xz
lxc image import ubuntu-sdk-15.04-amd64-armhf-lxd.tar.gz ubuntu-sdk-15.04-amd64-armhf-rootfs.tar.gz --alias=15.04-amd64
# This should state "Transferring image: xx%" and then "image imported with fingerprint: xyz"
```

Now we have an image based on the 15.04-amd64-rootfs, we can start a container by using

```
lxc launch <image-alias> <container name>
# in our case
lxc launch 15.04-amd64 ubuntu-15.04-amd64
```

#### Compiling the clicks

**Attention: The openstore in this example compiled and installed successfully on 16.04. However it has some problem with actually running which has probably to do with mir. So not everything may work exactly as described below**

The theoretical workflow once all dependencies needed is:

* get the source code (git clone <repo.git>)
* qmake <appname.pro>
* make install
* and launch



So far we have not moved inside the container, you can for example start a bash shell in the container by using 

```
lxc exec ubuntu-15.04-amd64 /bin/bash
```



##### Dependencies

Beside packages that should be included in the system anyway (build-essential, gcc) a few things may be needed additionally (i will show the error code i got when trying to compile the openstore on a ubuntu 16.04.3 amd64 without it in *italics*)

```
sudo apt install build-essentials git bzr libclick-0.4-dev qt5-qmake qtdeclarative5-dev
```

Unfortunately, we also need ```ubuntu-sdk-qmake-extras``` which however is only provided via PPA for 14.04/16.04 [here](https://launchpad.net/~ubuntu-sdk-team/+archive/ubuntu/ppa). **TODO: find alternative source for this**



##### Getting the code

Most apps (AFAIK) in the openstore have a link to their sources. Move to the sources and clone them to your local folder 

​	**Launchpad**

you navigate to the "code" tab and the necessary command to clone is displayed, something like:

```
bzr branch lp:<yourapp>
# exmaple: uNav navigation app by Marcos Costales
bzr branch lp:unav
```

​	**Github**

you click on the green "clone or download" button and copy the link (something like "https://github.com/[...].git") into the following command

```
git clone <your link>
# example: uMatrix messenger app by Larrea Mikel
git clone https://github.com/LarreaMikel/uMatriks.git
```



##### Compiling & Installing

So since these apps are Qt(QML) based we start with ```qmake```. This is a symlink to ```qmake-qt4``` and/or ```qmake-qt5``` respectively (you can have both simultaneously, however choosing the right one is necessary for the process to work) which are actually located in (/usr/lib/x86_64-linux-gnu/qt<4 or 5>/bin/qmake). You can specifiy which qt version to use by appending a flag to the qmake command:

```
qmake -qt=qt<4 or 5> <yourapp.pro>
# example openstore
qmake -qt=qt5 openstore.pro
```

* If "*manifest.json is not found*" sometime the manifest in the sources has the ending .in, to make this work just use ```mv manifest.json.in manifest.json``` inside the source folder

Once this has finished without being aborted, you can run

```
make install
```



##### Testing

You should now be able to test what happens when you execute the binary at (/lib/x86_64-linux-gnu/bin/<yourapp>)

```
cd /lib/x86_64-linux-gnu/bin/ 
./<yourapp>
# example: openstore
./openstore
```





### Additional Ressources

* http://www.techradar.com/how-to/computing/how-to-install-ubuntu-onto-a-windows-tablet-1319489/1 (older description how to install ubuntu on x86 tablets)
* https://gist.github.com/chihchun/30fd95f9f906ab1e7731040eddc840ee (how to use Ubuntu SDK containers (15.04/16.04))
* https://sdk-images.canonical.com/releases/ (the according SDK images)
* https://stgraber.org/2016/03/30/lxd-2-0-image-management-512/ (some more info on LXD)
* http://cdimage.ubuntu.com/ubuntu-touch/vivid/daily-preinstalled/current/ (vivid preinstalled images from canonical (don't know were they are for Ubports) perhaps for LXC containers)
* https://wiki.ubports.com/wiki/Set-up-a-Clickable-working-environment-inside-an-LXC-container (ubports wiki page were LXD is also a topic...)
* https://stgraber.org/2016/03/11/lxd-2-0-blog-post-series-012/ (Stephane Grabers series on LXD)
* https://stgraber.org/2012/02/03/ever-wanted-an-armel-or-armhf-container-on-an-x86-machine-its-now-possible-with-lxc-in-ubuntu-precise/ (OLD cross-architecture container guide, using qemu-user-static)
* https://resin.io/blog/building-arm-containers-on-any-x86-machine-even-dockerhub/ (more recent cross-architecture container info, also based on static qemu)



##### Usefull when using a VM

since most time we just need a shell, it's usefull to easily **ssh into the VM**. for this purpose, you can go to the settings of the VM (in Virtualbox that is), navigate to network -> NAT (adapter 1), open the additional feature and go to "port forwarding" and add a new rule:

| Name | Protocoll | HOST-IP | HOST-port | GUEST-IP | GUEST-port |
| ---- | --------- | ------- | --------- | -------- | ---------- |
| ssh  | TCP       |         | 3022      |          | 22         |



In the VM install openssh-server

```
sudo apt install openssh-server
```



and in the host you can simply do

```
ssh -p 3022 <user>@127.0.0.1 
```

 