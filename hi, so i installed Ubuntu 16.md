hi, so i installed Ubuntu 16.04.3 on my Chuwi Hi10 Pro (+"Laptop-Dock") and am now trying to get yunit, my touchscreen and everything to work and report what i did and what the result was;

#### Log

* Downloaded 16.04.3-ubuntu-amd64 
* Respun with isorespin using the Bay Trail/Cherry Trail stuff from Ian Morrison and updating to newest kernel (called isorespin..sh with '-i /path-to/ubuntu-16.04.3-desktop-amd64.iso --atom -u')
* Used etcher to create installable medium
* Booted and installed Ubuntu 16.04.3
* **Inside 16.04**:
  * My Display was rotated so went to *Settings>Displays>Rotation>* and selected ```clockwise``` (this also rotates the little touchpad on the dock to the same, correct rotation) HOWEVER this is not safed inbetween boot-ups and in yunit i did not have the option to rotate. Strangely i also don't get into the terminal with my normal user password, also not blank. Going into a console using ```Ctrl + Alt + F1``` i ran ```echo 1 | tee /sys/class/graphics/fbcon/rotate``` [source](https://askubuntu.com/questions/237963/how-do-i-rotate-my-display-when-not-using-an-x-server) which only rotated the console... using ```echo 1 | tee /sys/class/graphics/fbcon/rotate_all``` when switching back to graphical shell (F7) shows that NOW the terminal app is just logged in automatically...
  * performed the addition of yunit deb sources to apt repos as instructed on yunit.io

#### Notes for this device

*Working*: 
* Out of the box: WiFi (although my Wifi password is always only saved on the second time i try to connect), Bluetooth, Dockingkeyboard/Touchpad (in dmesg "LIZHICHIP USB Keyboard" as Keyboard and Mouse detected but both are named Keyboard; detaching and reattaching works flawless)
* With some help:

*Not-Working*:
* Touchscreen (it is none of the devices listed for ```xinput list``` nor any input in /dev/input/eventX)
* internal Audio
* Gyro if there is one (so no auto-rotation)