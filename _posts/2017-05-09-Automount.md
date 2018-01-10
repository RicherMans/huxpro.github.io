---
layout: post
title: Automounting and it's issues
---

I work pretty often on a server cluster. To do so I use (like mostly everybody else) ssh. When working with files or a remote environment `sshfs` is a reliable tool to mount remote storage on your computer. On a laptop, the automount functonality on boot might not be optimal, since wifi is not always available.

Automounting `sshfs` on demand is a pretty versatile solution. It only mounts the directory on the first access. To do so adjust your `/etc/fstab` and add another row:

```
user@REMOTE:REMOTEPATH LOCALPATH  fuse.sshfs noauto,x-systemd.automount,_netdev,users,idmap=user,IdentityFile=$HOME/.ssh/id_rsa,uid=1000,gid=1000,allow_other,reconnect,comment=x-gvfs-hide,compression=yes,port=xxx 0 0
```

The options are as follows:

1. **noauto** does tell fstab not to mount this device on boot.
2. **x-systemd.automount** does the magic.
3. **idmap** and **IdentityFile** are used as login credentials. Note that in order to work, the root user on your device needs to connect at least once with the server. Moreover the server needs to be access in a non-password fashion.
4. **compression** allows for less data traffic thus making the connection faster.
5. **x-gvfs-hide** masks the directory in being invisible to gvfs ( if there are any gvfs problems that might help )
6. **port** is the port of the ssh server.
7. **Identitiyfile** is the previously generated ssh id file. First run ``ssh-keygen`` then `ssh-copy-id USER@SERVER` to enable auto login onto the specific server.


## Troubleshooting with GVFS (Xfce)

In my case, that did not prevent automount from being fired `directly` after start leading to errors during mount. The bad guy here is gvfs, specifically `thunar-volman` and `gvfs-trash`. Both of them seem to access the sshfs mounted directory on start. 

Another possible solution is to turn the switch `AutoMount=true` to `AutoMount=false` in the file

```
/usr/share/gvfs/mounts/trash.mount
```