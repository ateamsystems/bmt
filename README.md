# BMT - Bhyve Management Tool

This is a super lightweight yet very functional tool to manage Bhyve VMs on FreeBSD.  It needs:

* `/bin/sh`
* A ZFS volume to place VMs into (UFS is not supported)
* GNU screen (`pkg install screen`)
* Grub2 Bhyve loader (`pkg install grub2-bhyve`), if you'll be running Linux VMs
* BHyve UEFI Firmware, if you'll be running Windows VMs (`pkg install -y bhyve-firmware`)

It supports most UNIX OSes and Windows, handles auto-booting VMs at system start and shutting them down at system shutdown/reboot.

## Initial Setup

### ZVol Location

By default BMT will create a ZFS volume `zroot/vms` to house the VM configs and virtual disks, and mount the base in `/usr/local/vms`.

If you wish to have this somewhere else (ie; if you wish to place it on a different zroot) create a file called `/usr/local/etc/bmt.conf`:

```
# Base ZFS Root
BASE_ZPATH="zssd/vms" 
```

Where `zssd` is your preferred ZPOOL, and `vms` is a standard dataset.

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

### Attach To Console

To attach to the text console:

`bmt attach vmname`

### Clone

To clone a vm:

`bmt clone vmname new-vmname`

### Destroy

To destroy a vm, it must be off:

`bmt destroy vmname`

### Status

To see the status of a vm:

`bmt status vmname`

### Block until vm stops

This command will block until a vm powers off:

`bmt wait_for_poweroff vmname`

### Send/Receive

The vm send/receive functionality works very similar to zfs.

This example sends the vm to a compressed file:

`bmt send vmname | xz > vmname.bmt.xz`

This example receives a vm from a compressed file:

`xzcat vmname.bmt.xz | bmt receive vmname`

This example sends a vm between hosts:

`bmt send vmname | ssh user@host bmt receive vmname`

### Get/Set

You can get and set parts of the vm configuration with these commands:

`bmt get vmname VM_CPUS`

`bmt set vmname VM_CPUS 2`

## Networking

By default with `AUTO_NETWORKING="YES"` set for a VM, a TAP device will automatically be created and assigned to bridge0 (for the example below).

*NOTE:* It will not automatically create bridge0, you still need to set that up in `/etc/rc.conf` and `ifconfig`

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
