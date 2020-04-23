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
```

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

- Flash to SDCard

```
$ cd ~/neat-var-fslc-yocto
$ . modular-tools generate_sd_card
```
