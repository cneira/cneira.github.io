Trying to create a tap device failed https://github.com/rootless-containers/rootlesskit/issues/69
I just needed to install iproute2
```bash
#apk add iproute2
```
And be sure to have the tun driver loaded
```bash
# modprobe tun 
```
