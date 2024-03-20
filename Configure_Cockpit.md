# Easy Mode - Use Cockpit

## Install cockpit

If it's not already installed, you will need to do these steps under fedora or RHEL.

```bash
dnf install cockpit cockpit-storaged
systemctl enable --now cockpit.socket
```

## Configure the storage

* Navigate to `https://yourserver.example.com:9090`
* [Login](/imgs/login.png) w/ your username and password.
* Select [Storage](/imgs/storage.png) from the menu tab on the right
* Turn on [Admin Mode](/imgs/admin.png)

Clicking the "blue hamburger" under Devices should give you a menu to add RAID.

![devices tab](/imgs/devices.png)

Select `Create RAID device` and configure your software RAID.

![create raid](/imgs/raid.png)

* Name: raid0
* RAID level: RAID 1 (mirror)
* Disks: [ select both disks ]

You should see something like under Jobs

```
Jobs
------------------------------
Synchronizing RAID device raid0         13%         1 minute
```

When that is complete, you can create a stratis device by select the "blue
hamburger" under Devices again, and selecting `Create Stratis pool`.

![create stratis](/imgs/stratis.png)

Make sure you select:

* Name: pool0
* Block devices: raid0
* [x] Encrypt data with a passphrase
* Passphrase: ########
* Confirm: ########
* [ ] Encrypt data with a Tang keyserver
* [x] Manage filesystem sizes

When that is complete, you should see a new entry under Devices, under the raid:

```
Devices
----------
raid0
10.7 GB RAID device
/dev/md/raid0

pool0
10.7 GB Stratis pool
/dev/stratis/pool0/
```

Click on the stratis pool (it's a link), and then `Create new filesystem`.

![create filesystem](/imgs/filesystem.png)

Configure it as:

* Name: test1
* Size: 2GB
* Mount point: /mnt/test1
* Mount Options: (don't select these)
    * [ ] Mount read only
    * [ ] Custom mount options
* At boot: Mount without waiting, ignore failure
* Click: Create and mount

## That's it!

You've now created a Stratis filesystem on a RAID1 blockdevice using the Cockpit UI!

[Head back to the README](/README.md)
