# Click 2 Snap

Rationale: Since Ubports (Ubuntu Touch) is moving on towards a Ubuntu 16.04 base and it seems that the decision is/will be to use **snaps instead of clicks** for packaging the converged apps, we need a good documentation for our devs how to migrate easily. Ultimate goal would be a script which takes the click sources, creates a snapcraft.yaml and changes whatever else is necessary.

## Infowars: Episode 1 - the revenge of the bad documentation

As always, we need to get the info we need from various blogs, launchpad, github, rumours and what-not-else since people at Canonical are great visionaries, coders and designers but sadly don't have time (as everyone, right?! ;-P) to document things in an easy fashion.

### Click Packaging

#### Some Links
* https://wiki.ubuntu.com/Click/Frameworks 
* https://wiki.ubuntu.com/Process/Merges/TestPlan/click
* https://launchpad.net/ubuntu/+source/click
* http://click.readthedocs.io/en/latest/index.html
* https://wayback.archive.org/web/20161119000342/http://developer.ubuntu.com/en/
* https://packages.ubuntu.com/xenial/ubuntu-sdk-libs code behind frameworks?
* http://notyetthere.org/on-manually-creating-click-packages/

#### Configuration files
With this expression, i am meaning the files inside the source code, where the specific interactions/installation of the click package in question is stated. Which **dependepncies, hooks, permissions are necessary?**

