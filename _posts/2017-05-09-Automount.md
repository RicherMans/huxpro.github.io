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

# Another alternative: Autofs

While `sshfs` is powerful indeed, I experienced quite some problems regarding the auto dismount function. Especially when I previously opened any `sshfs` mounted folder in sublime and then disconnected, the remote sshfs will hang. This is pretty frustrating, since even restarts will not work (but a restart of sshfs will do `sudo systemctl restart sshfs`).

Another alternative is [autofs](https://wiki.archlinux.org/index.php/autofs), which provided much more functionality compared to `sshfs`, e.g. ftp, nfs, samba support.

For a simple config, follow the following instructions:

1. Install autofs. In arch its just `sudo pacman -S autofs`
2. [Autofs](https://wiki.archlinux.org/index.php/autofs) generates couple of files in `/etc/autofs/`. At first, open the file `/etc/autofs/auto.master` with e.g., vim `sudo vim /etc/autofs/auto.master`. I personally commented the lines `/net -hosts` and `/misc/ /etc/autofs/auto.misc` since I dont use any misc formats or mounting points.
3. At the end of the opened `auto.master` file, add a line: `/-      /etc/autofs/auto.ssh    uid=1000,gid=1000,--timeout=10,--ghost`. The parameters mean:
    * `/- ` Mount absolute path (given in the `auto.ssh` config). If `/-` is replaced with e.g., `/ssh/`, all mounts in the `auto.ssh` config will be relatively mounted to `/ssh/`. I personally have two mounts which need to be placed in the root dir, thus need to use `/-`.
    * `uid=1000,gid=1000` Mount as root
    * `--timeout` Time until the mount will be dismounted (if a disconnect occured)
    * `--ghost` Keep a "dummy" folder as the mount folder, even if the directory is not mounted. This is very helpful for sublime.
4. Open the file `/etc/autofs/auto.ssh`, where you can configure your ssh mounts. The following syntax is used in this file:
    `/MOUNTPOINT     -fstype=fuse,rw,allow_other,reconnect,no_check_root,compression=yes,port=YOURPORT,IdentityFile=/home/YOURUSER/.ssh/id_rsa :sshfs\#REMOTEUSER@SERVER\:/REMOTEDIR`
    You can add multiple ssh mounts in this file. Note that in order to work, your user needs to have password-less login enabled (`ssh-copy-id`), as well as the necessity that the root user at least logged into the server once ( to obtain the RSA fingerprint )
5. Enable/Start autofs `sudo systemctl start autofs`, `sudo systemctl enable autofs` and enjoy! The mounts will be at the paths specified in `auto.ssh`  