# Network

## Determine interface names
```shell
root #ifconfig

eth0      Link encap:Ethernet  HWaddr 00:50:BA:8F:61:7A
          inet addr:192.168.0.2  Bcast:192.168.0.255  Mask:255.255.255.0
          inet6 addr: fe80::50:ba8f:617a/10 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1498792 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1284980 errors:0 dropped:0 overruns:0 carrier:0
          collisions:1984 txqueuelen:100
          RX bytes:485691215 (463.1 Mb)  TX bytes:123951388 (118.2 Mb)
          Interrupt:11 Base address:0xe800
```
## Configure the proxies
```shell
#http_proxy
root #export http_proxy="http://proxy.gentoo.org:8080"
#ftp_proxy
root #export ftp_proxy="ftp://proxy.gentoo.org:8080"
#rsync_proxy
root #export RSYNC_PROXY="proxy.gentoo.org:8080"
```

## Netowork automatic config Tool
```shell
root #net-setup eth0
```
## DHCP
```shell
root #dhcpcd eth0
```
## Preparing the wireless access
```shell
root #iw dev wlp9s0 info
Interface wlp9s0
  ifindex 3
  wdev 0x1
  addr 00:00:00:00:00:00
  type managed
  wiphy 0
  channel 11 (2462 MHz), width: 20 MHz (no HT), center1: 2462 MHz
  txpower 30.00 dBm
```
```shell
root #iw dev wlp9s0 link
Not connected.
```
```shell
root #ip link set dev wlp9s0 up
```
!!! note ""
    If the wireless network is set up with WPA or WPA2, then wpa_supplicant needs to be used. For more information on configuring wireless networking in Gentoo Linux, please read the Wireless networking chapter in the Gentoo Handbook.
