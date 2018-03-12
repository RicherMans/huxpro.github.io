---
layout: post
title: Linux powersaving features for XPS13 (9360)
---

This post is about how to enable powersaving for an XPS13 (9360) and my current configuration. If it may help you, I'm happy. I currently (05.03.2017) are quite happy with the device, even though I expected the battery to last much longer than it does currently. On a usual charge I get ~ 8 Hours worth of battery. Since I am a developer I mainly use the command-line and sublime as text editor as my tools plus heavy chrome/chromium usage.
While kaby lake is praised to be more power efficient than skylake, I personally struggle to "outperform" the new macbook with touchbar (colleagues have a ton of them). I am convinced that is mainly due to the unavailability of drivers for the kaby lake architecture ( especially the GPU ). 
However as time passes, I am convinced that as soon new drivers arrive on Linux, it's performance could be boosted up to ten hours.


# TLP

The most common way to save some battery life is [tlp](http://linrunner.de/en/tlp/tlp.html), which provides some very detailed and advanced battery saving options. 

After installing TLP (on a ubuntu based machine) with:

```bash
sudo apt install tlp
```
One can then proceed to look into the configs of TLP:

```bash
sudo vim /etc/default/tlp
```

In my case I use the 4.9.9 kernel, which supports the kaby lake architecture, thus enabling p-state drivers from intel. These drivers only use two states (powersave, performance). Apparently, they are more reliable and effective than the old ACPI drivers. Major advantages are that these drivers undervolt the CPU, whenever possible, making older PHC drivers obsolete. 

Moreover, if the intel p-state drivers are not loaded at startup, but rather the old ACPI onemand governor is active (check by running `sudo tlp-stat -p`), one can remove the ondemand init script from the system by running: 

```bash
 sudo update-rc.d -f ondemand remove 
 ```

Most default features of TLP are alreayd good enough to achieve reasonable battery gains. 

# Wifi Power management

Sometimes it is wise to discontinue the automatic power management (e.g. high latency or slow downloads). If you are using the default Networkmanager from Ubuntu/Debian, then consult the file ``/etc/NetworkManager/conf.d/default-wifi-powersave-on.conf``. The line ``wifi.powersave = 3`` should be changed to ``2`` (``2`` = Off, ``3`` = On).

# NVME (Obsolute with Kernel 4.11+ )

As already pointed out by the [Archwiki](https://wiki.archlinux.org/index.php/Dell_XPS_13_(9360)) entry for this machine, the NVME SSD driver has a powersaving feature, which is not enabled by any current mainline kernel. In order to enable it, one must recompile the given kernel
E.g. in order to compile kernel 4.9.9, follow the following:

```bash
VERSION="4.9.9"
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-$VERSION.tar.gz
tar xvfz linux-$VERSION.tar.gz
git clone https://github.com/damige/linux-nvme
cd linux-$VERSION
# copy current config
cp /boot/config-$(uname -r) .config
# Patching Kernel
patch -p1 < ../linux-nvme/src/$VERSION/APST.patch
patch -p1 <  ../linux-nvme/src/$VERSION/pm_qos1.patch
patch -p1 < ../linux-nvme/src/$VERSION/pm_qos2.patch
patch -p1 < ../linux-nvme/src/$VERSION/pm_qos3.patch 
patch -p1 < ../linux-nvme/src/$VERSION/nvme.patch
# Prepare Kernel build
make-kpkg clean
# Build
fakeroot make-kpkg --initrd --revision=1.0.NVME kernel_image kernel_headers
```

Wait for some minutes (Took me about 40), then two .deb packages will appear in ../*.deb 

In order to install just run ``sudo dpkg -i ../linux-[ih]*``.

These patches are already enqueued into kernel 4.11 and should come out of the box. Currently (21.04.2017) the 4.11rc7 kernel can be recommended.

This patch only decreases the powerconsumption during idle states, which usually is triggered when the harddrive was not used for some seconds. With wifi on and 25% brightness, disabled bluetooth and no open application, my XPS13 uses ~ 3.5-4 Watts. Moreover during a flight with disabled wifi and only light usage of text editors and pdfviewers, this consumption can be confirmed ( with 10% brightness ). This is of course not a representative scenario. 


# Chrome

Another big aspect of powersaving is the "always on" browser. Chrome savings can be pretty drastic. The main power-save which can be done is from enabling hardware acceleration. **Unfortunately** the current Linux distributions do not support full hardware decoding ( AFAIK ). Nevertheless, it is beneficial to enable hardware features in chrome, so that the CPU is not exhausted.

> In Chrome, open chrome://flags
`

1. **Override software rendering list** - enable
2. **GPU rasterization** - enable 
3. **GPU rasterization MSAA sample count** – 2
4. **Number of raster threads** (Machine thread number) - 4 
5. **Fast tab/window close** – Enabled
6. Add `-–enable-native-gpu-memory-buffers` to chrome exec shortcut at `/usr/share/applications/google-chrome.desktop`
7. **Display list 2D canvas** – enabled


> Open chrome://gpu and check if all features are hardware accelerated:

```
Canvas: Hardware accelerated
Flash: Hardware accelerated
Flash Stage3D: Hardware accelerated
Flash Stage3D Baseline profile: Hardware accelerated
Compositing: Hardware accelerated
Multiple Raster Threads: Force enabled
Native GpuMemoryBuffers: Hardware accelerated
Rasterization: Hardware accelerated
Video Decode: Hardware accelerated
Video Encode: Hardware accelerated
VPx Video Decode: Hardware accelerated
WebGL: Hardware accelerated
WebGL2: Hardware accelerated
```

What did somehow extend my battery life beyond expectations was the movement of the chrome cache to a memory mapped directory (e.g. tmpfs, on arch at `/tmp`)

First check where your tmpfs directory is mounted to. Run `df -h` and check for the following line:

`tmpfs                  3.9G  226M  3.7G   6% /tmp`


Doing so is pretty easy, just specify another parameter when starting up chrome, just adjust your `/usr/share/applications/chrome` or (preferred) create a new file in your `$HOME/.config/` folder named `chromium-flags.conf`, or `chromium-dev-flags.conf` or `chrome-flags.conf` depending on your browser. Then just enter the flag `--cache-dir` into the file e.g.
```bash
echo "--disk-cache-dir=/tmp/" >> $HOME/.config/chromium-dev-flags.conf
```

Another option on linux is to use a temporal profile manager, such as [psd](https://aur.archlinux.org/packages/profile-sync-daemon/). This application can be in addition used to store the current profile in the cache, speeding up chrome browsing.


                                     
# Intel Graphics

As most current Laptops, Intel HD Graphics are onboard and no dedicated graphic cards are installed. Most new features of the HD620 are supported with kernel 4.9 +. However to save battery, I use the following kernel parameters:

```
grep GRUB_CMDLINE_LINUX /etc/default/grub

###
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash i915.enable_guc_loading=1 i915.enable_guc_submission=1 i915.enable_psr=2 i915.enable_fbc=1"
GRUB_CMDLINE_LINUX=""
###

```

PSR can be set to any digit between 0 (off) and 7 (deep+ deeper + deepest) sleep modes. However, on my system I get an immediate freeze if anything > 2 is modified. This is also confirmed by the [archwiki](https://wiki.archlinux.org/index.php/Dell_XPS_13_(9360)) entry.
All of these parameters do not lead to a system freeze or weird effects. GUC drivers can be found on the intel website, and will be shipped with future linux-firmware packages ( e.g. the arch package already includes them ). 
From kernel 4.11 on the option `enable_fbc=1` is enabled by default. This option is apparently one of the main powersavers.


# Error messages

If after booting and running:

```
dmesg | grep aer
```
Lots of messages appear at the startup and fill up the log significantly, the error messages can be disabled by setting in the BIOS the option `fastboot` to `through` [according to comment #76](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1521173)