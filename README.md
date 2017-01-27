# BMT - Bhyve Management Tool

This is a super lightweight yet very functional tool to manage Bhyve VMs on FreeBSD.  It needs only `/bin/sh` and a ZFS volume to place VMs into (UFS is not supported).

It supports most UNIX OSes and Window,  handles auto-booting VMs at system start and shutting them down at system shutdown/reboot.

## Initial Setup

### ZVol Location

By default BMT will create a ZFS volume `zroot/vms` to house the VM configs and virtual disks, and mount the base in `/usr/local/vms`.

If you wish to have this somewhere else (ie; if you wish to place it on a different zroot) create a file called `/usr/local/etc/bmt.conf`:

```
# Base ZFS Root
BASE_ZPATH="zssd/vms" 
```

Where zssd is your preferred base ZFS.

### Init

The first time setting up BMT on a system (assumes bmt is installed into `/usr/local/bmt/`):

```
ln -s /usr/local/bmt/bmt.rc.sh /usr/local/etc/rc.d/bmt &&
sysrc bmt_enable="YES" &&
bmt rcstart &&
bmt setup
```

It is best to reboot after this just to be sure everything is applied.

## Usage

You can run just `bmt` to get a list of options.  This section needs to be expanded more but the following should get you started:

### List All VMs

`bmt list` will show all VMs, their states, used memory, CPU and if they're set to come up on system boot.

### Network Map

The `bmt netmap` command will attempt to map out VM's TAP devices an label them in an easy to understand way against each bridge.

You can specify user-friendly labels for the bridges by adding the following format lines to `/usr/local/etc/bmt.conf`:

```
BRIDGE_bridge0_NAME="Private LAN"
BRIDGE_bridge1_NAME="Public WAN"
```

### New VM

To create a new VM with a 16 GiB virtual disk:

```
bmt create newvmname -V 16G
```

### Edit VM

`bmt edit <vname>` launches opens the appropriate vm.conf file in your preferred editor.

### Start/Stop

To stop a VM:

`bmt stop vmname`

To start a VM:

`bmt start vmname`

(Where "vmname" is the name you gave it).

## Networking

By default with `AUTO_NETWORKING="YES"` set for a VM, a TAP device will automatically be created and assigned to bridge0 (for the example below).

*NOTE:* It will not automatically create bridge0, you still need to set that up in `/etc/rc.conf` and `ifconfig`/

This auto-provisioning can be overridden via these config blocks:

```
# -- Networking
#    Up to 12 nics are possible following the same naming convention )
#
AUTO_NETWORKING="NO" 
VM_N1_BRIDGE_NUM="0" 
VM_N1_TAP_NUM="21" 

VM_N2_BRIDGE_NUM="" 
VM_N2_TAP_NUM="" 
```

*NOTE*:  If you have PF enabled make sure to 'skip' any VM taps and bridges `/etc/pf.conf`:

```
set skip on bridge0
set skip on tap21
```

Otherwise nothing will work.  Obviously you can apply more granular control over this but this is a common issue when VM networking doesn't work.
