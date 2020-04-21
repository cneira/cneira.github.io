I was trying to update my jekyll blog and host it on Omniosce so I installed ruby from pkgsrc and tried :

```bash
$ gem install github-pages
```
But the following error messae  appeared:

```
you must install development tools first

```

The fix was to install cmake and gcc from pkgsrc :

```bash
$ pkgin in cmake gcc9
```
