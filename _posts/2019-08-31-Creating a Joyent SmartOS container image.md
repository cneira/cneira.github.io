Most of the time I’m using a lx-branded zones to host games like quake live or neverwinter nights, so I thought it would 
be nice to share them with other users or save up some work setting them up. 
I found this gist where an image is created manually: 
https://gist.github.com/glinton/9153193
https://wiki.smartos.org/managing-images/

It’s really straight forward, but I really want to be able to share the images.
I found out that SmartOS users are already have done that: 
http://blog.smartcore.net.au/dsapid-build-smartos-dataset-server/
https://blog.jasper.la/setting-up-a-smartos-image-server.html

Following these guides and importing image a0e719d6-4e21-11e4-92eb-2bf6399552e7 I could start serving datasets.
```bash
[root@frodo /opt/templates]# imgadm sources -a http://datasets.at/
[root@frodo /opt/templates]# imgadm update
[root@frodo /opt/templates]# cat dsapi.json
{
  "brand": "joyent",
  "image_uuid": "a0e719d6-4e21-11e4-92eb-2bf6399552e7",
  "autoboot": true,
  "alias": "dsapid-server",
  "hostname": "datasets.byteswizards.com",
  "resolvers": [
    "8.8.8.8",
    "8.8.4.4"
  ],
  "max_physical_memory": 1024,
  "max_swap": 1024,
  "tmpfs": 1024,
  "quota": 120,
  "nics": [
    {
      "nic_tag": "igb2",
      "ip": "192.168.1.114",
      "netmask": "255.255.255.0",
      "gateway": "192.168.1.1",
      "primary": true
    }
  ]
}
[root@frodo /opt/templates]# vmadm create -f dsapi.json
Successfully created VM 7333223b-daea-4703-c068-97509ace52eb
[root@frodo /opt/templates]# zlogin 7333223b-daea-4703-c068-97509ace52eb
[Connected to zone '7333223b-daea-4703-c068-97509ace52eb' pts/4]
     _                 _     _
  __| |___  __ _ _ __ (_) __| |
 / _` / __|/ _` | '_ | |/ _` |
| (_| __  (_| | |_) | | (_| | SmartMachine (dsapid 0.6.7)
 __,_|___/__,_| .__/|_|__,_| https://github.com/datasets-at/mi-dsapid
                |_|
[root@datasets ~]# vi /data/
config.json  files/       users.json
[root@datasets ~]# vi /data/config.json
[root@datasets ~]# svcadm  dsapid
[root@datasets /data]# cat config.json
{
  "log_level": "info",
  "base_url": "http://datasets.byteswizards.com/",
  "mount_ui": "/opt/dsapid/ui",
  "datadir": "/data/files",
  "users": "/data/users.json",
  "hostname": "datasets.byteswizards.com",
  "listen": {
    "http": {
      "address": "0.0.0.0:80",
      "ssl": false
    }
  },
  "sync": [
    {
      "name": "official joyent dsapi",
      "active": true,
      "type": "dsapi",
      "provider": "joyent",
      "source": "https://datasets.joyent.com/datasets",
      "delay": "12h"
    },
    {
      "name": "official joyent imgapi",
      "active": true,
      "type": "imgapi",
      "provider": "joyent",
      "source": "https://images.joyent.com/images",
      "delay": "12h"
    },
    {
      "name": "datasets.at",
      "active": true,
      "type": "dsapi",
      "provider": "community",
      "source": "http://datasets.at/api/datasets",
      "delay": "12h"
    }
  ]
}
```
# How to publish images  

Here is how to publish images from this blog :
https://blog.jasper.la/setting-up-a-smartos-image-server.html
One downside of dsapid is that imgadm publish does not work and you’ll have to use this magic curl command instead:  
```bash
curl -v -u admintoken: http://192.168.178.172:8000/api/upload -F manifest=@manifest.json  -Ffile=@python-15.4.0.0.zfs.gz  
```
I’m pushing an example image that’s created with sm-prepare-image and a manifest.json like:  
```json
{
  "uuid": "48ea274b-0668-64f6-fde0-e3ad3bc552eb",
  "name": "python",
  "version": "15.4.0.0",
  "description": "Minimal image with Python installed (e.g. for use with Ansible)",
  "os": "smartos",
  "type": "zone-dataset",
  "platform_type": "smartos",
  "creator_name": "jasperla",
  "creator_uuid": "3433bb9e-0144-40ad-aeb9-ed51b88c3d0f",
  "vendor_uuid": "3433bb9e-0144-40ad-aeb9-ed51b88c3d0f",
  "urn": "jasperla:jasperla:python:15.4.0.0",
  "created_at": "2016-03-06T11:40:20.000Z",
  "updated_at": "2016-03-06T11:40:20.000Z",
  "published_at": "2016-03-06T11:40:20.000Z",
  "files": [
    {
      "path": "python-15.4.0.0.zfs.gz",
      "sha1": "7595eac569ef67fab8f286f6d2f88a3a0191b11c",
      "size": 168310282,
      "compression": "gzip"
    }
  ]
}
```
