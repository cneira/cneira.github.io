I/O devices are treated as special files called device files,  
major numbers are used in device files to identify the type of device and the minor number identifies a specific device. 
For example :  
```
cneira@ebpf-metal:~/bpf-next$ ls -l /dev/vda*
brw-rw---- 1 root disk 254, 0 Sep 3 02:38 /dev/vda
brw-rw---- 1 root disk 254, 1 Sep 3 02:38 /dev/vda1
brw-rw---- 1 root disk 254, 2 Sep 3 02:38 /dev/vda2
brw-rw---- 1 root disk 254, 5 Sep 3 02:38 /dev/vda5
```

254 is the major number and 0, 1, 2, 5 are the minor numbers from this device, it just means that 254  
is the device type and 0, 1, 2 ,5 minor numbers are used to differentiate between them.   
You could have several devices that share the same major numbers if they are the same type of device,  
but they will have different minor numbers to identify each one. For example a group of disks managed by  
the same disk controller have the same major number but different minor numbers.  
The inode of a device file does not have pointers to blocks of data on the disk, instead it must include  
an identifier of the hardware device corresponding to the character or block device file.  
This identifier corresponds to the major and minor numbers of the device file.  
Since Linux 2.6 the major number is encoded in 12 bits and the minor number is encoded in 20 bits,  
both numbers are kept in a single 32-bit variable of type dev_t.  
MAJOR and MINOR macros extracts the major and minor number from a dev_t value.
The major and minor numbers are stored in the i_rdev field of the inode object and the type of device file is stored in the i_mode field.

# References:
Understanding the Linux Kernel 3rd edition.
