# Install repo 

To get the BSP you need to have `repo` installed and use it as:

Install the `repo` utility:

```
$ mkdir ~/bin
$ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
```

# 2. Download BSP source for image

After installed `repo` tool, we can download BSP source to use some commands as below:

```
git config --global credential.helper 'cache --timeout=3600'
GIT_BRANCH=default
PATH=${PATH}:~/bin
mkdir ~/neat-var-fslc-yocto
cd ~/neat-var-fslc-yocto
repo init -u https://github.com/neat-tech-repos/neattech_yocto-manifest -b $GIT_BRANCH
CORES=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || sysctl -n hw.ncpu || echo "$NUMBER_OF_PROCESSORS")
repo sync -f -n -j 4 --force-sync && repo sync -l -j $CORES --force-sync
```
# Theory of operation
```
the device usually boots from EMMC, however, if a new SOM is being used, it's partition needs to be modified
in order to support software update (double partition is being used, so if update failes it can automatically rollback)
so only once per new SOM device, we need to boot device from SD card, issue update command from SD card to partition and flash EMMC
with siftware upgrade support, then we can normally boot from EMMC and update images 
```

# Build image

To build image/flash image to SDCard or prepare swupdate image. We can use some commands as below  

## setup PC 
- Install all dependency packages (note: needs to be done only once when downloading the software)

```
$ cd ~/neat-var-fslc-yocto

$ . modular-tools setup
```

## Build image on PC 

```
$ cd ~/neat-var-fslc-yocto

$ . modular-tools build_image
`
on this stage we expect to have an error at some point, this is OK and expected``
```
## Append build layers on PC 

- Append layers, needs to be done only one time, after first build, and then run build image again

```
$ cd ~/neat-var-fslc-yocto

$ . modular-tools append_layers
```

## Build update image 

```
every time after performing once the previuse stages, we need only to run this command in order to create a new update image package

$ cd ~/neat-var-fslc-yocto

$ . modular-tools build_update_file
```
## Check if initial update from SD card was done to a specific SOM
```
connect to SOM using terminal
sudo picocom -b 115200 /dev/ttyUSB0
write : swupdate
see if swupdate exists, if so then SD update was already done and no need to re-perform
```


## Flash image to SD card 

- Flash to SDCard - the SD card flash needs to be done only once per device, this sets different  emmc partitions supporting swupdate 

```
$ cd ~/neat-var-fslc-yocto

$ . modular-tools build_sd_image

$ cd ~/neat-var-fslc-yocto

$ . modular-tools generate_sd_card

