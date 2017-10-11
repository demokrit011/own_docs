## Getting screen rotation right

So let's think about what needs to be done? in X we would simply use xrandr, but this won't work with mir so we need to find the config for unity8/yunit.	

Using these three references:

* https://wiki.ubuntuusers.de/Ubuntu_Touch/Erweiterte_Konfiguration/
* https://wiki.ubuntuusers.de/Baustelle/Ubuntu_Touch/Snippets/
* http://notyetthere.org/unity8-windowed-mode/

I figured that gsettings has something to do with it ( [german info from ubuntuusers](https://wiki.ubuntuusers.de/GNOME_Konfiguration/dconf/) ) which should have it's config at 

```
~/.config/dconf/user
```

Otherwise, unity8 and the unity8-dash have .desktop files @ ```/usr/share/applications/``` where we might add the Option used for "normal" Apps to run in one certain mode which would be **X-Ubuntu-Supported-Orientation=landscape** ( [Ref](https://wiki.ubuntu.com/Unity8/FullShellRotation) )

Returning at first to the gsettings stuff.... The config is not in a human readable format so ```less``` does not work here and we need to install the *dconf-editor* and start it afterwards

```
sudo apt install dconf-editor
dconf-editor
```

This seems to produce a problem with mir... *sigh* Trying directly with gsettings ( [manpage](http://manpages.ubuntu.com/manpages/xenial/en/man1/gsettings.1.html) )...

```
gsettings list-keys com.canonical.Unity8
```

However going through the list which is presented, i don't think there is an option we can set to rotate the screen :'-( They are btw:

```
always-show-osk
enable-indicator-menu
appstore-url
edge-barrier-min-push
autohide-launcher
edge-barrier-max-push
launcher-width
osk-switch-visible
edge-barrier-sensitiviy
enable-launcher
edge-drag-width
usage-mode
```



One thing i just realized as i wanted to send some logfiles via browser and webmail... There is currently no support (at least in this configuration i am using) to attach a file since the browser is forbidden to access my home directory and it can't interact with the filemanager ("files"), HOWEVER, USB-drives work so i can do it "the old way ;-P"



**Some info on the mir side of life**

So i just had the opportunity to ask the great Alan Griffiths from canonical (and seemingly THE mir guy) to ask how i would go about rotating the screen and whether to ask mir or yunit. This was his answer:

> the libmirclient API supports that request, the yunit server would decide if/how to handle it. mirout (from mir-utils) provides a command-line to use this API.

I think he means the ```out.c``` located @ http://bazaar.launchpad.net/~mir-team/mir/development-branch/files/head:/src/utils/ where we can see within lines 40 to 50:

```c++
static const char *orientation_name(MirOrientation ori)
{
    static const char * const name[] =
    {
        "normal",
        "left",
        "inverted",
        "right"
    };
    return name[(ori % 360) / 90];
}

```



**Back to unity8/yunit**

Ok, so if i move inside the "unity8" folder and ```grep -R 'Orientation'``` there is  a hell lot of Orientations around. Only listing the names of the files with path (starting from unity8/):

* plugins/bash/listviewwithpageheader.h
* plugins/Utils/deviconfigparser.h
* plugins/Utils/deviconfigparser.cpp
* plugins/Unity/Session/plugin.cpp
* plugins/Unity/Session/orientationlock.h
* plugins/Unity/Session/orientationlock.cpp
* *omitting lots of stuff in tests/ here* ....
* doc/devices.conf
* data/devices.conf
* *also lots of hits inside the qml/ folder*....

My uneducated guess here is that orientationlock is the pendant for our display-indicator which locks orientation (my friends call me Sherlock btw)

#### Some diving into Yunit/Unity8

So there are basically two orientations, Landscape (broader than high) and Portrait. These are triggered by some values of the gyroscope however i would like to controll them manually... Also a mirroring and further turning should be possible. End-goal is to integrate this into indicator-display

##### Check out

* https://github.com/ubports/indicator-display
* https://github.com/ubports/unity8 



#### Log

```
git clone https://github.com/ubports/unity8.git
cd unity8/
grep -R 'x'
```

x here was done with

* 'gyro' > nothing
* 'Gyro' > nothing
* 'landscape' > some hits
* 'ScreenOrientation'



###### 'landscape'

* /unity8/plugins/Utils/deviceconfigure.h
* /unity8/plugins/Utils/deviceconfigure.cpp

also lots of hits in /unity8/tests/ and /unity8/qml/



###### 'ScreenOrientation'

Interesting: there is something inside ```unity8/plugins/Unity/Session/```  namely:

* orientationlock.cpp
* orientationlock.h > this has infos that should help



**This seems to be the key** since the second property *Qt::ScreenOrientation* which saves the Orientation across Sessions. If we can just alter this manually we should have what we want. If i get this correctly it should be something like

```
WRITE setSavedOrientation
```

The official Qt docu might help here: http://doc.qt.io/qt-5/qscreen.html especially this http://doc.qt.io/qt-5/qscreen.html#orientation-prop 



### Fixing the touchscreen

Basically, there seems to be no driver and i don't think the TS is detected. I extracted a few logs to go through...

```
sudo lshw > lshw.log
sudo lspci > lspci.log
dmesg > dmesg.log
```

I also tried to use ```xinput``` (https://wiki.ubuntu.com/X/Config/Input & https://linux.die.net/man/1/xinput)

```
xinput list
xinput --test idX
```

going through all available device-ids, only the keyboard and touchpad from the docking station are detected... moving on...

In windows 10, opening the device manager shows that, interestingly, ther is a Bosch Accelerometer inside as well as an Ambient light sensor... Anyways for the TS i think these entries might be correct:

* Human Interface Devices
  * HID-compliant touch screen
  * KMDF HID Minidriver for Touch I2C Device

These two also stay after unpluggin the dock (btw, Ubports/yunit should perhaps copy the behaviour that there is automatically an notification that auto-rotate is now enabled and if i want to switch to tablet mode... it sucks that windows is really thought-through in some cases ;-)

##### HID compliant TS

Under details for the first entry (HID-compliant TS) it says "Location: on KMDF HID minidriver" (aka entry 2) so these are interconnected and probably what we are looking for...

##### KMDF HID Minidirver

So this points to "Location: on Intel(R) Serial IO I2C ES Controller" some properties from the Details Tab:

* Hardware IDs = 
  * ACPI\VEN_MSSL$DEV_1680
  * ACPI\MSSL1680
  * *MSSL1680
* Compatible IDs = PNP1680
* BIOS Device Name = "\_SB.PCI0.I2C6.TCS1"
* Configuration ID = "oem21.inf:ACPI\MSSL1680.SileadTouch.Inst.NT"
* Dependency Providers = "ACPI\INT33FF\2" and "ACPI\808622C1\6"
* Manufacturer = "Sileadinc.com"
* Legacy Bus Type = "00000011"



And from the ressources tab we get the following Ressource settings: type "IRQ" with setting "0x0000040C (1036)"

There is no further especially usefull info inside the Intel I2C Serial driver...



So this device is on the 6th I2C ES Serial IO, maybe we can get something out of this. I think the most important parts are the Hardware ID as well as BIOS Name and Dependency providers.

##### Analysing our logs

so i put the logs into a folder ```/logs``` and now we can ```grep``` for a few things...

```
grep -R '808622C1'
```

and voila:

![grep_01](/home/demokrit/Dokumente/ubports/own_docs/grep_01.png)

This tells us that the I2C device is handled by **PUNIT semaphore**... I have no clou what exactly that is but there is lot's of references online, e.g. 

* https://sae762.wordpress.com/linux-for-chuwi-hi10/ (attention, russian)
* http://linuxiumcomau.blogspot.de/2016/10/running-ubuntu-on-intel-bay-trail-and.html

Anyways, i think this works correct, moving on and trying to find the TS itself...

```
grep -R 'MSSL1680'
```

this looks interesting:

![grep_02](/home/demokrit/Dokumente/ubports/own_docs/grep_02.png)

So the first 3 lines are okay but then it starts....

**Direct firmware load for silead/mssl1680.fw failed with error -2** 

Some valuable information is gained from the [Arch Wiki on a Teclast Tablet](https://wiki.archlinux.org/index.php/Teclast_X98_Plus_II) and through that wiki in the [driver info on github](https://github.com/sigboe/gslX68X) and following that info we finally land at the [gsl-firmware on Github](https://github.com/onitake/gsl-firmware) where our tablet is listed... wooohooo

Tada: our link https://github.com/onitake/gsl-firmware/tree/master/firmware/chuwi/hi10_pro-z8350