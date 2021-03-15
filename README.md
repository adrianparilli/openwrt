# OpenWRT for Old devices:

## TL;DR
### DO NOT BUY DEVICES WITH 4MB FLASH / 32MB RAM!


## Introduction:
This code builds a *discontinued* version of OpenWRT (18.06), and has been specifically modified to build targets which due its reduced RAM/space, a.k.a ["4/32 devices"](https://openwrt.org/supported_devices/432_warning), are unable to hold more recent versions of 'standard' OpenWRT, and therefore, support has been discontinued for them.

Already as of version 18.06, some devices are misworking despite being supported: Such is the case of TL-WA850RE v2 (and probably others of the same kind), which it's unable to keep changes between reboots due that firmware size is too big for the available space (<4 MB) leaving no room for overlay partition. As of 19.07 and afterwards, support for this, and many other 4/32 devices has been completely dropped and for version 21.02 and afterwards most probably none will have support at all.

While the reason for this it's totally justified (Linux code grows every day and simply there si no room for fit in these devices) many people is open to give a 'last update' or keep providing own support for their devices, no matter the reasons behind.

While getting a more modern router it's the most viable, and recommended option, many of these "4/32 devices" can still serve as 'home routers' or be re-used for different, specific purposes. Also, not anyone is in the power of discard a fully functional router for a newer one, very specially for economical reasons or lack of worth availability in their countries.

The only way to use OpenWRT on such devices, and expand/extend its usability for 'non-production scenarios' such as home, small business, or simply *give it away to someone that needs a basic router and can't afford one*, etc., it's by building custom firmware with less packages and dropping support for non-vital things (e.g. IPv6 support); also building its kernel specifically for the target mchine.

The approach used has been [previously described at OpenWRT Wiki](https://openwrt.org/docs/guide-user/additional-software/saving_space), and code has been modified following such guide as base. It's highly recommended to read that article to have an idea of what is being done first.

Currently, builds in this way are for the AR71XX/tiny subtarget (The TL-WR841N/ND v8) but it can be easily modified to build other devices with minimal modifications. It has been tested successfully, and currently works fine with:

- TL-WR841N/ND v8
- TL-WR1043N/ND v1.x
- TL-WA850RE v2

## DISCLAIMER:

It should work with other TP-Link routers and APs supported by OpenWRT, but has not been tested except with the above mentioned ones.

Not following the steps described below may lead to brick your device permanently.

USE AT YOUR OWN RISK !!

## Contents:

- [config-tl-wr841-v8-minimal-luci](config-tl-wr841-v8-minimal-luci): .config file template for a very minimal, still fully functional OpenWRT with LUCI and firewall app.

- [config-default](target/linux/ar71xx/tiny/config-default): Functional example of target set specifically for this router.

- [config-tl-wr841-v8-minimal-luci-mwan3-curl](config-tl-wr841-v8-minimal-luci-mwan3-curl): example of .config template based upon the the above that builds firmware with but with Load balancing, IPset and curl for the TL-WR841N/ND v8.

CAUTION: Do not edit these files manually. Use the instructions below so the builder system creates files accordign with the devices you are building firmware for!

## How to build minimal firmware for your router:

1. Check the [OpenWrt Hardware Database](https://openwrt.org/toh/start?dataflt%5BSupported+Current+Rel_releasepage*%7E%5D=18.06) to see if your device it's supported by version 18.06

2. If it's supported by model and hardware version, check its page at OpenWRT database. Look for features, issues, de-bricking methods and other info. If look suitable for you to proceed, then clone this repo on your machine.

3a. Set your environment by reading [this guide](https://openwrt.org/docs/guide-developer/build-system/install-buildsystem)

*or*

3b. Create a Docker container to be your build environment. You will need to install Docker Engine (read [these instructions]((https://docs.docker.com/engine/install/)) if you don't have it) and pull a docker image which has the full environment set. [This Docker image](https://hub.docker.com/r/p3terx/openwrt-build-env) has been used for doing this work and has prooved to work just fine.

*Once you have your environment ready, up and running (no matter the method above, it's your choice), then follow *the classic steps* to build firmware:*

4. Run `./scripts/feeds update -a && ./scripts/feeds install -a`

5. Run `cp config-tl-wr841-v8-minimal-luci .config`, in order to provision a minimal default in the next steps

6. Run `make menuconfig`, set Target Architecture and Subtarget Machine to your own. Save and exit.

7. Run `make kernel_menuconfig CONFIG_TARGET=subtarget` and go to Machine selection / <Platform> machine selection, look for your specific device, select it, *AND disable tl-wr841-v8*. Save changes and exit.

### At this point, you should save this exact setup as base for all further additions you want to put on your firmware, by saving a copy .config file as a template.

8. Run `make download` to download source code of packages you're next to build

9. Then, run `make -j<n>` where *n* is the number of cores/threads plus 1. If you have, e.g. a 4 cores / 8 threads CPU the right option would be '`make -j9`', but if you have 4 cores / 4 threads, you should use instead '`make -j5`'. Use that logic to fit with your CPU. In case of doubt, just use `make` (will take longer since you'd be using just one thread.)

### If builds without errors, you should have resulting firmwares on bin/targets/ar71xx/tiny/ (relative to the local copy of this repository's path in your machine). Pay attention to two specific files:

- <your-router-model-and-version>-squashfs-factory.bin (will be always around the size of your flash memory: ~ 3.9 MB for 'tiny' targets)

- <your-router-model-and-version>-squashfs-sysupgrade.bin (around 2.9 - 3.0 MB if nothing but 'minimal default's as of template was built.)

Looks as a firmware that is safe to flash. Consult the OpenWRT's page for your device to know how to flash safely.

From this point, additions are optional and most likely, you can just add one or two features as much, since you will run out of space quickly. *Repeat Step #8 if you added more features before to build again.*

In my case, I added Mwan3 for load balancing and a very minimal build of curl to use DDNS via custom script via crontab (luci-app-ddns was too much big.) For this, I had to drop a more busybox features rather than just IPV6 support; such as drop help commands and compress /bin/ash interpreter in order to make fit all.

## Troubleshooting / Hints

- When building exists with an error, or you experiment any of the below described issues, build with command `make V=sc` to see what is happening on each step of the build.

### 'Tiny' subtarget

- TL-WR841N/ND v8: The two above mentioned .bin files are missing, or not re-created anymore: *There is not enough room.* You need to drop other components and/or options than I did, in order to make it fit. Kind of try and error.

- TL-WA850RE v2: -sysupgrade.bin is EQUAL OR GREATER than factory.bin in size. This Wifi extender has so few available space for overlay that basically doesn't allow any other additions, even if there's room. If you added something and see this kind of scenario YOU MUST NOT flash such images, because you're in risk of brick the device!. This particular model has not known de-bricking workaround except by opening (breaking) the case and use TTL.

### 'Generic' subtarget
- TL-WR1043N/ND v1.x: Unstability and/or aleatory reboots. Reason: PRocessor is alow and not enough RAM. zram-swap is needed to overcome -partially- this issue, but the best choice it's to limit the intended additions to a minimal that allows it to work in stable way.
