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
* https://launchpad.net/ubuntu/+source/ubuntu-touch-meta

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
* https://packages.ubuntu.com/xenial/ubuntu-sdk-libs

#### Generell lessons

Assuming the canonical devs had the following working history:
1. Apps were running as clicks on a modified minimal ubuntu + stable phone overlay (meta-click)
2. Apps were made running on ubuntu minimal + stable phone overlay (core-snap) as big snaps with all dependencies
3. Apps were made dependent on plattform snap whith UITK, Ubuntu-sdk-libs, [etc.](https://git.launchpad.net/ubuntu-app-platform/tree/snapcraft.yaml) as smaller snaps with individual dependencies

This means that by having the stable-phone overlay + ubuntu-app-platform packages, we should be able (idealy) to build and run any current click package by compliling it locally.

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



### Which Apps to migrate first?

From Stefano Verzegnassi is the following list of the 20 most downloaded apps from the openstore

1. OpenStore
2. UT Tweak Tool
3. Telegram
4. WiFi Scanner
5. Gallery
6. uNav
7. Bluetooth File Transfer
8. TweakGeek (unsupported)
9. Falcon
10. UBports Welcome App
11. OSM Scout
12. Calendar
13. Beru
14. Fishy Shooter (w/ 140 updates!)
15. Terminal
16. Indicator Weather
17. File Manager
18. Compass
19. LoquiIM
20. OwnCloud Sync



Personally i would add a few and have different priorities...

Let's create a table where we directly link to the openstore webpage, the source code and have checkboxes for building ok locally, running ok locally (both on Ubuntu 16.04 x86 + Unity7) same goes for Ubuntu 16.04 + Yunit (atop mir) and for snapping isolated/snapping using app-platform

| App                                      | Source                                   | Builds on 16.04 (unity7) | Runs on 16.04 (unity7) | Builds and Runs on 16.04 (yunit) | Snapped isolated | Snapped + platform |
| ---------------------------------------- | ---------------------------------------- | ------------------------ | ---------------------- | -------------------------------- | ---------------- | ------------------ |
| [Beru](https://open.uappexplorer.com/app/com.ubuntu.developer.rschroll.beru) | [source](https://github.com/rschroll/beru) |                          |                        |                                  |                  |                    |
| [Bt File transfer](https://open.uappexplorer.com/app/ubtd.mzanetti) | [source](https://launchpad.net/~mzanetti/+junk/ubtd) |                          |                        |                                  |                  |                    |
| [Calendar](https://open.uappexplorer.com/app/com.ubuntu.calendar) | [source](https://github.com/ubports/calendar-app) |                          |                        |                                  |                  |                    |
| [Calculator](https://open.uappexplorer.com/app/com.ubuntu.calculator) | [source](https://github.com/ubports/calculator-app) |                          |                        |                                  |                  |                    |
| [Document Viewer](https://open.uappexplorer.com/app/com.ubuntu.docviewer) | [source](https://github.com/sverzegnassi/docviewer-app) |                          |                        |                                  |                  |                    |
| [Falcon](https://open.uappexplorer.com/app/falcon.bhdouglass) | [source](https://github.com/bhdouglass/falcon) |                          |                        |                                  |                  |                    |
| [File Manager](https://open.uappexplorer.com/app/com.ubuntu.filemanager) | [source](https://github.com/ubports/filemanager-app) |                          |                        |                                  |                  |                    |
| [Gallery](https://open.uappexplorer.com/app/com.ubuntu.gallery) | [source](https://github.com/ubports/gallery-app) |                          |                        |                                  |                  |                    |
| [Guitar tools](https://open.uappexplorer.com/app/guitar-tools.t-mon) | [source](https://code.launchpad.net/guitar-tools) |                          |                        |                                  |                  |                    |
| [InstantFX](https://open.uappexplorer.com/app/instantfx.sverzegnassi) | [source](https://launchpad.net/instantfx) |                          |                        |                                  |                  |                    |
| [Kodimote](https://open.uappexplorer.com/app/com.ubuntu.developer.mzanetti.kodimote) | [source](https://github.com/mzanetti/kodimote) |                          |                        |                                  |                  |                    |
| [LoquiIM](https://open.uappexplorer.com/app/loquiim.nfsprodriver) | [source](https://github.com/loqui/im)    |                          |                        |                                  |                  |                    |
| [Notification Sender](https://open.uappexplorer.com/app/com.edi.npost) | [source](https://github.com/BigET/NotificationPost) |                          |                        |                                  |                  |                    |
| [OpenStore](https://open.uappexplorer.com/app/openstore.openstore-team) | [source](https://github.com/UbuntuOpenStore/openstore-app) | yes                      | yes?                   |                                  |                  |                    |
| [ScummVM](https://open.uappexplorer.com/app/scummvm.michael-sheldon) | [source](https://github.com/scummvm/scummvm) |                          |                        |                                  |                  |                    |
| [Shorter](https://open.uappexplorer.com/app/doniks.shorter) | [source](https://github.com/doniks/shorter) |                          |                        |                                  |                  |                    |
| [Tagger](https://open.uappexplorer.com/app/com.ubuntu.developer.mzanetti.tagger) | [source](https://code.launchpad.net/~mzanetti/tagger/trunk) |                          |                        |                                  |                  |                    |
| [Telegram](https://open.uappexplorer.com/app/com.ubuntu.telegram) | [source](https://github.com/ubports/telegram-app) |                          |                        |                                  |                  |                    |
| [Syncthing](https://open.uappexplorer.com/app/syncthing.zeropointenergy) | [source](https://github.com/syncthing/syncthing) |                          |                        |                                  |                  |                    |
| [Terminal](https://open.uappexplorer.com/app/com.ubuntu.terminal) | [source](https://github.com/ubports/terminal-app) |                          |                        |                                  |                  |                    |
| [Timer](https://open.uappexplorer.com/app/timerpro.mivoligo) | [source](https://launchpad.net/timer)    |                          |                        |                                  |                  |                    |
| [TouchRules](https://open.uappexplorer.com/app/com.ubuntu.developer.psasse.touchrules) | [source](https://launchpad.net/touchrules) |                          |                        |                                  |                  |                    |
| [UBports welcome app](https://open.uappexplorer.com/app/com.ubuntu.ubports) | [source]( <https://github.com/ubports/ubports-app) |                          |                        |                                  |                  |                    |
| [UT Tweak Tool](https://open.uappexplorer.com/app/ut-tweak-tool.sverzegnassi) | [source]( <https://code.launchpad.net/~ubuntu-touch-tweak-tool-devs/ubuntu-touch-tweak-tool/trunk) |                          |                        |                                  |                  |                    |
| [Calculus 2](https://open.uappexplorer.com/app/com.ubuntu.developer.dinko-metalac.calculus-app2) | ?                                        |                          |                        |                                  |                  |                    |
| [WifiTransfer](https://open.uappexplorer.com/app/wifitransfer.sil) | [source]( <https://code.launchpad.net/~sil/wifitransfer/trunk/) |                          |                        |                                  |                  |                    |
| [tweakgeek](https://open.uappexplorer.com/app/tweakgeek.mzanetti) | [source](  <https://code.launchpad.net/~mzanetti/+junk/tweakgeek) |                          |                        |                                  |                  |                    |
| [uMatrix](https://open.uappexplorer.com/app/umatriks.larreamikel) | [source]( <https://github.com/LarreaMikel/uMatriks) |                          |                        |                                  |                  |                    |
| [uNav](https://open.uappexplorer.com/app/navigator.costales) | [source]( <https://launchpad.net/unav)   |                          |                        |                                  |                  |                    |

