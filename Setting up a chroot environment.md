

## Setting up a chroot environment

### Installing prerequisites

```
sudo apt install schroot debootstrap
```



### Getting static chroot

```
wget http://people.canonical.com/~bzoltan/static_chroots/click-ubuntu-sdk-15.04-armhf-amd64.tar.gz
```



where to put this??

probably have to insert something into ```/etc/schroot/schroot.conf``` there are a few entries already.

#### Testing with a "standard chroot"

So since i don't know what i need to do, i will be following some tutorials on chroot to create a chroot with the classic approach and then try to implement the static one in the same way.

We already installed schroot and debootstrap so we can dive right into the game...



##### Reference enrty @ schroot.conf

```
#[squeeze]
#description=Debian squeeze (stable) 32-bit
#directory=/srv/chroot/squeeze
#groups=sbuild-security
#aliases=stable
#personality=linux32
```



##### Creating necessary folders

Looking into ```/etc/schroot/schroot.conf``` the standard aproach seems to be to use the directory ```/srv/chroot/``` for our chroot, so let's create a directory named vivid:

```
sudo mkdir /srv/chroot
sudo mkdir /srv/chroot/vivid
```

##### Fetching Image (Debootstrappin')

So let's get an ```armhf``` image of ```ubuntu 15.04 (vivid)``` directly from the ubuntu repos, call it ```vivid``` and place it inside ```/srv/chroot/vivid```

```
sudo debootstrap --arch armhf vivid /srv/chroot/vivid http://de.archive.ubuntu.com/ubuntu/ 
```

###### First error

```
I: Retrieving InRelease 
I: Checking Release signature
I: Valid Release signature (key id 790BC7277767219C42C86F933B4FE6ACC0B21F32)
I: Retrieving Packages 
I: Retrieving Packages 
I: Retrieving Packages 
E: Couldn't download dists/vivid/main/binary-armhf/Packages
```

This tells the little fox that i am that we need to have packages inside <server>/dists/<name>/main/binary-<arch>/ HOWEVER, for vivid **there are only i386 and amd64 binaries**. This seems to be the case for all releases -.-



##### Trying again with vivd i386
So let's get an ```i386``` image of ```ubuntu 15.04 (vivid)``` directly from the ubuntu repos, call it ```vivid``` and place it inside ```/srv/chroot/vivid```

```
sudo debootstrap --arch i386 vivid /srv/chroot/vivid http://de.archive.ubuntu.com/ubuntu/ 
```

This seems to download all packages inside the basic minimal Ubuntu 15.04 needed for a chroot until the last message reads:

```
I: Base system installed successfully.
```

So let's have a look into the folder we wanted to use:

```
ls /srv/chroot/vivid/
```

Output =
```
bin   dev  home  media  opt   root  sbin  sys  usr
boot  etc  lib   mnt    proc  run   srv   tmp  var
```
looks like the normal root directory of a ubuntu system (strange ;-P). **This is exactly what is packed inside the static_chroot.tar.gz we downloaded earlier**

using the find command, we can check whether there was any change outside of the chroot directory we let the image be created in...

```
sudo find / -mmin 20
```

nope, only chroot... so far so good.



##### Creating schroot.conf entry

This may be copied more or less from the above sid entry and from or from [the schroot.conf manpage](http://manpages.ubuntu.com/manpages/trusty/man5/schroot.conf.5.html). It seems to be custom to use the "sbuild" group as access-permitted group so we will add our user to this group and afterwards start editing our entry:

```
sudp addgroup sbuild
sudo adduser <user> sbuild
sudo gedit /etc/schroot/schroot.conf
```



```
[vividi386]
description=Ubuntu Vivid (15.04) 32-bit
directory=/srv/chroot/vivid
groups=sbuild
aliases=vivid
personality=linux32
```

Let's see if this suffices....

```
schroot -l
```

OUTPUT=

```
chroot:vivid
chroot:vivid32
```

we get two entries as both the name AND the alias are shown but not as the same chroot....

##### Testing the chroot

We can directly try to move into the chroot by using

```
sudo schroot -c vivid32
```

which besides some not too interesting errors works... (it tries to enter the chroot inside the /home/<user> directory with the same user the host uses, this however is non existant in this chroot so it falls back to opening the root dir. By running ```mkdir ../home/<user>``` you can use the above command next time without an error)

**Leaving the chroot**

```
exit
```

ok, this seems to work. Let's try the downloaded armhf chroot...



#### Testing Vivid armhf 

So i downloaded the tarball into my ~/Downloads directory... Moving there and using ```tar -xvzf <path-to-tar.gz> -C /srv/chroot/``` to unpack it into our chroots directory

```
cd ~/Downloads
sudo tar -xzvf click-ubuntu-sdk-15.04-armhf-amd64.tar.gz -C /srv/chroot/
```

Creating the schroot.conf entry

```
[vividarmhf]
description=Ubuntu vivid (15.04) armhf
directory=/srv/chroot/click-ubuntu-sdk-15.04-armhf
groups=sbuild
aliases=armhf
personality=linux32
```

the personality may be wrong and perhaps we need ```qemu-arm-static``` inside ```chroot/usr/bin``` and register it as an ARM interpreter in the kernel (? dont know about that, just read it at the instructions from github user mikkeloscar?)

Login 

```
sudo schroot -c vividarmhf
```

works!

Let's test installing a click package... (the following happens inside the chroot)

```
mkdir /home/yunit
cd /home/yunit/
exit
```

The repos are down and ```wget``` is not installed to fetch the click, however we downloaded it into our host earlier so we'll leave, copy it to the chroot directory and enter the chroot again

```
wget https://open.uappexplorer.com/api/download/openstore.openstore-team/openstore.openstore-team_latest_armhf.click
sudo mv openstore.openstore-team_latest_armhf.click /srv/chroot/click-ubuntu-sdk-15.04-armhf/home/yunit/
sudo schroot -c vividarmhf
sudo click install --user <username> openstore.openstore-team_latest_armhf.click
```

DUmDUmDum... click is not found... I give this up now, have to study -.-






### Ressources

* https://launchpad.net/ubuntu-sdk-api-15.04 (usefull and with links to static 15.04 chroots as used in the SDK	however from 2015 so probably best to update this once it works)
* http://bazaar.launchpad.net/~bzoltan/ubuntu-sdk-api-15.04/trunk/view/head:/README ("How-To" install the static chroots, lots of info missing :-( )
* http://manpages.ubuntu.com/manpages/trusty/man1/click.1.html (something about click e.g. for click chroot was indicated by clickable)
* https://github.com/bhdouglass/clickable (clickable)
* https://wiki.ubuntuusers.de/schroot/ (schroot = secure chroot)
* https://wiki.ubuntuusers.de/chroot/ (chroot)
* https://help.ubuntu.com/community/BasicChroot (basic chroot)
* http://manpages.ubuntu.com/manpages/xenial/man8/debootstrap.8.html (debootstrap manpage)
* https://gist.github.com/mikkeloscar/a85b08881c437795c1b9 (armv7 chroot under x64 host Archlinux)