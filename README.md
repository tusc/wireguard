# Wireguard for UDM/UDM pro

## Distributed under MIT license

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Project Notes
**Author:** Carlos Talbot (@tusc69 on ubnt forums)

The tar file in this repository is a collection of binaries that can be loaded onto an UDM to run wireguard in userland mode. I plan to include instructions on how to compile the software shortly.

ssh into the UDM and type the following command from the /root folder to download the tar file:

**# curl -LJo udm-wireguard.tar.Z  https://github.com/tusc/wireguard/blob/master/udm-wireguard.tar.Z?raw=true**

From this directory type the following, it will extract the files under the /mnt/data path:

**# tar -C / -xvf udm-wireguard.tar.Z**

Once the extraction is complete, cd into /mnt/data and run the script setup_bin.sh. This will setup the symbolic links for the various binaries to the /bin path as well as create a symlink for the /etc/wireguard folder.

There's a sample wg0.conf in /etc/wireguard you can use to create your own, provided you update the public and private keys. There are various tutorials out there for setting up a client/server config for wireguard (e.g. https://www.stavros.io/posts/how-to-configure-wireguard/ )

In order to start the wireguard tunnel you need to run this command from the cli:

**# WG_QUICK_USERSPACE_IMPLEMENTATION=boringtun wg-quick up wg0**

you should see output similar to the following:

```
[#] ip link add wg0 type wireguard
RTNETLINK answers: Operation not supported
[!] Missing WireGuard kernel module. Falling back to slow userspace implementation.
[#] boringtun wg0 --disable-drop-privileges=1
BoringTun started successfully
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.253.0.3/32 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] ip -4 route add 10.253.0.0/24 dev wg0
```

You can also execute the wg binary for status on the tunnel:

**# wg**

interface: wg0
  public key: ************************
  private key: (hidden)
  listening port: 43724

peer: *********************************
  preshared key: (hidden)
  endpoint: 192.168.1.106:51820
  allowed ips: 10.253.0.0/24
  
 Finally, in order to shutdown the tunnel you'll need to run this command:
 
**# WG_QUICK_USERSPACE_IMPLEMENTATION=boringtun wg-quick down wg0**

You'll find some other useful utils in the /mnt/data/bin path such as bash, htop, iftop and iperf3.

