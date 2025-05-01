# How to compile a kernel for the Galaxy Grand Duos (baffinvj)

## Using Arch Linux in May 2025  



### 0.) Foreword

I think that this should work for other baffin variants as well.  
Some are here: https://desktop.firmware.mobi/device:1330  
My version specifically got released in Thailand. That's why there is the L at the end of product version (L for latin america?):  
`adb shell getprop | grep platform`
```
[ro.build.product]: [baffin]
[ro.product.board]: [capri]
[ro.product.brand]: [samsung]
[ro.product.cpu.abi2]: [armeabi]
[ro.product.cpu.abi]: [armeabi-v7a]
[ro.product.device]: [baffin]
[ro.product.locale.language]: [en]
[ro.product.locale.region]: [GB]
[ro.product.manufacturer]: [samsung]
[ro.product.max_num_touch]: [2]
[ro.product.model]: [GT-I9082L]
[ro.product.multi_touch_enabled]: [true]
[ro.product.name]: [baffinvj]
[ro.product_ship]: [true]
```
(if you got several devices connected, `export ANDROID_SERIAL`)
For this device I've got GT-I9082_MEA_JB_Opensource.zip from https://opensource.samsung.com/.  
I actually asked them late January 2024 to upload kernel source code for my device cause they already had it for SEA.  
And compiling it worked for me but I figured there would be no harm in asking for the minor different regional version for my device. And guess what? THEY (RE-)UPLOADED GT-I9082_MEA_JB_Opensource.zip and kept it until earlier this year I think. They actually spent time for a device that was already a bit over ten years old. I really never liked Samsung that much for overpriced devices but there aren't many companies that just deliver source code with barely any trouble.  



### 1.) Preparing the environment

This is an older armv7 device. You might have seen tutorials where ARCH=arm and CROSS_COMPILE are set. You won't need this in this case when just putting the toolchain at the correct place.  
`adb shell getprop | grep cpu`
```
[ro.product.cpu.abi2]: [armeabi]
[ro.product.cpu.abi]: [armeabi-v7a]
```
```
git clone https://android.googlesource.com/platform/prebuilts/gcc/linux-x86/arm/arm-eabi-4.6 
cd arm-eabi-4.6 
git checkout -f d73a051b1fd1d98f5c2463354fb67898f0090bdb
cd ..
mv arm-eabi-4.6/ /opt/toolchains/
```
Why the checkout is needed? Cause Google wants you to use clang.  
No idea how to make this work for compiling old Android devices & this GCC is known to work.  

