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
$ repo sync -f -n -j 4 && repo sync -l -j $CORES
```

# Build image

## first time only
cd ~/neat-var-fslc-yocto

. modular-tools setup                  #needs to be done only once when downloading the software

. modular-tools build_image

cd ../

. modular-tools append_layers

## on each build:
. modular-tools build_update_file

cd ../

. modular-tools update_unit_image_prepare

#then ssh to device and issue from device: ~/update_script.sh
