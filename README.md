
# Notes and Tips on Building Linux From Scratch

###Table of Contents
[TOC]

### __General Tips__
1. Don't panic, although the code for Linux Kernel as well as all the utilities were written by wizards, they were kind enough to provide us plebs with easy ways to install them.
2. If you think you messed something up while compiling delete the extracted tarball and re-extract and try to rebuild it, it will probably work just fine.
3. Have a lot of patience, building some packages can take up to hours, so ideally have popcorn to enjoy the scenery that is coming up on your terminal and cheer whenever you see recognizable words like size of `void *`, `errno.h`, but you may quickly get bored or you can do any other work.
4. It is extremely frustrating when packages which take hours to build fail after half an hour, don't loose hope, figure out what the error is and do it again more carefully.
5. When a package build fails due to some issue in previous package, you need to rebuild all the packages from that point on, because usually they will also be probably be broken.

### __1. Setting up the Host__

1.  Check the Host system toolings satisfy the version requirements, by running the version-check.sh
	* If `/bin/sh`  doesn't point to `/bin/bash` run                        
              

		    sudo rm /bin/sh                                         
		    sudo ln -s /bin/bash /bin/sh                         
    you can restore the links back  using                                  
                              
    	    sudo rm /bin/sh
            sudo ln -s /bin/dash /bin/sh

   (Apparently Ubuntu uses `dash` as the default shell because it was faster for running boot scripts, they also [attribute a large factor of their boot speed improvement to this](https://wiki.ubuntu.com/DashAsBinSh) )
   * Updating `grep` 
	   * Ubuntu 16.04 still comes with the older version of `grep` which doesn't satisfy the tooling requirement, so download `grep` and install it
	   * A Common trick for extracting any kind of file is to let `tar` figure out and extract the file contents, so run `tar xf <filename>` to extract any kind of file. https://askubuntu.com/questions/92328/how-do-i-uncompress-a-tarball-that-uses-xz
	* Install `gawk` (It's present in the repositories)

2. Run `library-check.sh`, either all 3 files should be present or all of them should be absent
3. **Partitioning**
	* If you have unparitioned space you can directly create a new partition
	* If you don't download gparted Live CD, Load it during boot up and shrink up a partition to accommodate LFS partition
	* Come back to Linux, use [`fdisk` to create a new primary partition](https://www.howtogeek.com/106873/how-to-use-fdisk-to-manage-partitions-on-linux/) (I think you can do this using gparted itself)
	* Format the partition using `mkfs` command
4.  Mount the partition to appropriate directory after setting the `LFS` variable, by using the `mount` command (Make sure to mount it as an `ext4` filetype) 
	* Some things about exporting LFS
		* write `export LFS=/mnt/lfs` in both  `.bashrc` as well as `.bash_profile` (similarly in `/root` directory). The Reasoning behind this is that when bash is invoked as an interactive login shell bash read `.bash_profile` , but when it is invoked as a non-login shell it just reads `.bashrc` ([Or something like that..](http://stefaanlippens.net/bashrc_and_others/) and also [this](https://stackoverflow.com/questions/415403/whats-the-difference-between-bashrc-bash-profile-and-environment) seems to explain it in a much more broader scope)

### __2. Downloading  Packages and Patches__

1. Download all the necessary packages and patches and place them in `$LFS/sources` directory, using the wget list provided
2. Check the checksums
	* A Cool feature is bash is `pushd <filename>` and `popd` to quickly move between directories as noted [here](https://www.eriwen.com/bash/pushd-and-popd/), probably nifty when moving a lot between different 

### __3. Final Preaparations__
1. Make `$LFS/tools` directory and symlink `/tool` to it
2. Create a new user called lfs who belongs to a new group lfs
3. Give him ownership of `$LFS/source` and `$LFS/tools`
4. Login as him and set up his environment

### __4. Building the Toolchain__
1. Change to `$LFS/source` directory
2. Extract the contents of package to be built, using the `tar` command
3. Change to the directory that was created when the package was extracted
4. Build the package
5. Change back to `$LFS/sources` directory
6. Delete the extracted package directory (Honestly don't slack with this, it's gonna cost you a lot of pain if you try to build `gcc` and `libstdc++` from the same extracted package)
7.  Rinse and repeat for all the packages
                       
                Package        Time-Taken (Approximately in Mins)
        ----------------------------------------------------------
		 Binutils                5
	     GCC                     35
		 Glibc                   25
		 Libstdc++               2
		 Binutils (Pass 2)       6
		 GCC (Pass 2)            50
		 
		The rest of them all build under 5 minutes, taking a maximum time of under 2 hours

* Interesting Stuff to Notice along the journey                    
		* `ncurses` test suit is pretty fun to play around with
	   * Apparently `bash` has it's own malloc function, which is known to cause seg faults, so you need to compile it with the option saying you need it to use Glibc `malloc` function
	   * The way `bison` test checks for all the weird output names in the tests
	   * In General after installing the package, look around the directory especially `src/` directories, they contain the source code which even though i don't completely understand are pretty fun to look around
