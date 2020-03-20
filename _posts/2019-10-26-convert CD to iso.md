I needed to convert some old cds to isos, this worked like a charm

```bash
$ isoinfo -d -i /dev/cdrom | grep -i -E 'block size|volume size' 
 Logical block size is: 2048
 Volume size is: 1439856
```
Then take the blocksize and pass it to dd

```bash
$ dd if=/dev/cdrom of=~/ISOS/marvel_ultimate_alliance.iso bs=2048 count=1439856 status=progress 

2943076352 bytes (2.9 GB, 2.7 GiB) copied, 224 s, 13.1 MB/s 

1439856+0 records in

1439856+0 records out

2948825088 bytes (2.9 GB, 2.7 GiB) copied, 224.303 s, 13.1 MB/s
```
