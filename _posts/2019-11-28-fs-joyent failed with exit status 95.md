I encountered this error in my Triton instance

![](https://carlosneirablog.files.wordpress.com/2019/11/screenshot-from-2019-11-27-18.25.35.png)
  
And the community at irc.freenode.org #smartos helped me solve it.  
The error was related to running out of space and my dump device was not big enough,  
so here are the steps to solve it:  

* Add a new disk to your Triton instance and boot it using a SmartOS usb and select rescue mode then execute:
```bash
# zpool import zones
# zfs destroy zones/dump
# zfs create -V <your ram amount>G zones/dump
# dumpadm -y -d /dev/zvol/dsk/zones/dump 
# reboot
```
Relevant information on how to fix this, is in this irc conversation :  

[https://echelog.com/logs/browse/smartos/1503957600](https://echelog.com/logs/browse/smartos/1503957600)
