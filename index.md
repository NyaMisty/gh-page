## Welcome to my repo :)

### About KernBypass's fakevar version

#### Before start
- This version is VERY unstable and may panic you system, and there ARE data-loss risk as this tweak will make the normal reboot/shutdown process fail, which may cause the filesystem failing (so if you still want to use this, don't restart your phone that much!)

#### Usage

0. OFC, download the modified KernBypass :)
1. download the [fakevar.zip](http://repo.misty.moe/apt/fakevar13.zip), and extract it to /var/mobile/fakevar (whatever method you use to extract, make sure there's /var/mobile/fakevar/mobile)
2. execute `preparerootfs`
3. execute `changerootfs &; disown %1`
4. fin :)