Also no idea anymore on which additional packages you might need on Arch but maybe this [setup doc for Ubuntu](https://source.android.com/docs/setup/start/older-versions) or the Arch Wiki sites on kernel compiling can give you a hint.  
However, you should really install `android-sdk-platform-tools` for getting e.g. [adb‌](https://developer.android.com/tools/adb) in the unlikely case that you don't already have it. And at the very end you will also need something to unpack and repack boot image.

Well, don't worry for now and just extract the kernel source files:
```
unzip GT-I9082_MEA_JB_Opensource.zip
tar xf Kernel.tar.gz
tar xf Platform.tar.gz
```

To get english output (and maybe less issues... cause I had kernels that only compiled when I had this set!):
`export LANG=C`



### 2.) Compiling the kernel

In `./arch/arm/configs/` are a lot of defconfigs.  
You might recall capri from earlier getprop output. Let's grep for that:  
`adb getprop | grep capri`
```
[ro.board.platform]: [capri]
[ro.chipname]: [capri]
[ro.hardware]: [capri_ss_baffin]
[ro.product.board]: [capri]
```
The string for ro.hardware appears in various defconfig names. Let's list those:  
`find . -name "*capri_ss_baffin*"`
I got the following 18 results from that:
```
./arch/arm/configs/bcm28155_capri_ss_baffin_cu_rev05_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_cmcc_rev03_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffinss_cmcc_rev03_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_cu_rev02_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_cu_rev03_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_cmcc_rev02_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_rev01_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_rev02_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_rev05_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_cmcc_rev00_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffinss_rev04_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_rev00_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_cu_rev01_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_cu_rev00_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_cmcc_rev05_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_cmcc_rev01_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_rev03_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffinss_rev03_defconfig
```
What I noticed is that there are various rev(isions). And I think you can also grep for that?:  
`adb getprop | grep revision`
```
[ro.revision]: [5]
```
So quickly added the 5 to our find command:  
`find . -name "*capri_ss_baffin*5*"`
```
./arch/arm/configs/bcm28155_capri_ss_baffin_cu_rev05_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_rev05_defconfig
./arch/arm/configs/bcm28155_capri_ss_baffin_cmcc_rev05_defconfig
```

So I run `make bcm28155_capri_ss_baffin_rev05_defconfig`
Don't worry about this error, it will only show on the first make (until clean):
```
  HOSTCC  scripts/basic/fixdep
  HOSTCC  scripts/kconfig/conf.o
  HOSTCC  scripts/kconfig/zconf.tab.o
In Datei, eingebunden von scripts/kconfig/zconf.tab.c:2503:
In Funktion »dep_stack_insert«,
    eingefügt von »sym_check_print_recursive« bei scripts/kconfig/symbol.c:977:3,
    eingefügt von »sym_check_deps« bei scripts/kconfig/symbol.c:1148:3:
scripts/kconfig/symbol.c:953:19: Warnung: Adresse der lokalen Variable »cv_stack« wird in »check_top« gespeichert [-Wdangling-pointer=]
  953 |         check_top = stack;
      |         ~~~~~~~~~~^~~~~~~
scripts/kconfig/symbol.c: In Funktion »sym_check_deps«:
scripts/kconfig/symbol.c:974:26: Anmerkung: »cv_stack« ist hier deklariert
  974 |         struct dep_stack cv_stack;
      |                          ^~~~~~~~
scripts/kconfig/symbol.c:944:4: Anmerkung: »check_top« ist hier deklariert
  944 | } *check_top;
      |    ^~~~~~~~~
  HOSTLD  scripts/kconfig/conf
arch/arm/mach-capri/custom_boards/Kconfig:19:warning: defaults for choice values not supported
arch/arm/mach-capri/custom_boards/Kconfig:25:warning: defaults for choice values not supported
arch/arm/mach-capri/custom_boards/Kconfig:31:warning: defaults for choice values not supported
arch/arm/mach-capri/custom_boards/Kconfig:37:warning: defaults for choice values not supported
#
# configuration written to .config
#
```

Continue with `make`.
You will run into a real issue ([reason looks to be here](https://github.com/torvalds/linux/commit/e33a814e772cdc36436c8c188d8c42d019fda639)):
```
/usr/bin/ld: scripts/dtc/dtc-parser.tab.o:(.bss+0x30): multiple definition of `yylloc'; scripts/dtc/dtc-lexer.lex.o:(.bss+0x0): first defined here
collect2: error: ld returned 1 exit status
make[2]: *** [scripts/Makefile.host:127: scripts/dtc/dtc] Error 1
make[1]: *** [scripts/Makefile.build:441: scripts/dtc] Error 2
make: *** [Makefile:507: scripts] Error 2
```
Thankfully that one is common and a easy fix turns up when using a search engine.  
One way is to add `extern` in front of `YYLTYPE yylloc;`.  
I found that adding that to `scripts/dtc/dtc-lexer.l` and `scripts/dtc/dtc-lexer.lex.c_shipped` works.  
If you are lazy and don't want to open the file to change a single line then you can use this one-liner:  
`sed -i '/^YYLTYPE yylloc;/ s/^/extern /' scripts/dtc/dtc-lexer.l`  
Then continue with `make`. Which presents us with this awesome problem after a moment:  
```
Can't use 'defined(@array)' (Maybe you should just omit the defined()?) at kernel/timeconst.pl line 373.
make[1]: *** [/home/duda/AndroidDevices/BCM28155/Samsung/Galaxy_Grand_Duos/kernel/2025/kernel/Makefile:140: kernel/timeconst.h] Error 255
make: *** [Makefile:946: kernel] Error 2
```
You may be able to use Perlbrew to get a old perl version! (5.18 maybe?)  
But I opted to edit kernel/timeconst.pl and replaced line 373.  
`if (!defined(@val)) {`
with
`if (!@val) {`.
And here is your one-liner you have waited for:
`sed -i '373s/if (!defined(@val)) {/if (!@val) {/g' kernel/timeconst.pl`
`make` again.
And owo what's this? (25 lines from below): 
```
  OBJCOPY arch/arm/boot/zImage
  Kernel: arch/arm/boot/zImage is ready
```
Ayo it's the kernel!



### 2.) Putting the kernel on the device

Create and pull a custom named boot image from the boot partition of this phone:
`adb shell su -c 'dd if=/dev/block/mmcblk0p5 of=/sdcard/boot_grandduos.img' && adb pull /sdcard/boot_grandduos.img`
Then I use a tool (like from https://github.com/pbatard/bootimg-tools) to unpack it:
`unmkbootimg -i boot_grandduos.img`
Now you will see `kernel` and `ramdisk.cpio.gz` on listing via e.g. `ls`.
Replace the kernel with our kernel:
`cp ./arch/arm/boot/zImage kernel`
And then just make a new boot image by running e.g. the command that the tool for unpacking gave us (but with a different output name):
`mkbootimg --base 0 --pagesize 4096 --kernel_offset 0xa2008000 --ramdisk_offset 0xa3000000 --second_offset 0xa2f00000 --tags_offset 0xa2000100 --cmdline 'console=ttyS0,115200n8 mem=832M@0xA2000000 androidboot.console=ttyS0 vc-cma-mem=0/176M@0xCB000000' --kernel kernel --ramdisk ramdisk.cpio.gz -o my_boot_for_grandduos.img`

Now there might be a few ways on how to bring our freshly baked boot image onto the device...  
If you got a recovery like CWM (or newer TWRP) that you can reach via Power+UP+Home or `adb reboot recovery` onto it already, then just
`adb push my_boot_for_grandduos.img /sdcard`
and flash it from there.  
Odin or heimdall might also work but I'm not sure.  

Next time I will build the recovery. So in case you don't already have it... you will get it.

**Enjoy your kernel!**

