# How to port TWRP to SurfTab breeze 10.1 quad plus (ST10408-11)

## Using Arch Linux (6.13.1-arch1-1) in February 2025  



### 0.) Foreword

I recently just felt like giving this another go. When I tried using this by following a tutorial that used Carlvic I just did not get some files that are mandatory. Funnily, just about one week after I released the tutorial to root this device, some tool that makes porting TWRP almost piss easy got released.  
There are some general guides using it (e.g. [here](https://xdaforums.com/t/building-twrp-for-an-unsupported-unknown-device.4699437/) and
[here](https://xdaforums.com/t/guide-zero-to-hero-what-to-do-when-your-phone-has-nothing-no-twrp-no-custom-roms.4627809/) including some videos.)  
Another thing to note is that AlaskaLinuxUser recommended in a video guide building for some known device like shamu (Nexus 6) first to be safe that everything is set up correctly but the funny thing is that I actually wasn't able to succeed in building for that; in the process I contined to get errors when I didn't in the process for our device anymore.  

By the way, lots of thanks towards Athwale (white-bear.cz). If it wasn't for that person, I might have given up using the new tool.  



### 1.) Prerequisites

I'm assuming you have already followed the tutorial to root the device.
It might not be necessary to actually have the device rooted but it definitely helps; if you followed this tutorial you should have made a backup of the recovery partition that we are going to use.  

You will definitely need some way to run some **python2** stuff, at least in a virtual environment. Can't tell you exactly what packages you need or if you need to install something via pip for it; you might figure this out at the point when it is needed or ask an LLM (e.g. at duck.ai) to help you install it on Arch or to even port stuff to python3.  

You need OpenJDK 7 as mentioned in the [doc for building Android](https://source.android.com/docs/setup/start/older-versions); TWRP will complain if you use others.  
After running `archlinux-java status` you can check if **java-7-openjdk** is listed.  
If you don't have it:  
`sudo pacman -U https://archive.archlinux.org/packages/j/jdk7-openjdk/jdk7-openjdk-7.u261_2.6.22-1-x86_64.pkg.tar.zst`  

And last but not least, you will need the repo tool:  
`sudo pacman -S repo`



### 2.) Get device tree from recovery image

(This is about the newer fancy tool I mentioned in the foreword. See how short this section is? It's awesome.)  

`git clone https://github.com/twrpdtgen/ && cd twrpdtgen`  
Then make modifications as described in this issue here:  
https://github.com/twrpdtgen/twrpdtgen/issues/217#issuecomment-2636832637  
To install after adding modifications:  
`poetry build; pip install dist/*.whl --break-system-packages --force-reinstall`  

Then you can use it anywhere. Copy our backup of the recovery partition (in our case named ROM_10) anywhere. Then run:  
`python3 -m twrpdtgen ROM_10`  
You will see some output; at the end something like:  
_Done! You can find the device tree in /home/duda/mnt1/AndroidDEV/output/trekstor/ST10408-11_  
(while the part after home/ and before /output will be different for you; I have a user called duda and just created two sub directories in its home directory)  


### 3.) Preparing the build
(...using Android 5.1 TWRP minimal manifest & python virtual environment)  

```
mkdir minTWRPOmni511 && cd minTWRPOmni511
repo init -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni.git -b twrp-5.1
repo sync
```
(After that you will see 35 GB of storage used. Maybe less if you use shallow clones. The repo mentioned in the commands contains a bit of info in the README that might be worth a read.)  

Copy our device tree in:  
`cp ../output/ minTWRPOmni511/device`  


Usually you should now be able to just build, but there will be some issues.  
E.g. there is a bug in `lunch`; it can't handle the hyphen-minus (-) symbol in our device codename. So I just replaced it with a underscore/low line (_):  
```
cd minTWRPOmni511/device/trekstor/ST10408-11
mv omni_ST10408-11.mk omni_ST10408_11.mk
sed -i 's/ST10408-11/ST10408_11/g' setup-makefiles.sh BoardConfig.mk device.mk vendorsetup.sh AndroidProducts.mk extract-files.sh omni_ST10408_11.mk Android.mk
cd ..
mv ST10408-11/ ST10408_11/
cd ../../
```

Interesting to us is the _device/trekstor/ST10408_11/recovery.fstab_.
I tried building TWRP without modifying that and it build it after I fixed some issues which I will tell you how I fixed them in a moment.  
To get correct partitions extracting using [Carlvic](https://github.com/LowTension/carliv_image_kitchen/tree/linux64) and checking `cat /proc/emmc` (file [here](proc_emmc)) on my device as well as comparing recovery.fstab files for other devices helped a bit but a lot have by-name partition inside, took a while until I found out I could also e.g. use /dev/bootimg, I think that was using in a file for another MTK device.  
(My [recovery.fstab](recovery.fstab). Feel free to notify me if that can be improved. E.g. I just wasn't able to get the recovery partition to show up as a backup option in TWRP so far; flashing img to it works...  
I used two auto partitions for microSD card since I use Apps2SD to store app data on SD card and various partition types can be used...).  


Now getting close to building the thing. Get a virtual python2 environment:  
```
python2 -m virtualenv myenv
source myenv/bin/activate
```
You should see (myenv) in front of your terminal prompt string.
Then basically run the commands similar to mentioned in the minimal manifest repo.  
Just to be extra sure I also exported LC_ALL since I had issues with that not being set when compiling kernels in the past — probably cause I don't use a english locale but a german one...  
```
export LC_ALL=C
export ALLOW_MISSING_DEPENDENCIES=true
. build/envsetup.sh
lunch omni_ST10408_11-eng # or lunch, (probably) 9 & enter
```

I got this output (the warning can be ignored):  
```
WARNING: device/trekstor/ST10408_11/omni.dependencies file not found

============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=16.1.0
TARGET_PRODUCT=omni_ST10408_11
TARGET_BUILD_VARIANT=eng
TARGET_BUILD_TYPE=release
TARGET_BUILD_APPS=
TARGET_ARCH=arm
TARGET_ARCH_VARIANT=armv7-a-neon
TARGET_CPU_VARIANT=generic
TARGET_2ND_ARCH=
TARGET_2ND_ARCH_VARIANT=
TARGET_2ND_CPU_VARIANT=
HOST_ARCH=x86_64
HOST_OS=linux
HOST_OS_EXTRA=Linux-6.13.1-arch1-1-x86_64-with-glibc2.34
HOST_BUILD_TYPE=release
BUILD_ID=LYZ28N
OUT_DIR=/home/duda/mnt1/AndroidDEV/minTWRPOmni511/out
============================================
```

(If you changed something you might need to re-run the envsetup and lunch.)   



### 3.) Actually building ✨

To finally build the recovery image (and to have some unwanted fun of fixing upcoming bugs...)  
`mka recoveryimage`  
**I re-run this once every time it did not build recovery.img**  

There could be additional issues you encounter when run this in the future. I cover what I encountered at the time of writing. If you had an environment as recommended by Google then you would likely not encounter that many, if any. Problem is just that their Ubuntu Docker image is broken... :)  

Well, first thing that grabbed my attention:  
```
external/checkpolicy/checkpolicy.c: In function 'display_bools':
external/checkpolicy/checkpolicy.c:294:23: warning: comparison of integer expressions of different signedness: 'int' and 'uint32_t' {aka 'unsigned int'} [-Wsign-compare]
  294 |         for (i = 0; i < policydbp->p_bools.nprim; i++) {
      |                       ^
external/checkpolicy/checkpolicy.c: In function 'main':
external/checkpolicy/checkpolicy.c:470:48: warning: comparison of integer expressions of different signedness: 'unsigned int' and 'long int' [-Wsign-compare]
  470 |                                 if (policyvers != n)
      |                                                ^~
external/checkpolicy/checkpolicy.c:665:29: error: implicit declaration of function 'isdigit' [-Wimplicit-function-declaration]
  665 |                         if (isdigit(ans[0])) {
      |                             ^~~~~~~
external/checkpolicy/checkpolicy.c:86:1: note: include '<ctype.h>' or provide a declaration of 'isdigit'
   85 | #include "parse_util.h"
  +++ |+#include <ctype.h>
   86 |
external/checkpolicy/checkpolicy.c:448:25: warning: this statement may fall through [-Wimplicit-fallthrough=]
  448 |                         usage(argv[0]);
      |                         ^~~~~~~~~~~~~~
external/checkpolicy/checkpolicy.c:449:17: note: here
  449 |                 case 'M':
      |                 ^~~~
```
... which I basically "solved" by including the ctype.h in `external/checkpolicy/checkpolicy.c` (by removing the ifdef and endif lines surrounding it — at currently line 73 and 75 — that prevent it from being included when not on darwin, lol:  
```
#ifdef DARWIN
#include <ctype.h>
#endif
```
)  

After another set of re-runs I got:  
```
make: *** [build/core/host_static_library_internal.mk:27: /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/host/linux-x86/obj32/STATIC_LIBRARIES/libselinux_intermediates/libselinux.a] Error 1
make: *** Waiting for unfinished jobs....
collect2: error: ld returned 1 exit status
make: *** [build/core/host_executable_internal.mk:31: /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/host/linux-x86/obj32/EXECUTABLES/checkpolicy_intermediates/checkpolicy] Error 1
```
After "No such file or directory" and "first defined here" lines that should be ignored according to one of the guides linked at the beginning...  
But when searching for **"checkpolicy] Error 1"** by using a search engine I stumpled upon https://stackoverflow.com/questions/78715321/missing-pcreperl-compatible-regular-expressions-object-files-while-building-tw. Before reading this I did not notice the **"/bin/bash: line 1"** things at the start of some lines that end with "first defined here".  
Following this site, I changed the  
```
filelist="$$filelist $$ldir/$$f"; \
```
line at 1223 within `build/core/definitions.mk` to  
```
filelist="$$filelist $$f"; \
```
(basically removing the `$$ldir/` part)  

Next double re-run:  
```
/usr/bin/ld: /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/host/linux-x86/obj32/EXECUTABLES/checkpolicy_intermediates/checkpolicy.o:/home/duda/mnt1/AndroidDEV/minTWRPOmni511/external/checkpolicy/checkpolicy.h:16: multiple definition of `te_assertions'; /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/host/linux-x86/obj32/EXECUTABLES/checkpolicy_intermediates/policy_define.o:/home/duda/mnt1/AndroidDEV/minTWRPOmni511/external/checkpolicy/checkpolicy.h:16: first defined here
/usr/bin/ld: /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/host/linux-x86/obj32/EXECUTABLES/checkpolicy_intermediates/policy_parse.o:/home/duda/mnt1/AndroidDEV/minTWRPOmni511/external/checkpolicy/checkpolicy.h:16: multiple definition of `te_assertions'; /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/host/linux-x86/obj32/EXECUTABLES/checkpolicy_intermediates/policy_define.o:/home/duda/mnt1/AndroidDEV/minTWRPOmni511/external/checkpolicy/checkpolicy.h:16: first defined here
```
I found https://lists.yoctoproject.org/g/yocto/topic/meta_selinux_patch/74954287 which basically proposes to remove the following stuff (which I just turned into comments by using `/*` and `*/`) from `external/checkpolicy/checkpolicy.h`:  
```
/*
#include <sepol/policydb/ebitmap.h>

typedef struct te_assert {
        ebitmap_t stypes;
        ebitmap_t ttypes;
        ebitmap_t tclasses;
        int self;
        sepol_access_vector_t *avp;
        unsigned long line;
        struct te_assert *next;
} te_assert_t;

te_assert_t *te_assertions;
*/
```

New interesting stuff after re-run:  
```
bootable/recovery/partition.cpp:1464:24: error: use of undeclared identifier 'nullptr'
            if (sml != nullptr) {
                       ^
bootable/recovery/twrp-functions.cpp:1522:10: warning: 'auto' type specifier is a C++11 extension [-Wc++11-extensions]
                                for (auto it = backups.begin(); it != backups.end(); it++) {
                                     ^
```
For the **nullptr** thing within `bootable/recovery/partition.cpp` at line 1464 I just went with using **NULL** there instead of the nullprt (but there might be a better solution https://stackoverflow.com/questions/24433436/compile-error-nullptr-undeclared-identifier).  
The **auto** thing I just left as it is after I got errors due to changing it.  

Re-run:  
```
target Executable: recovery (/home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/obj/EXECUTABLES/recovery_intermediates/LINKED/recovery)
target Symbolic: recovery (/home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/symbols/recovery/root/sbin/recovery)
target Strip: recovery (/home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/obj/EXECUTABLES/recovery_intermediates/recovery)
Install: /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/recovery/root/sbin/recovery
sed -i "s/{themeversion}/5/" /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/recovery/root/twres/splash.xml; sed -i "s/{themeversion}/5/" /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/recovery/root/twres/ui.xml;
----- Making recovery filesystem ------
Copying baseline ramdisk...
Modifying ramdisk contents...
sed -i 's/ro.build.date.utc=.*/ro.build.date.utc=0/g' /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/recovery/root/default.prop
sed -i 's/ro.adb.secure=1/ro.adb.secure=0/g' /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/recovery/root/default.prop
----- Made recovery filesystem -------- /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/recovery/root
----- Making uncompressed recovery ramdisk ------
/home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/host/linux-x86/bin/mkbootfs /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/recovery/root > /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/ramdisk-recovery.cpio
----- Making recovery ramdisk ------
/home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/host/linux-x86/bin/minigzip < /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/ramdisk-recovery.cpio > /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/ramdisk-recovery.img
----- Making recovery image ------
/home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/recovery.img maxsize=17031168 blocksize=135168 total=12191744 reserve=270336
----- Made recovery image: /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/recovery.img --------
```
We got recovery.img!  
([Here](recovery.img) is mine btw.)  

Flash it:  
```
cp /home/duda/mnt1/AndroidDEV/minTWRPOmni511/out/target/product/ST10408_11/recovery.img recovery.img
adb reboot bootloader
fastboot flash recovery recovery.img
```

Now you should be able to always boot into recovery by either pressing a key combo I don't know for sure for this device, by using `adb reboot recovery` or by rebooting into it by just going from Magisk manager app.  

**Please note, that I'm still not a expert in this, as you probably should have guessed already when following this guide**  

### Also keep hating Trekstor, even tho it doesn't exist anymore (but you can avoid the new company that is run by the Szmigiel family... tho you likely barely heard of it. But it's funny to research what kind of companies Shimon founded and ruined :D... If you happen to know that guy, any of the Trekstor people that have contacts or anybody from ktc.cn — cause [ro.product.ota.host]: [update.ktc.cn:2400] —, bug them about kernel source and feel free to mention it to me)  

**Enjoy TWRP on your old tablet! ~~And see you in 2030 when I got a custom ROM running on it, hopefully Android 6+ even; tho maybe earlier, at least for another 5.1 one...~~**  



(Video of the device booting into TWRP and unlocking it:)  
