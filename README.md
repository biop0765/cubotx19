### cubotp20

#### dd the system

```shell
adb shell su -c dd if=/dev/block/mmcblk0p30 of=/storage/emulated/0/x19-mmcblk0p30-28-12-2020.img
```

#### enable user setup complete. See last 2 lines of bootstat.rc 
```shell
su -c cp -v /storage/emulated/0/bootstat.rc /etc/init/bootstat.rc
```

Replace ```/etc/init/bootstat.rc```

```shell
bootstat.rc 
# This file is the LOCAL_INIT_RC file for the bootstat command.

# mirror bootloader boot reason to system boot reason
on property:ro.boot.bootreason=*
    setprop sys.boot.reason ${ro.boot.bootreason}

on post-fs-data
    mkdir /data/misc/bootstat 0700 system log
    # To deal with ota transition resulting from a change in DAC from
    # root.root to system.log, may be deleted after ota has settled.
    chown system log /data/misc/bootstat/absolute_boot_time
    chown system log /data/misc/bootstat/boot_complete
    chown system log /data/misc/bootstat/boot_complete_no_encryption
    chown system log /data/misc/bootstat/boot_reason
    chown system log /data/misc/bootstat/boottime.bootloader.1BLE
    chown system log /data/misc/bootstat/boottime.bootloader.1BLL
    chown system log /data/misc/bootstat/boottime.bootloader.2BLE
    chown system log /data/misc/bootstat/boottime.bootloader.2BLL
    chown system log /data/misc/bootstat/boottime.bootloader.AVB
    chown system log /data/misc/bootstat/boottime.bootloader.KD
    chown system log /data/misc/bootstat/boottime.bootloader.KL
    chown system log /data/misc/bootstat/boottime.bootloader.ODT
    chown system log /data/misc/bootstat/boottime.bootloader.SW
    chown system log /data/misc/bootstat/boottime.bootloader.total
    chown system log /data/misc/bootstat/build_date
    chown system log /data/misc/bootstat/factory_reset
    chown system log /data/misc/bootstat/factory_reset_boot_complete
    chown system log /data/misc/bootstat/factory_reset_boot_complete_no_encryption
    chown system log /data/misc/bootstat/factory_reset_current_time
    chown system log /data/misc/bootstat/factory_reset_record_value
    chown system log /data/misc/bootstat/last_boot_time_utc
    chown system log /data/misc/bootstat/ota_boot_complete
    chown system log /data/misc/bootstat/ota_boot_complete_no_encryption
    chown system log /data/misc/bootstat/post_decrypt_time_elapsed
    chown system log /data/misc/bootstat/ro.boottime.init
    chown system log /data/misc/bootstat/ro.boottime.init.cold_boot_wait
    chown system log /data/misc/bootstat/ro.boottime.init.selinux
    chown system log /data/misc/bootstat/time_since_factory_reset
    chown system log /data/misc/bootstat/time_since_last_boot
    # end ota transitional support

# Record the time at which the user has successfully entered the pin to decrypt
# the device, /data is decrypted, and the system is entering the main boot phase.
#
# post-fs-data: /data is writable
# property:init.svc.bootanim=running: The boot animation is running
# property:ro.crypto.type=block: FDE device
on post-fs-data && property:init.svc.bootanim=running && property:ro.crypto.type=block
    exec_background - system log -- /system/bin/bootstat -r post_decrypt_time_elapsed

# sys.logbootcomplete is a signal to enable the bootstat logging mechanism.
# This signaling is necessary to prevent logging boot metrics after a runtime
# restart (e.g., adb shell stop && adb shell start).  /proc/uptime is not reset
# during a runtime restart, which leads to false boot time metrics being reported.
#
# The 'on boot' event occurs once per hard boot (device power on), which
# switches the flag on. If the device performs a runtime restart, the flag is
# switched off and cannot be switched on until the device hard boots again.

# Enable bootstat logging on boot.
on boot
    setprop sys.logbootcomplete 1

# Disable further bootstat logging on a runtime restart. A runtime restart is
# signaled by the zygote stopping.
on property:init.svc.zygote=stopping
    setprop sys.logbootcomplete 0

# Record boot complete metrics.
on property:sys.boot_completed=1 && property:sys.logbootcomplete=1
    # Converts bootloader boot reason to system boot reason
    # Record boot_complete and related stats (decryption, etc).
    # Record the boot reason.
    # Record time since factory reset.
    # Log all boot events.
    exec_background - system log -- /system/bin/bootstat --set_system_boot_reason --record_boot_complete --record_boot_reason --record_time_since_factory_reset -l


on property:sys.boot_completed=1
#   exec u:r:shell:s0 shell shell input log adb sdcard_rw sdcard_r net_bt_admin net_bt inet net_bw_stats -- /system/bin/sh /system/bin/settings put global airplane_mode_on 0
    exec u:r:shell:s0 shell shell input log adb sdcard_rw sdcard_r net_bt_admin net_bt inet net_bw_stats -- /system/bin/sh /system/bin/settings put global device_provisioned 1
    exec u:r:shell:s0 shell shell input log adb sdcard_rw sdcard_r net_bt_admin net_bt inet net_bw_stats -- /system/bin/sh /system/bin/settings put secure user_setup_complete 1
```




Replace ```/etc/mkshrc```

```shell 
# Copyright (c) 2010, 2012, 2013, 2014
#	Thorsten Glaser <tg@mirbsd.org>
# This file is provided under the same terms as mksh.
#-
# Minimal /system/etc/mkshrc for Android
#
# Support: https://launchpad.net/mksh
alias cb1='echo 1 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness'
alias cb2='echo 2 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness'
alias cb3='echo 3 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness'
alias cb4='echo 4 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness'
alias cb5='echo 5 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness'
alias cb6='echo 6 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness'
alias cb7='echo 7 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness'
alias cb8='echo 8 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness'
alias cb9='echo 9 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness'

alias b1='/sbin/su -c  "echo 1 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness"'
alias b2='/sbin/su -c  "echo 2 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness"'
alias b3='/sbin/su -c  "echo 3 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness"'
alias b4='/sbin/su -c  "echo 4 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness"'
alias b5='/sbin/su -c  "echo 5 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness"'
alias b6='/sbin/su -c  "echo 6 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness"'
alias b7='/sbin/su -c  "echo 7 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness"'
alias b8='/sbin/su -c  "echo 8 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness"'
alias b9='/sbin/su -c  "echo 9 > /sys/devices/platform/leds-mt65xx/leds/lcd-backlight/brightness"'


if (( USER_ID )); then PS1='$'; else PS1='#'; fi
PS4='[$EPOCHREALTIME] '; PS1='${|
	local e=$?

	(( e )) && REPLY+="$e|"

	return $e
}$HOSTNAME:${PWD:-?} '"$PS1 "

```
