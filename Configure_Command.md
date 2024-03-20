# Advanced mode

## Create the RAID device

```bash
mdadm --create /dev/md/raid0 --level=1 --raid-devices=2  --name=raid0 /dev/nvme1n1 /dev/nvme2n1
cat /proc/mdstat
```

You should see something like this:

```
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md/raid0 started.

Personalities : [raid1]
md127 : active raid1 nvme2n1[1] nvme1n1[0]
      10476544 blocks super 1.2 [2/2] [UU]
      [=>...................]  resync =  9.0% (944512/10476544) finish=1.0min speed=157418K/sec

unused devices: <none>
```

## Create the Stratis pool and filesystem

```bash
stratis key set mykey --capture-key
# type in the key
stratis pool create --key-desc mykey --no-overprovision pool0 /dev/md/raid0
stratis fs create --size $[ 1000 * 1000 * 1000 * 2 ]B pool0 test1
eval $(blkid -p -o export /dev/stratis/pool0/test1)
grep -q $UUID /etc/fstab || echo "UUID=$UUID /mnt/test1 xfs defaults 0 0" >> /etc/fstab
systemctl daemon-reload
mount -a
```

## That's it!

You've now created a Stratis filesystem on a RAID1 blockdevice via the command line! Very Fancy!

[Head back to the README](/README.md)
