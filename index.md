## Welcome to my repo :)

### [Add to cydia](cydia://url/https://cydia.saurik.com/api/share#?source=http://repo.misty.moe/apt/)
The repo url is http://repo.misty.moe/apt

### About KernBypass's fakevar version

#### Before start
- This version is VERY unstable and may panic you system, and there ARE data-loss risk as this tweak will make the normal reboot/shutdown process fail, which may cause the filesystem failing (so if you still want to use this, don't restart your phone that much!)

#### Usage

0. OFC, download the modified KernBypass :)
1. download the [fakevar.zip](http://repo.misty.moe/apt/fakevar13.zip), and extract it to /var/mobile/fakevar (whatever method you use to extract, make sure there's /var/mobile/fakevar/mobile)
2. execute `preparerootfs`
3. execute `changerootfs & ; disown %1`
4. fin :)

#### How it works???

1. The original KernBypass is charming, which works pretty well for pokemon go
2. Why? Because KernBypass only chrooted the /, and then direcyly linked /var into the chroot jail. 
    - However, many jailbreak app stores their configuration in places like: /var/lib, /var/mobile/Library, /var/mobile/Library/Caches
    - and those app includes the Cydia and apt, so we got killed
3. So how to cope with these checks?
    - we can make a fake /var userdata, and only link those directory we want it to present in /var into chroot

#### Why it's difficult?
In fact there's pretty much:
- If we link /var/ to a path in the already mounted /var or /, there will be circular references, making the chroot jail freeze
- If we use devfs, no file could be created in devfs
- If we attach the dmg into chroot and link it, then there's some magic will cause ReportCrash daemon to crash the whole system

Considering all the above approaches, the only one that's working and stable is devfs.

#### Known issue

- App with AppGroup simply doesn't work now. (will crash due to [NSFileManager containerURLForSecurityApplicationGroupIdentifier:], seems it's a appgroup related issue)
- Currently not work with Chimera due to Sileo storing into some strange directory, but may be fixed
- The opening speed of apps will slow down a lot, beware :)

#### For developer

The source is available at https://github.com/NyaMisty/KernBypass-Public

And there's an AnalysisResults directory, hope you can find some useful information to improve this there :)

