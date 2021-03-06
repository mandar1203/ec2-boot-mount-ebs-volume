Mounts an EBS Volume on ec2 instance boot
=========================================
This utility is intended to be used in the boot phase of an EC2 instance. It will 
generate a boot command sequence, which:

- waits for the EBS volume to be attached
- formats and labels the volume, if unformatted
- adds an /etc/fstab mount entry using LABEL=
- mounts the device if not mounted.

The utility will also on Nitro-based instance types, where
the device name specified in the attach volume command is ignored. The 
volume will be assigned a device name which matches the pattern 
/dev/nvme[0-9]+n1, where the number is associated with the order 
in which the volumes was mounted.

On Nitro-based instance types, the volume will be queried for the
assigned device name using ebsnvme-id. This will ensure that 
the mount command is the same and independent of the machine type and
and order of volume attachment.

As the device name now potentially fluctuates on every boot,
the use of hard coded nvme device names is hazarduous.

## Usage
To generate the required boot commands, type:
```
$ ./generate-mount-ebs-volume-bootcmd /dev/xvdd /var/mqm wmq-data
```
The first parameter is your device name, the second the mount point and the third the label.
It will generate a bootcmd snippet, you can add to your machine's cloud-config.
```
  bootcmd:
    - mkdir -p /var/mqm
    - while [ ! -b $(readlink -f /dev/xvdd) ]; do echo "waiting for device /dev/xvdd"; sleep 5 ; done
    - blkid $(readlink -f /dev/xvdd) || mkfs -t ext4 $(readlink -f /dev/xvdd)
    - e2label $(readlink -f /dev/xvdd) wmq-data
    - grep -q ^LABEL=wmq-data /etc/fstab || echo 'LABEL=wmq-data /var/mqm ext4 defaults' >> /etc/fstab
    - grep -q "^$(readlink -f /dev/xvdd) /var/mqm " /proc/mounts || mount /var/mqm
```

It has to be hard-coded, as the write-files module of cloud-init is run after the bootcmd.