* CMakeLists.txt if CMake is used for building [some german info](https://wiki.ubuntuusers.de/CMake/)
* manifest.json.in which has *general (biolerplate) info* for the later snapcraft.yaml like arch, desciption, used framework, **hooks**
* <app-name>.apparmor has **permissions** [ref](https://docs.ubuntu.com/phone/en/platform/guides/app-confinement) which are helpfull for the **interfaces** [section](https://docs.snapcraft.io/reference/interfaces) in snapcraft.yaml
* Perhaps contenthub.json for content sharing permissions


### Snap Packaging

#### Some Links
* https://snapcraft.io/
* https://docs.snapcraft.io/reference/interfaces
* https://github.com/snapcore/snapd/wiki/Distributions
* https://docs.ubuntu.com/core/en/reference/interfaces/index
* https://forum.snapcraft.io/t/the-content-interface/1074 "future" of the content-hub?!

### Core Team Transition History
The Core App Devs already ported some apps (ate least to some part) to snap so we might learn a few things from these transitions:

* https://github.com/ubuntu/snappy-playpen
* https://launchpad.net/ubuntu-app-platform
* https://insights.ubuntu.com/2016/12/08/using-the-ubuntu-app-platform-content-interface-in-app-snaps/
* http://bazaar.launchpad.net/~phablet-team/address-book-app/trunk/revision/625 also check later snap specific commits!
* https://launchpad.net/~ubuntu-sdk-team check related projects!
* https://launchpad.net/~phablet-team
* https://launchpad.net/ubuntu-system-apps
* https://docs.ubuntu.com/phone/en/platform/guides/app-confinement

#### Individual transition stories

##### ubuntu-terminal-app (Click 2 Snap)
* [First snapping revision, rev208: "Adds snapcraft.yaml"](http://bazaar.launchpad.net/~ubuntu-terminal-dev/ubuntu-terminal-app/trunk/revision/208)
* [Usage of the *shared* ubuntu-app-plattform, rev254](http://bazaar.launchpad.net/~ubuntu-terminal-dev/ubuntu-terminal-app/trunk/revision/254)




##### ubuntu-rssreader-app/reboot (NOT snapped)

* [latest, unsnapped launchpad revision](http://bazaar.launchpad.net/~ubuntu-shorts-dev/ubuntu-rssreader-app/reboot/files)

##### ubuntu-calculator-app (Click 2 Snap)

* [First snapping revision, rev302: "Adds snapcraft.yaml"](http://bazaar.launchpad.net/~ubuntu-calculator-dev/ubuntu-calculator-app/trunk/revision/302)
* [Usage of *shared* ubuntu-app-plattform, rev312](http://bazaar.launchpad.net/~ubuntu-calculator-dev/ubuntu-calculator-app/trunk/revision/312)

Interesting on a quick first sight at the first snapcraft.yaml:

* apps: 
  * plugs: [**unity7**, opengl]
* parts:
  * calculator:
    * build-packages....	
    * stage-packages:
      * **ubuntu-sdk-libs**
      * qtubuntu-desktop
    * after: [dekstop/qt5]

This is **before using ubuntu-app-platform** which changes the snapcraft.yaml to this:

apps: 

- plugs: [unity7, opengl, **platform**]
- **plugs:**
  - **platform:**
    - interface: content
    - content: ubuntu-app-platform1
    - target: ubuntu-app-platform
- **parts: #remove stage-packages (inside ubuntu-app-platform)**
  - calculator:
    - build-packages...
    - **after: [desktop-ubuntu-app-platform]**
    - **environment:**
      - source: snap/
      - plugin: dump

So **there is no change in code besides packaging inbetween switching from click-framework to ubuntu-app-platform**! This means that perhaps all the 15.04.6 apps can be made running with this!! (Actually, manifest.json.in says ubuntu-sdk-15.04.1-qml is used)



##### sudoku-app (NOT snapped)

[Link](http://bazaar.launchpad.net/~sudoku-touch-dev/sudoku-app/trunk/files/head:/) this is interesting because when comparing to the other only Clicked app above, we can learn a few things about click configuration which seems to take place (more or less) inside the folder */click* which contains:

* CMakeLists.txt
* manifest.json.in
* sudoku.apparmor



However, neither terminal nor calculator had a */click/* folder, even before introducing snap-packaging.BTW, only other snapped core app (that made it to the snap store) is [Clock](https://uappexplorer.com/snap/ubuntu/ubuntu-clock-app)



##### ubuntu-clocl-app

* [First snapping revision, rev479](http://bazaar.launchpad.net/~ubuntu-clock-dev/ubuntu-clock-app/trunk/revision/479)
* [Using ubuntu-app-platfrom, rev497](http://bazaar.launchpad.net/~ubuntu-clock-dev/ubuntu-clock-app/trunk/revision/497)

This has a completely different **complexity** and looks very interesting for in-depth stuff...

### Ancient Core Apps Team History

There was once a PPA to install the core apps. We might learn some usefull stuff for transitioning comparing traditional .debs with the intermittant .clicks and final .snaps

* https://wiki.ubuntu.com/Touch/Collection
* https://wiki.ubuntu.com/Touch/CoreApps
* https://launchpad.net/ubuntu-system-apps
* https://launchpad.net/ubuntu-phone-coreapps
* https://launchpad.net/~ubuntu-touch-coreapps-drivers
* https://launchpad.net/~ubuntu-touch-coreapps-drivers/+archive/ubuntu/daily
* https://launchpad.net/~ubuntu-touch-coreapps-drivers/+archive/ubuntu/collection



### How doe the KDE people do it?
Since KDE is, as you might say it, the closest relative to Yunit out there (both are Qt/QML based, want convergence, use Toolkits derived from QtQuick2 i.e. UITK and Kirigami) it seems advisable to 
1. **share as much code as possible** with them (c.p. Halium) so everyone has less work. This might also make the DE on the phone be interchangeable some day (a further step towards a "real linux experience")
2. **learn from their efforts**

In snapd, there are now the *KDE-framework-5* and *GNOME-3.26-framework* (or whatever) for those two DEs. The question is if Yunit/Ubports should also **release Yunit/Ubports-framework**?

#### Some links
* https://apachelog.wordpress.com/2016/12/02/snapping-kde-applications/
* https://blog.neon.kde.org/index.php/2017/08/29/great-web-browsing-coming-back-to-kde-with-falkon-new-packaging-formats-coming-to-kde-with-snap/