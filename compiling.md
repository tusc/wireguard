# Wireguard for UDM/UDM pro


## Building the binaries
**Author:** Carlos Talbot (@tusc69 on ubnt forums)

In order to cross compile the userland tools for wireguard and the various utilities such as htop, iftop, iperf3, you will need to download and install buildroot. For this example we are using an Unbuntu 20.04 server.

You need about ~ 5GB of disk space for buildroot after everything is compiled. You also need to install addtional packages for preping the compiler environment. The command below will install the essentials:

```
$ sudo apt-get install build-essential ncurses-base ncurses-bin libncurses5-dev dialog unzip bc python curl git
```

Go ahead and download and extract the most recent version of buildroot (at this time that’s 2020.05). 

```
$ curl  https://buildroot.org/downloads/buildroot-2020.05.tar.gz
$ tar -xvzf buildroot-2020.05.tar.gz
$ cd buildroot-2020.05
```
Once you change into that directory go ahead and download the .config and kernel.config file from the github page into this directory.

```
$ curl -LJo .config  https://github.com/tusc/wireguard/blob/master/buildroot.config?raw=true
$ curl -LJo kernel.config  https://github.com/tusc/wireguard/blob/master/buildroot-kernel.config?raw=true
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
$ ls -l output/target/usr/bin/htop
-rwxr-xr-x 1 ctalbot ctalbot 311592 Jun 27 16:27 output/target/usr/bin/htop
$
```

At this point you can copy the static binaries over to your UDM/UDM pro with scp and ensure they are executable with this command (run from UDM):

```
# chmod +x htop
```

You can compile additional tools if you wish. Just run “make menuconfig” and go into the “Target packages” from the top menu.

```
$ make menuconfig
```
**note: these binaries are built with the musl library in a static build so some packages won’t be available to compile**

The next section is for compiling the binary for establishing the wireguard tunnel.

## Compiling Boringtun
Since we cannot use the kernel module for wireguard we will have to compile a userland equivalent. There are two available, wireguard-go and boringtun. I went with the latter since it is supposed to be faster (go vs rust).
In order to cross compile the boringtun binary you will need a couple of tools:

rustup & cargo (rust compiler and package manager)<br/>
cross (cross compiling of rust crates)<br/>
docker (used to create the cross compile environment)<br/>
boringtun sources

These tools require about ~ 2 GB of space after compilation.

In order to install rustup and cargo you run the installer from your Unbutu server linux prompt:

```
$ curl https://sh.rustup.rs -sSf | sh
```
Accept the defaults, it will update you .profile with paths to the rust compiler under the path $HOME/.cargo/bin. You'll have to log out and log back in in order to have the $PATH environment variable updated.

Install cross
```
$ cargo install cross
```

Install docker for the cross compile rust environment
```
$ sudo apt-get install docker.io
```
Make sure your account is part of the docker group in order to start/stop conatiners. You'll need to log out and back in for the group add to take effect. Finally, start up docker

```
$ sudo usermod -a -G docker {YOUR LOGIN ID}
$ logout
$ sudo systemctl start docker
```

Download the source code for boringtun and change into that directory

```
$ git clone https://github.com/cloudflare/boringtun
$ cd boringtun
```

At this point we are ready to cross compile the boringtun binary

```
$ cross build --bin boringtun --release --target aarch64-unknown-linux-musl
```

This will download the docker image for the cross compile conatiner and will create a static ARM64 binary using the musl library.
When it's finally done your new binary will be in ./target/aarch64-unknown-linux-gnu/release/boringtun !

```
$ ls -l ./target/aarch64-unknown-linux-gnu/release/boringtun
-rwxr-xr-x 2 ctalbot ctalbot 5026952 Jun 28 12:12 ./target/aarch64-unknown-linux-musl/release/boringtun
$
```
Go ahead and copy the following binaries to your UDM/UDM pro and place them in a directory that'll persist after reboots (e.g. /mnt/data/bin)

wg (found in buildroot-2020.05/output/target/usr/bin) <br/>
wg-quick (use this [patched](https://github.com/tusc/wireguard/blob/master/wg-quick) version)<br/>
boringtun (found in boringtun/target/aarch64-unknown-linux-musl/release/<br/>

The patched wg-quick includes an extra parameter to the boringtun cmd to disable drop permissions as well as commenting out dns updates since UbioOS does not include resolvconf.

You'll also need to create a symlink for /dev/fd. This is needed as this is used in the wg-quick script.
```
# ln -s /proc/self/fd /dev/fd &>/dev/null
```
The last step is to create the  /etc/wireguard folder and file called wg0.conf. You can follow the [README](https://github.com/tusc/wireguard/blob/master/README.md) file for the rest of the wireguard configuration.

All of the steps above are done for you if you download the tar file and run the setup_bin.sh so you might want to review it for reference.

At this point you are ready to bring up the tunnel. This is done with the following command:
```
# WG_QUICK_USERSPACE_IMPLEMENTATION=boringtun wg-quick up wg0
```
You can also execute the wg binary for status on the tunnel:
```
# wg

interface: wg0
  public key: ************************
  private key: (hidden)
  listening port: 43724

peer: *********************************
  preshared key: (hidden)
  endpoint: 192.168.1.106:51820
  allowed ips: 10.253.0.0/24
```
Finally, in order to shutdown the tunnel you'll need to run this command:
```
# WG_QUICK_USERSPACE_IMPLEMENTATION=boringtun wg-quick down wg0
```
