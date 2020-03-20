I needed to connect to a firecracker instance on /dev/pts/2 so this was the [solution](https://stackoverflow.com/questions/20532195/socat-with-a-virtual-tty-link-and-fork-removes-my-pty-link), then just nc to the listening port
```bash
$socat -d -d open:/dev/pts/2,nonblock,echo=0,raw TCP-LISTEN:11313,reuseaddr,fork 
```
Output from socat connecting to firecracker
```bash
[ OK  ] Started Create Volatile Files and Directories.

         Starting Update UTMP about System Boot/Shutdown…

[  OK  ] Started Update UTMP about System Boot/Shutdown.

[  OK  ] Reached target System Initialization.

[  OK  ] Reached target Timers.

[  OK  ] Reached target Paths.

[  OK  ] Listening on RPCbind Server Activation Socket.

[  OK  ] Listening on D-Bus System Message Bus Socket.

[  OK  ] Reached target Sockets.

[  OK  ] Reached target Basic System.

         Starting Permit User Sessions…

         Starting Dump dmesg to /var/log/dmesg…

[  OK  ] Started OpenSSH server daemon.

         Starting OpenSSH server daemon…

         Starting LSB: Bring up/down networking…

[  OK  ] Started D-Bus System Message Bus.

         Starting D-Bus System Message Bus…

         Starting Login Service…

[  OK  ] Started Permit User Sessions.

[  OK  ] Started Dump dmesg to /var/log/dmesg.

[  OK  ] Started Login Service.

[  OK  ] Started Getty on tty1.

         Starting Getty on tty1…

[  OK  ] Started Serial Getty on ttyS0.

         Starting Serial Getty on ttyS0…

[  OK  ] Reached target Login Prompts.

[  OK  ] Started LSB: Bring up/down networking.

[  OK  ] Reached target Network is Online.

[  OK  ] Reached target Multi-User System.

[  OK  ] Reached target Graphical Interface.

         Starting Update UTMP about System Runlevel Changes…

[  OK  ] Started Update UTMP about System Runlevel Changes.
```
