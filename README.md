Mounts an EBS Volume and updates /etc/fstab

This utility is intended to be used in the boot commands of
an EC2 instance. It will:

- wait for the EBS volume to be attached
- format the volume, if unformatted
- add/update an entry in /etc/fstab
- unmount & remove /etc/fstab entries mounted on the same device
- unmount & remove /etc/fstab entries with the same mountpoint
- mount the device

The utility will also work on Nitro-based instance types, where
the device name specified in the attach volume command is ignored. The 
volume will be assigned a device name which matches the pattern 
/dev/nvme[0-9]+n1, where the number is associated with the order 
in which the volumes was mounted.

On Nitro-based instance types, the volume will be queried for the
assigned device name using ebsnvme-id. This will ensure that 
the mount command is the same and independent of the machine type and
and order of volume attachment.

