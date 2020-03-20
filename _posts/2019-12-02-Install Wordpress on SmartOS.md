
Installing Wordpress on Triton is pretty straight forward, this guide still holds but we will use base-64-lts@18.4.0 image.
```bash
$ triton create instance -w --name blog base-64-lts@18.4.0 sample-1G 
$ triton ssh blog
$ pkgin in wordpress
$ pkgin in php71-json-7.1.26.tgz  
```
This will install all packages needed for wordpress, but we will to manually do these steps from this [guide](http://www.machine-unix.com/how-to-install-your-wordpress-blog-on-your-joyent-smartmachine-smartos/)
If this error is found : 
```quote
"Apache is running a threaded MPM, but your PHP Module is not compiled to be threadsafe. You need to recompile PHP."
```
Replace 
```bash
LoadModule mpm_event_module modules/mod_mpm_event.so
```
with 
```bash
 LoadModule mpm_prefork_module modules/mod_mpm_prefork.so 
```
then svcadm restart apache 

## References
[Basic smf commands](https://docs.joyent.com/public-cloud/instances/infrastructure/images/smartos/managing-smartos/using-smf/basic-smf-commands)
