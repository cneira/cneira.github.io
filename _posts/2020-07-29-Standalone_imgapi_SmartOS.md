# Standalone imgapi in SmartOS

[imgapi](https://github.com/joyent/sdc-imgapi/blob/master/docs/index.md) is the SmartOS image management api, that allows you to publish, delete
and import images SmartOS don't have this api installed by default but is
available in Triton, the purpose of this document is to explain how to make it
available in SmartOS. All the information needed to enable the imgapi in SmartOS
was taken from this document : [Setting up imgapi standalone in SmartOS](imgapistandalone.pdf) which was provided
on the [SmartOS mailing list](https://smartos.topicbox.com/groups/smartos-discuss/Tdf42362ada3cb232/smartos-imgapi-standalone-setup).
   
I used SmartOS (build: 20200603T203505Z) to create the imgapi instance.
    
    
The problems I found are the following: 
- Installing sdc-imagcli I needed to add flags --unsafe-perm=true --allow-root to install it to the GZ.   
  
 ```bash
 npm install -g git+https://github.com/joyent/sdc-imgapi-cli.git --unsafe-perm=true --allow-root
  ```
-  script inside the imgapi-standalone image expects /opt/smart-dc instead of /opt/sdc-imgapi just a symlink fixed the issue.

- Trying to create a custom image using as base the minimal 64 image and just
installed postgresql. But I encountered this error:

```bash 
[root@cl-west-05 /opt/images/custom]# imgadm create -c gzip -i -s /opt/smartdc/sdc-imgapi/tools/prepare-image/smartos-prepare-image ${uuid} name=test1 version=1.0 public=true owner=00000000-0000-0000-0000-000000000000
Inheriting from origin image e1a0b1ec-c690-11ea-8e16-d3e64a9e7fe3 (minimal-64 20.2.0)
Manifest:
{
"v": 2,
"uuid": "368e00d2-7b64-4fe5-b996-01a70ef62c91",
"name": "test1",
"version": 1,
"public": true,
"owner": "00000000-0000-0000-0000-000000000000",
"type": "zone-dataset",
"os": "smartos",
"requirements": {
"min_platform": {
"7.0": "20200603T203505Z"
},
"networks": [
{
"name": "net0",
"description": "public"
}
]
},
"origin": "e1a0b1ec-c690-11ea-8e16-d3e64a9e7fe3"
}
Stopping VM b1236456-4b25-6651-9b54-9b22b130fc6e to snapshot it
Snapshotting VM "b1236456-4b25-6651-9b54-9b22b130fc6e" to @imgadm-create-pre-prepare
Preparing VM b1236456-4b25-6651-9b54-9b22b130fc6e (starting it)
Timeout waiting for prepare-image script to signal it started
Rollback VM b1236456-4b25-6651-9b54-9b22b130fc6e to pre-prepare snapshot (cleanup)
Restarting VM b1236456-4b25-6651-9b54-9b22b130fc6e (cleanup)
imgadm create: error (PrepareImageDidNotRun): prepare-image script did not run on VM b1236456-4b25-6651-9b54-9b22b130fc6e boot
```
  
  
The problem was that it seems that the base 64 minimal image has the mdata
services disables, which are needed for custom image creation.
With the help of wiedi in #smartos irc channel we found out the problem. 
**The zone I'm using to create the custom image has mdata services disabled** 
Enabling those then running the script again worked!.
I followed this steps:
  
- First enable the mdata services on the zone that will be used to create the
  custom image
  
```bash 
[root@postgresql-12 ~]# svcadm enable svc:/smartdc/mdata:fetch  
[root@postgresql-12 ~]# svcadm enable svc:/network/nfs/status:default
```
  
- Now from the GZ create the custom image.
  
  
```bash 
[root@cl-west-05 /opt/images/custom]# imgadm create -c gzip -i -s /opt/smartdc/sdc-imgapi/tools/prepare-image/smartos-prepare-image ${uuid} name=test1 version=1.0 public=true owner=00000000-0000-0000-0000-000000000000 
Inheriting from origin image e1a0b1ec-c690-11ea-8e16-d3e64a9e7fe3 (minimal-64 20.2.0)
Manifest:
    {
      "v": 2,
      "uuid": "9249ec95-580d-48ab-8ec8-c4c325b081b0",
      "name": "test1",
      "version": 1,
      "public": true,
      "owner": "00000000-0000-0000-0000-000000000000",
      "type": "zone-dataset",
      "os": "smartos",
      "requirements": {
        "min_platform": {
          "7.0": "20200603T203505Z"
        },
        "networks": [
          {
            "name": "net0",
            "description": "public"
          }
        ]
      },
      "origin": "e1a0b1ec-c690-11ea-8e16-d3e64a9e7fe3"
    }
Stopping VM b1236456-4b25-6651-9b54-9b22b130fc6e to snapshot it
Snapshotting VM "b1236456-4b25-6651-9b54-9b22b130fc6e" to @imgadm-create-pre-prepare
Preparing VM b1236456-4b25-6651-9b54-9b22b130fc6e (starting it)
Prepare script is running
Prepare script succeeded
Prepare script stopped VM b1236456-4b25-6651-9b54-9b22b130fc6e
Snapshotting to "zones/b1236456-4b25-6651-9b54-9b22b130fc6e@final"
Sending image file to "test1-1.zfs"
Saving manifest to "test1-1.imgmanifest"
Rollback VM b1236456-4b25-6651-9b54-9b22b130fc6e to pre-prepare snapshot (cleanup)
Restarting VM b1236456-4b25-6651-9b54-9b22b130fc6e (cleanup)
[root@cl-west-05 /opt/images/custom]# 
```
