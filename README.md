Important: Follow these instructions meticulously

Note: The OP is using Wheezy and the latest kernel version from raspberrypi-bootloader then is 4.1.7-v7+, while on jessie that version is 4.1.13-v7+ and that's a critical difference!
If you're going to apply these steps to your own situation, adjust the values accordingly.

Step 1: Get the git_hash from the firmware
Get the firmware-commit-id: zgrep '* firmware as of' /usr/share/doc/raspberrypi-bootloader/changelog.Debian.gz | head -1 which returns 960832a6c2590635216c296b6ee0bebf67b21d50 or 960832a for short.
On https://github.com/raspberrypi/firmware/commits/master you look for that firmware-commit-id (960832a on 23 Sep 2015 (the date is only as extra helper, the only relevant thing is the firmware-commit-id)) and click on Browse the repository at this point in the history and switch to the extra directory. When you're there, download Module7.symvers and save it in your home directory (for the Pi 1, you'd download Module.symvers).
In the extra directory you'll also find a file git_hash which you should open and write down the id (59e76bb7e2936acd74938bb385f0884e34b91d72) you see there. This is the kernel-commit-id.

Step 2: Prepare the kernel for module compilation
Get the kernel sources and put them into the rpf-linux-kernel folder:
git clone https://github.com/raspberrypi/linux rpf-linux-kernel
This will download more then 1GB of data and then process it, which will take a while.

To prepare the sources for kernel module compilation there are some other steps to do.
First you need to checkout the kernel code at the exact kernel-commit-id you found earlier. I always do my 'work' in a separate branch instead of in 'master' as that makes it easier to start over in case things go wrong and it is considered best practice. Then you clean everything up (mrproper), load the default configuration for the Pi 2 (bcm2709_defconfig) and prepare the sources for module compilation (modules_prepare):

cd rpf-linux-kernel
git checkout -b rpi-bootloader-4.1.7 59e76bb7e2936acd74938bb385f0884e34b91d72
make mrproper
make bcm2709_defconfig
make modules_prepare
For the Pi 1 you should replace bcm2709_defconfig with bcmrpi_defconfig.

Now copy the Module7.symvers file you downloaded earlier into the kernel tree and rename it to Module.symvers: cp ../Module7.symvers Module.symvers
(for the Pi 1, just copy the Module.symvers into the kernel tree).

Step 3: Set up your system for kernel module compilation
When you're compiling a kernel module, the build system looks in the /lib/modules/<kernel-version>/build directory for the kernel headers/sources. So make the link from that build directory to our current directory:
sudo ln -s /home/pi/rpf-linux-kernel/ /lib/modules/$(uname -r)/build

When you now do ls -l /lib/modules/4.1.7-v7+/build you should get the following:
lrwxrwxrwx 1 root root 32 dec 15 21:38 /lib/modules/4.1.7-v7+/build -> /home/pi/rpf-linux-kernel/

And now you're ready to compile a kernel module.

Step 4: Compiling the kernel module
You're lucky as someone has posted the source file and a Makefile to compile the module.
To compile it, go back to your home directory, clone the git repository and make/install the driver:

cd ~
git clone https://github.com/xor-function/usb-ethernet-adapter
cd usb-ethernet-adapter
make
sudo make install
And then to load it, do sudo modprobe ch9200.
As the USB ID is part of the driver code, when you plug the device in it should detect it automatically and load the driver.
