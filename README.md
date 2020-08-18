# Linux support for ScrollPoint mice

In short: On current Linux releases, Scrollpoint mice should work well out of the box. On older releases, they can be made to work with a bit of tweaking.

## Horizontal Scrolling

This repo contains a kernel module to enable horizontal scrolling. This was merged in Linux 4.17, thanks to Peter Ganzhorn. From that release on, horizontal scrolling works out of the box.

### Older kernels

If for some reason you're stuck on an kernel before 4.17, you can compile this module.

You'll also need to install some udev rules, to force Linux to actually use this driver. Without this, Linux will still use the generic driver. Create a file `/etc/udev/rules.d/99-scrollpoint.rules`:

          ACTION=="add", ATTRS{idVendor}=="04b3", ATTRS{idProduct}=="3108", SUBSYSTEM=="hid", DRIVER=="hid-generic", \
          RUN+="/bin/bash -c 'echo $kernel > /sys/bus/hid/drivers/hid-generic/unbind'", \
          RUN+="/bin/bash -c 'echo $kernel > /sys/bus/hid/drivers/scrollpoint/bind'"

Replace the USB vendor and product IDs with those for your mouse. Then run `udevadm control --reload`.    

## Sensitivitiy (libinput)

Peter Ganzhorn contributed a patch to libinput 1.11 to fix Scrollpoint sensitivity. From that release on, scroll sensitivity should be fine out of the box. (libinput is used by current Xorg and by Wayland.)

### Older libinput

If you're stuck on an older libinput release, you can try this. Create a file `/etc/udev/hwdb.d/70-scrollpoint.hwdb`:

          mouse:usb:v04b3p3108:*
            MOUSE_DPI=800@126
            MOUSE_WHEEL_CLICK_ANGLE=1
            MOUSE_WHEEL_CLICK_COUNT=240
            MOUSE_WHEEL_CLICK_ANGLE_HORIZONTAL=5

Replace the USB vendor and product IDs with those for your mouse. Then run `systemd-hwdb update` and `udevadm trigger`.

## Sensitivity (Xorg evdev)

Before libinput, Xorg used the evdev driver. If you're still using that driver, you can configure the sensitivity by tweaking the VertScrollDelta/HorizScrollDelta driver parameters. Try adding something like this to `/etc/X11/xorg.conf`:

          Section "InputClass"
            Identifier "ScrollPoint"
            MatchUSBID "04b3:3100|04b3:3103|04b3:3105|04b3:3108|04b3:3109"
          
            Option "VertScrollDelta" "16"
            Option "HorizScrollDelta" "16"
          EndSection

This can also be tweaked while running X using the xinput command.