* in order to find drive letter enter command "df", and view SD card drive (for exampleif ,[/dev/sdb1   media/neat/BOOT-VAR6UL]  -> drive letter is b (from sd b 1)
```
## Flash image from SD card to EMMC
```
start the device from SD card by changing the boot select switch to start from SD card
connect to SOM using UART terminal
wait for device to upload from SD card
issue the following command :

install_yocto.sh -u

this will install basic image from SD card to EMMC, that image supports two things:

1. dual bank partition of EMMC, to allow software update with rollback support 
2. support for software update functionality

after compleating installation remove SD card, and return switch to EMMC boot select, from now on all that is needed is to update software using 
modular tools update support
```

## Update software easilly using modular tools utility
```
. modular-tools locate_board_ip   -  searches for variscite board name in network, if PC and variscite board are connected to the same network, it will 
print the boards IP address

 . modular-tools update_unit_image_prepare   - it will copy the needed images + setup the board to be ready to update software, notice you needed to specify board IP, you can locate the board IP by using the previuse command (. modular-tools locate_board_ip)
 
 update software by ssh to device address (ssh root@<board IP>)
 and issue command:  ~/update_script.sh
 then reboot the SOM device for update to take effect
```

# Activate react support
```
at first device powerup issue the following command:
npm install -g serve
this will install the needed support for react, this needs to be done only once after flashing the device
```

# Update recips, neattech specific software additional to the OS 
```
notice all recips are taken from: 
https://github.com/neat-tech-repos/neattech_yocto-meta-kama
this is being pulled whle performing repo sync
notice that repo xml file is located under .repo/manifasts/default.xml
and specific git version is specified under:
 <project remote="kamacode" path="sources/meta-kama" name="neattech_yocto-meta-kama" revision="de378558c10950de24ea2d7ed77b70e20ecdf491" >
 
 so in order to update recips do the following:
 1. edit the directory : sources/meta-kama
 2. push to remote : https://github.com/neat-tech-repos/neattech_yocto-meta-kama
 3. update repo file under: .repo/manifasts/default.xml
 4. push to remote : https://github.com/neat-tech-repos/neattech_yocto-manifest

```
# Cancel Autoload for debugging (only within the Linux system)


The script is located at :

neat@neat-System-Product-Name:~/neat-var-fslc-yocto/sources/poky$ 

To open the script file type the following:

nano meta/recipes-extended/cronie/files/start_application.sh

After changing the file save it with ctr+x.

To cancel the autoload delete the "node app.js" from the following line from the start_application.sh:

@reboot root cd /opt/server/ && systemctl stop nodejs-server && systemctl restart run-chromium && node app.js

# Cancel Autoload for debugging (only within the SOM)

additionally, remove the line for starting node JS from this location:

nano /opt/bar.sh 

notice this line:

cd /opt/server/ && node app.js &

change to this :

cd /opt/server/ 


# Update boot images
```
notice there are 3 boot images
first is called "splash image" and is shown first thing on start
second is called "logo image" and it's shown on upper left corner of screen on kernel load
third is called "fb image", and it's shown after the "logo image" untill system sinishing loading

this is how to setup and change each one of them:


splash image:

copy bmp file , 800x480 to : ~neat/neat-var-fslc-yocto/sources/meta-variscite-fslc/recipes-bsp/u-boot/u-boot-splash/splash.bmp
cp <filename.bmp>  ~neat/neat-var-fslc-yocto/sources/meta-variscite-fslc/recipes-bsp/u-boot/u-boot-splash/splash.bmp
example: cp /home/neat/Downloads/splash.bmp ~neat/neat-var-fslc-yocto/sources/meta-variscite-fslc/recipes-bsp/u-boot/u-boot-splash/splash.bmp

logo image:

image should be png, size 80x80
on pc :
apt-get install netpbm
//notice output image name must be named as logo_variscite_clut224.ppm
pngtopnm /path/to/image.png | ppmquant -fs 223 | pnmtoplainpnm > logo_variscite_clut224.ppm
and copy file "logo_variscite_clut224.ppm" under: "linux-imx/drivers/video/logo/logo_variscite_clut224.ppm"
cp <logo_variscite_clut224.ppm> ~neat/neat-var-fslc-yocto/linux-imx/drivers/video/logo/logo_variscite_clut224.ppm



fb image: 

use this script :

#!/bin/sh
#
# SPDX-License-Identifier: GPL-2.0-or-later
#

set -e

imageh=`basename $1 .png`-img.h
name="${2}_IMG"
gdk-pixbuf-csource --macros $1 > $imageh.tmp
sed -e "s/MY_PIXBUF/${name}/g" -e "s/guint8/uint8/g" $imageh.tmp > $imageh && rm $imageh.tmp


for example :

cd /tmp/
nano splash_to_h.sh
now copy the script as described above
chmod 777 splash_to_h.sh
./psplash_to_h.sh psplash_white.png


run as: 
cd /tmp/
./psplash_to_h.sh psplash_white.png

result file would be :  "psplash_white-img.h"

copy as : 

cp psplash_white-img.h ~/neat-var-fslc-yocto/sources/meta-variscite-fslc/recipes-core/psplash/files/psplash-poky-img.h



```

# Prepare and install NRFUTIL

To prepare new firmware and package with iMX environment, we can setup it normally like PC. Here are some steps as below

Step 1: Check python and pip to make sure iMX environment is ready. If not, please install by following command below

```
curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
python3 get-pip.py --user
```

Step 2: Install nrfutil by command as below

```
. modular-tools nrfutil_install
```

**Notice** To make sure nrfutil is ready. Please check with command

```
fron the SOM terminal: 
export LC_ALL=en_US.utf-8
export LANG=en_US.utf-8
/opt/server/nrfutil --help
```

Step 4: Upgrade via serial port.

**Notice** To find serial port of nRF52832, we can use commands. `(It depends on your hardware serial type)`

```
dmesg | grep cdc_acm
or
ls -l /dev | grep ttyACM 
```


To upgrade firmware via serial port. Run command as below 

```
nrfutil -vvv dfu serial -pkg test.zip -p /dev/ttyACM0
notice /dev/ttyACM0 is serial port name connected to the Nordic, name may be different depends on the platform
```
