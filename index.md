## Welcome to my repo :)

### [Add to cydia](cydia://url/https://cydia.saurik.com/api/share#?source=http://repo.misty.moe/apt/)
The repo url is http://repo.misty.moe/apt

### About KernBypass's iOS 14 arm64e version

- For DEV only
- Tested on iOS 14.2 & 14.3

#### Usage

##### Installation
1. Download the attach-detach, mount_bindfs, kernbypass in myrepo
2. Download flexdecrypt, plutil (in Packix and bingner)
3. Download applink (in https://repo.estertion.win/ )
4. Download http://repo.misty.moe/apt/kernbypass-resource.tar
5. Extract the tar into /var/mobile ( ensure that /var/mobile/kernbypass_asset ) is present

##### On the first run:
1. Run `mksparsedmg /var/mobile/kernbypass_asset/kernbypass.dmg` (either in mobile shell or root shell is OK)

##### To enable the kernbypass:
1. In the root shell, cd into /var/mobile, and execute `./preparefullroot.sh`
2. Then just like the old kernbypass, use the changerootfs

##### To enable kernbypass for a new application
1. Enable that app in settings of kernbypass
2. Before launching of the app, execute `./kernbypass.sh "Your app's bundle ID"`, like `./kernbypass.sh jp.co.cygames.princessconnectredive`, bundle ID can be found in apps manager)
3. When the app updated, run above command again.

#### Technical Details
KernBypass is working by chrooting an app into a fakeroot. The fakeroot is built by linking the required directory. On iOS13, this is achieved by swapping all fields of target folder's vnode. However on iOS14, with PAC enabled, the system will now check if pointers in vnode matches with the vnode. Thus we can't simply swapping the vnode. So, we have to find an alternative way to link the folder. 

Actually, I really found it. For every linked folder, they are either read only (system libraries), or read write (data or temp files). For a normal application, they do not need the temp file, and the only writable location for them is simply the app data container.

For RO folders, after analyzed the firmlink/unionmount/bindfs, only bindfs a bindfs filesystem (which you can see in the default mount list), which can be used to to bind two folders (RO only).

But bindfs cannot bind as RW folder. But the RW folders are all data container folders(/var/mobile/Containers/Data/Application, so I decided to mount a dmg on these locations. 


And after fixing Attach-Detach, I'm now able to mount it. However the it seems that the app cannot access the new root like it used to be. And that's because we didn't switched the vnode, so the path that system seen is a path that app shouldn't be able to access. Normally, we can simply remove all sandbox to grant app all access to everywhere. We(I and @XsF1re) have tried that, but sadly iOS14 introduced IOKit hardening, and by default an executable cannot open any IOService, so all game that uses IOGPU will fail :(. We also tried to resign the app with sandbox exemption profile, however we also failed, as system will simply ignore all sandbox exemption rule for all executables located in user bundle path (/bar/containers/)

To fix this, I investigated the app launching process, and hooked the runningboardd, then redirected app's executable to a copy in system application (/Applications/DemoApp.app/kernbypass), and in this way we can make the sandbox to allow the exemption.

Then I found that the iOS will assign a brand new container to app each time it updates and moving the container's content, but as we are in fakeroot, the container won't be moved. So I instead keep the app's container in a fixed location, and link to the actually sandbox caontainer path each time the app updates.

Now chaining all these pieces, we can build a new KernBypass.

### About KernBypass's fakevar version (iOS qe, obsoleted)

#### Before start
- This version is not that stable and MAY panic you system, and there ARE data-loss risk as this tweak will make the normal reboot/shutdown process fail, which may cause the filesystem failing (so if you still want to use this, don't restart your phone that much!)
- Currently we only tested with 13.5, so other version might not work. Feel free to make issue on GH~

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

