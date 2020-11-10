# Install repo 

To get the BSP you need to have `repo` installed and use it as:

Install the `repo` utility:

```
$ mkdir ~/bin
$ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
```

# Download BSP source for image

After installed `repo` tool, we can download BSP source to use some commands as below:

```
$ git config --global credential.helper 'cache --timeout=3600'
$ GIT_BRANCH=default
$ PATH=${PATH}:~/bin
$ mkdir ~/neat-var-fslc-yocto
$ cd ~/neat-var-fslc-yocto
$ repo init -u https://github.com/deadpoolcode1/neattech_yocto-manifest -b $GIT_BRANCH
$ CORES=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || sysctl -n hw.ncpu || echo "$NUMBER_OF_PROCESSORS")
$ repo sync -f -n -j 4 --force-sync && repo sync -l -j $CORES --force-sync
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

- Install all dependency packages (note: needs to be done only once when downloading the software)

```
$ cd ~/neat-var-fslc-yocto

$ . modular-tools setup
```

- Build image 

```
$ cd ~/neat-var-fslc-yocto

$ . modular-tools build_image
`
on this stage we expect to have an error at some point, this is OK and expected``

- Append layers, needs to be done only one time, after first build, and then run build image again

```
$ cd ~/neat-var-fslc-yocto

$ . modular-tools append_layers
```

- Build update image 

```
$ cd ~/neat-var-fslc-yocto

$ . modular-tools build_update_file
```

- Flash to SDCard - the SD card flash needs to be done only once per device, this sets different  emmc partitions supporting swupdate 

```
$ cd ~/neat-var-fslc-yocto

$ . modular-tools build_sd_image

$ cd ~/neat-var-fslc-yocto

$ . modular-tools generate_sd_card

* in order to find drive letter enter command "df", and view SD card drive (for exampleif ,[/dev/sdb1   media/neat/BOOT-VAR6UL]  -> drive letter is b (from sd b 1)
```
