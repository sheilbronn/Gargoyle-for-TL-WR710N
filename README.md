# Full Notes on building Gargoyle 1.13 on Debian 11 and Ubuntu 20.04 on WSL (for TL-WR710N and TL-WR810N)

(as of 2022Jun01)

* TLDR: How to build a [Gargoyle](https://gargoyle-router.com/) 1.13 image for the cordless [TP-Link travel routers TL-WR710N V1 and v2.1](https://openwrt.org/toh/tp-link/tl-wr710n) on _Debian 11_ #bullseye on _Windows Subsystem for Linux_ (#WSL #21H2).  
(Instructions for building for the [TL-WR810N V1 and v2](https://openwrt.org/toh/tp-link/tl-wr810n) are given, too, but haven't tried running the resulting image yet))
* Some links to start:  
a) Here's the [Gargoyle forum post about the TP-Link TL-WR710N](https://www.gargoyle-router.com/phpbb/viewtopic.php?f=13&t=14062) that lead to this description.  
b) Many thanks to Lantis and ispyisail for all their help and engagement!  
c) Start with reading the [Gargoyle Developer Documentation](https://www.gargoyle-router.com/wiki/doku.php?id=developer_documentation) > "Building from source".  
d) Plus the forum post [How-To Quick Build](https://www.gargoyle-router.com/phpbb/viewtopic.php?f=14&t=11883):
From later posts further down it seems that adding the repositories _universe_ and _multiverse_ is not necessary anymore.
* BTW: I've been using Debian 11 since that is recommended by the [OpenWrt build guide for WSL](<https://openwrt.org/docs/guide-developer/toolchain/wsl>) but it seems to run fine to try Ubuntu _20.04_ LTS on WSL, too. But, I haven't had success with Ubuntu _22.04_ yet.
* Install [Debian 11 "Bullseye"](https://www.microsoft.com/en-gb/p/debian/9msvkqc78pk6) (as of 24Apr2022) from the Microsoft store on your Windows Subsystem for Linux.
* When being asked for the normal used during the installation create one, e.g. called "pi", and then after logging in as this user run:  
        `sudo apt update && sudo apt upgrade`
* Then follow the section "Debian/Ubuntu" in the  [OpenWRT developer guide](<https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem>) for a 64bit system:  
        `sudo apt install build-essential gawk gcc-multilib flex git gettext libncurses5-dev libssl-dev python3-distutils zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo`  
        as well as:  
        `sudo apt install unzip file wget python2`  (for Debian, maybe also Ubuntu, as found out during "make'ing" further down in this Readme, as well as recommended in this post: <https://www.gargoyle-router.com/phpbb/viewtopic.php?f=14&t=11883>)  
        Ensure python executes python3:  
        `sudo apt install python-is-python3`  (This links python3 to python as found in the [TL-WR710N forum thread](<https://www.gargoyle-router.com/phpbb/viewtopic.php?p=60685#p60685>))  
        plus :  
        `sudo apt install npm nodejs`  (also strongly recommended in the Gargoyle docs)  
        `npm -v # in my case 7.5.2`  
        `nodejs -v # in my case 12.22.5`  
        and maybe also:  
        `sudo npm install terser -g`  (to avoid installation during build time, not fully verified/tested that)  
        `sudo npm install uglifycss -g`  

* Now prepare a writable build directory as 'root' for the user 'pi' in _/opt_:  
        `cd /opt && sudo mkdir gargoyle && sudo chown pi:pi gargoyle`  
    (BTW I also tried it on /mnt/c, but there the file ownerships can't be set to pi:pi...)
* From here on keep going as normal user, e.g. the user "pi" from above ...
* Follow the given recommendations, such as: Unset environment variables like SED and GREP_OPTIONS and aliases/functions (which should be ok on fresh installations)
* Now start creating the Gargoyle build environment in /opt:  
        `cd /opt`  
        `git clone https://github.com/ericpaulbishop/gargoyle.git`
* Gargoyle: Continue the [Gargoyle docs]([Gargoyle Developer Documentation](https://www.gargoyle-router.com/wiki/doku.php?id=developer_documentation)) at section "Building".
* Also read: <https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem> (already mentioned above)
    ... and especially section "Debian/Ubuntu" (python2 still seems to be necessary for Gargoyle, so I extended the list of additionally installed packages above)
* Also browsed though "Building Gargoyle on Windows 10 using WSL" (<https://www.gargoyle-router.com/phpbb/viewtopic.php?t=11378>) (posts from 2017) - however, I didn't find anything relevant.
* Verify that [dirs in $PATH have no spaces](<https://openwrt.org/docs/guide-developer/toolchain/wsl>) - a quick fix for non-root users if necessary:  
        `export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`
* Change to the build dir:  
        `cd gargoyle`
* If building fails further down, consider a [correction for the remote URL](<https://github.blog/2021-09-01-improving-git-protocol-security-github>):  
        `git remote set-url origin https://github.com/ericpaulbishop/gargoyle.git`
* Start the build - for the beginning choose a very limited target scope, e.g. for _ath79_ and _usb_:  
        `make FULL_BUILD=true ath79.usb`  
    (N.B. if you get error messages about missing packages, install these as mentioned above with "apt install npm ....")  
    BTW: The different _profiles_ were explained here:  
    <https://www.gargoyle-router.com/phpbb/viewtopic.php?p=60665#p60665>
* If you get the error msg _"The unauthenticated git protocol on port 9418 is no longer supported"_ then run (taken from <https://itsmycode.com/the-unauthenticated-git-protocol-on-port-9418-is-no-longer-supported>):  
        `git config --global url."https://github.com/".insteadOf git://github.com`  
    ... and rerun the make command above.  
* Have lunch, spend the afternoon, make dinner ... since building takes a few hours depending on your hardware...
* When done, hopefully the images were created in the subdir _images_:  
        `find images -ls`
* Since this here is about the small travel router _TP-Link TL-WR710N_ (versions 1 and 2.1), here are the special instructions for this one:      
        `s=ath79`  
        `for p in usb default ; do`  
        `tdir=/opt/gargoyle/targets/$s/profiles/$p`  
        `echo CONFIG_TARGET_DEVICE_${s}_generic_DEVICE_tplink_tl-wr710n-v1=y   >> $tdir/config # w tplink_!`  
        `echo CONFIG_TARGET_DEVICE_${s}_generic_DEVICE_tplink_tl-wr710n-v2.1=y >> $tdir/config`  
        `echo tplink_tl-wr710n-v1-squashfs   >> $tdir/profile_images # like the other ones in the file`  
        `echo tplink_tl-wr710n-v2.1-squashfs >> $tdir/profile_images`  
        `# echo tplink_tl-wr710n-v1            >> $tdir/profile_images # like from forum post 6298`  
        `# echo tplink_tl-wr710n-v2.1          >> $tdir/profile_images`  
        `grep -i wr710 $tdir/*`  
        `done`  
        `set +x`  

* EXPERIMENTAL alternative - including the builds for TL-WR810N v1 and v2 as well - these builds went fine, but haven't tried installation yet:  
        `set -x`  
        `for profile in usb   ; do # skip the architecture NAMED. default`  
                `for arch in ath79 ; do   # skip for now: ar71xx`  
                `tdir=/opt/gargoyle/targets/${arch}/profiles/${profile}`  
                `for device in tl-wr710n-v1 tl-wr710n-v2.1 tl-wr810n-v1 tl-wr810n-v2 ; do`  
                        `[ "$arch" = ath79 ] && device="tplink_$device"`  
                        `echo ARCH: $arch,  PROFILE: $profile,  DEVICE: $device`  
                        `echo CONFIG_TARGET_DEVICE_${arch}_generic_DEVICE_${device}=y  >> $tdir/config`  
                        `echo $([ "$arch" = ar71xx ] && echo -)${device}-squashfs   >> $tdir/profile_images`  
                `done`  
                `egrep -r -i 'wr710|wr810' $tdir`  
                `done`  
        `done`  

* Then (re)start the build:  
        `make ath79.usb   # NB: ... not repeating the full build...`  
    However, you should skip the full build only if you don't have a lot of patience - it will probably save you a few hours, but if it fails it might be more difficult to find out why....)

* If all goes well, the new images look like:

        pi@CSL:/opt/gargoyle$ find images -name \*-wr710\* -ls
        844424933069831    7936 -rw-r--r--   1 pi       pi     8126464 Apr 27 22:01 images/ath79/gargoyle_1.13.x-ath79-generic-tplink_tl-wr710n-v1-squashfs-factory.bin
        844424933069832    6980 -rw-r--r--   1 pi       pi     7144229 Apr 27 22:01 images/ath79/gargoyle_1.13.x-ath79-generic-tplink_tl-wr710n-v1-squashfs-sysupgrade.bin
        844424933069833    7936 -rw-r--r--   1 pi       pi     8126464 Apr 27 22:01 images/ath79/gargoyle_1.13.x-ath79-generic-tplink_tl-wr710n-v2.1-squashfs-factory.bin
        1125899909780490   6980 -rw-r--r--   1 pi       pi     7144233 Apr 27 22:01 images/ath79/gargoyle_1.13.x-ath79-generic-tplink_tl-wr710n-v2.1-squashfs-sysupgrade.bin

* Now, in order to flash the TL-WR710M build follow the instructions on ["OpenWrt installation using web UI"](<https://openwrt.org/toh/tp-link/tl-wr710n>) , i.e. I didn't install the new image directly but flashed an [older LEDE 17 image](<https://archive.openwrt.org/releases/17.01.5/targets/ar71xx/generic/>) first. (Don't know if flashing the Gargoyle image directly would have worked, too....)

* And then from LEDE 17 UI I flashed the newly built image _images/ath79/gargoyle_1.13.x-ath79-generic-tplink_tl-wr710n-v2.1-squashfs-sysupgrade.bin_ for my V2.1 router...

* A few minutes later, I was able to reach <http://192.168.1.1/> and to log into Gargoyle with the default password "password" ... 

* My use case:  
        I use the TL-WR710 as a very portable "Wifi Repeater" with a  _Wireless_ uplink and a _Client+AP_ wifi access point!  
        From the TL-WR710's USB port one can also power a simple GL-AR150 router with Freifunk which then gets it Internet uplink from TL-WR710's Ethernet LAN port...  
        BTW: To make both the Ethernet ports (LAN and LAN/WAN) available for LAN access to the Internet uplink, set the option "Wan Ethernet Port" to "Bridge To LAN" in the Gargoyle UI !.

* Once agaain: Big thanks to all the people that make Gargoyle and OpenWrt work!

* Note: Older instructions from 2019 for building on Ubuntu Server 18.04 are here: <https://www.gargoyle-router.com/phpbb/viewtopic.php?f=14&t=11883> - they might be out of date.

* Note: Beware that OpenWrt's config variables and settings are different for the different architectures and profiles! So here are the ones for the architecture _ar71xx_ for the "TP-Link TL-WR710N" (both versions 1 and 2.1) - The build for the profile _usb_ went fine for me, I haven't tried _default_....:  
        `# export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`  
        `set -x`  
        `s=ar71xx`  
        `for p in usb default ; do`  
        `tdir=/opt/gargoyle/targets/$s/profiles/$p`  
        `echo CONFIG_TARGET_DEVICE_${s}_generic_DEVICE_tl-wr710n-v1=y   >> $tdir/config # no tplink_ !`  
        `echo CONFIG_TARGET_DEVICE_${s}_generic_DEVICE_tl-wr710n-v2.1=y >> $tdir/config`  
        `echo tl-wr710n-v1-squashfs   >> $tdir/profile_images  # like the other ones in the file`  
        `echo tl-wr710n-v2.1-squashfs >> $tdir/profile_images`  
        `echo tl-wr710n-v1            >> $tdir/profile_images # derived from forum post 6298`  
        `echo tl-wr710n-v2.1          >> $tdir/profile_images`  
        `grep -i wr710 $tdir/*`  
        `done`  
        `set +x`  
        `# make ar71xx.usb`

        pi@CSL:/opt/gargoyle$ find images -name \*-wr710\* -ls
        2251799816175603   8576 -rw-r--r--   1 pi       pi     8126464 Apr 28 23:15 images/ar71xx/gargoyle_1.13.x-ar71xx-generic-tl-wr710n-v1-squashfs-factory.bin
        1688849862754293   7552 -rw-r--r--   1 pi       pi     7143428 Apr 28 23:15 images/ar71xx/gargoyle_1.13.x-ar71xx-generic-tl-wr710n-v1-squashfs-sysupgrade.bin
        2533274792886273   8512 -rw-r--r--   1 pi       pi     8126464 Apr 28 23:15 images/ar71xx/gargoyle_1.13.x-ar71xx-generic-tl-wr710n-v2.1-squashfs-factory.bin
        1970324839464962   8832 -rw-r--r--   1 pi       pi     7143428 Apr 28 23:15 images/ar71xx/gargoyle_1.13.x-ar71xx-generic-tl-wr710n-v2.1-squashfs-sysupgrade.bin

