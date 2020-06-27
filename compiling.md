# Wireguard for UDM/UDM pro


## Building the binaries
**Author:** Carlos Talbot (@tusc69 on ubnt forums)

In order to cross compile the userland tools for wireguard and the various utilities such as htop, iftop, iperf3, you will need to download and install buildroot. For this example we are using an Unbuntu 20.04 server.

You need to prepare the server for the necessary compilers. The command below will install the essentials:

```
# sudo apt-get install build-essential ncurses-base ncurses-bin libncurses5-dev dialog unzip bc python curl
```

Go ahead and download and extract the most recent version of buildroot (at this time that’s 2020.05). 

```
# curl  https://buildroot.org/downloads/buildroot-2020.05.tar.gz
# tar -xvzf buildroot-2020.05.tar.gz
# cd buildroot-2020.05
```
Once you change into that directory go ahead and download the .config and kernel.config file from the gitbhub page into this directory.

```
# curl -LJo .config  https://github.com/tusc/wireguard/blob/master/buildroot.config?raw=true
# curl -LJo kernel.config  https://github.com/tusc/wireguard/blob/master/buildroot-kernel.config?raw=true
```

Once this is all in place you are ready to begin the compile. Type the following:

```
make
```

At this point you might want to step away and grab a cup of coffee or tea as this will take a while.
The binaries will be located in the following folders:<br/>
./output/target/bin<br/>
./output/target/sbin<br/>
./output/target/usr/bin<br/>
./output/target/usr/sbin

For example, htop should be in here:
```
# ls -l output/target/usr/bin/htop
-rwxr-xr-x 1 build build 311592 Jun 27 16:27 output/target/usr/bin/htop
#
```

At this point you can copy the static binaries over to your UDM/UDM pro with scp and ensure they are executable with this command:

```
# chmod +x htop
```

You can compile additional tools if you wish. Just run “make menuconfig” and go into the “Target packages” from the top menu.

```
# make menuconfig
```
**note: these binaries are built with the musl library in a static build so some packages won’t be available to compile**

Since we cannot use the kernel module for wireguard we will have to compile a userland equivalent. There are two available, wireguard-go and boringtun. I went with the latter since it is suppose to be faster (go vs rust).
In order to cross compile the boringtun binary you will need a couple of tools:
