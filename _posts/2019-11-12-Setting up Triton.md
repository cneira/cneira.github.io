I have been using [Triton](https://github.com/joyent/triton) lately  using this awesome guide 
[SmartOS Coal on Linux](https://www.daveeddy.com/2019/02/12/smartos-coal-on-linux-kvm-with-virt-manager/)

The only gotchas I found are the following:

* https://github.com/joyent/node-triton/issues/71 
Need to add insecure to your profile on the triton cli https://docs.joyent.com/public-cloud/api/triton-cli to be able to provision
After installation execute : 
```bash
$ sdcadm post-setup cloudapi
$ sdcadm post-setup dev-headnode-prov
```
Per the docs to be able to provision on the head node (https://github.com/joyent/triton/blob/master/docs/developer-guide/coal-setup.md)
the json script on the install points to /usr/bin/node and in my triton install node exists in /usr/node/bin/node 
changing that script made sdcadm healthcheck work.
Execute 
```bash
sdcadm post-setup common-external-nics 
```
[To be able to import images](https://smartdatacenter.topicbox.com/groups/sdc-discuss/T2e043601a627e58d-Me79b91c063f7d6c215c87dd9/image-search-and-download)
* Create a custom package to provision vms https://docs.joyent.com/private-cloud/install/advanced-configuration

* docker setup : 
```bash
sdcadm post-setup docker
sdcadm experimental update dockerlogger
sdcadm post-setup dev-sample-data
```
* Install triton-docker cli and triton profile docker-setup <profile>
triton-docker –tls ps works
* Enable cns so triton-cns docker compose start working (https://docs.joyent.com/private-cloud/install/cns)
* Set dns https://docs.joyent.com/public-cloud/network/cns/faq
  sdc-login cns , get the zone uuid, then get the zone ip address and point your resolvers to that ip to resolve the containers names.
* Disable booting from pxe : sdc-usbkey set-variable ipxe false
* Add an sdd physical drive to the vm add storage an in custom the path from /dev/disk/by-id and virtio driver https://ckirbach.wordpress.com/2017/07/25/how-to-add-a-physical-device-or-physical-partition-as-virtual-hard-disk-under-virt-manager/
