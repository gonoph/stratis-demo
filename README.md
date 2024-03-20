# stratis-demo
Scripts and playbooks used to demo stratis: https://stratis-storage.github.io/

## Layer considerations

Stratis uses technology already included in the Linux stack:

* device-mapper
* LUKS
* XFS
* Clevis

You can add additional layers, but be mindful of where it sits on the stack. Examples would be:

* VDO
* Software RAID

## Prerequisites - You will need

* A VM running [RHEL 9.3][rhel], which you can run via
    * a [developer subscription][rhel-dev]
    * In the public cloud via [PAYG or BYOS][ccsp]
* Two (2) additional disks / storage attached to the VM of at least 10 GB each.
* stratis and software raid tools installed (below)

[rhel]: https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux
[rhel-dev]: https://developers.redhat.com/articles/faqs-no-cost-red-hat-enterprise-linux
[ccsp]: https://www.redhat.com/en/blog/how-deploy-red-hat-enterprise-linux-cloud

## Install stratis and RAID

First, you need to install software.

### Install stratis

```bash
dnf install stratis-cli.noarch stratisd-tools.x86_64 stratisd.x86_64
systemctl enable stratisd
```

### Install MD Raid Admin tools
```bash
dnf install mdadm
```

## Configure the storage

* With [Cockpit](/Configure_Cockpit.md) aka Easy Mode
* With [Command-line](/Configure_Command.md) aka Advanced Mode
* With [Ansible](/Configure_Ansible.md) aka Automate all the things!

## Monitoring

There's several aspects to monitor, but it's beyond the scope of this project
to be prescriptive. However, there are areas of focus and some commands that
are useful.

* You could configure [Ansible Automation Platform][aap] to run playbooks to collect metric information and then alert on it.
* [Performance Co-Pilot][pcp] can capture performance metrics, send to logs, or somewhere else.
* [Zabbix][zabbix] is a great monitoring tool with a lot of features.
* [Splunk][splunk] could also consume all the logs on the system, and you could set up alerts.
* And finally, you could setup [grafana, prometheus, and node exporter][grafana] (I haven't done this, but it looks cool).

[aap]: https://www.redhat.com/en/technologies/management/ansible
[pcp]: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/monitoring_and_managing_system_status_and_performance/setting-up-pcp_monitoring-and-managing-system-status-and-performance
[zabbix]: https://www.zabbix.com/index
[splunk]: https://www.splunk.com/
[grafana]: https://grafana.com/docs/grafana-cloud/send-data/metrics/metrics-prometheus/prometheus-config-examples/noagent_linuxnode/

A couple of areas need to be monitored:

* Monitoring the RAID.
* Monitoring the pool allocation.

### Raid status

You can use mdadm to email you of alerts.

```bash
# mdadm --detail --scan >> /etc/mdadm.conf
# vi /etc/mdadm.conf
MAILADDR example@example.com
```

or you can process the proc file system for errors:

```bash
# cat /proc/mdstat
Personalities : [raid1]
md127 : active raid1 nvme2n1[1] nvme1n1[0]
      10476544 blocks super 1.2 [2/2] [UU]
      bitmap: 1/1 pages [4KB], 65536KB chunk

unused devices: <none>
```

Or monitoring the logfiles / journal

```bash
# journalctl -t mdadm
Mar 19 19:24:59 ip-172-30-4-223.aws.awesomedemos.net mdadm[134839]: Rebuild74 event detected on md device /dev/md/raid0
Mar 19 19:25:20 ip-172-30-4-223.aws.awesomedemos.net mdadm[134839]: RebuildFinished event detected on md device /dev/md/raid0
Mar 19 20:13:51 ip-172-30-4-223.aws.awesomedemos.net mdadm[149003]: Rebuild74 event detected on md device /dev/md/raid0
Mar 19 20:14:11 ip-172-30-4-223.aws.awesomedemos.net mdadm[149003]: RebuildFinished event detected on md device /dev/md/raid0
Mar 20 00:02:12 ip-172-30-4-223.aws.awesomedemos.net mdadm[169881]: Rebuild73 event detected on md device /dev/md/raid0
Mar 20 00:02:33 ip-172-30-4-223.aws.awesomedemos.net mdadm[169881]: RebuildFinished event detected on md device /dev/md/raid0
Mar 20 04:10:07 ip-172-30-4-223.aws.awesomedemos.net mdadm[196024]: Rebuild73 event detected on md device /dev/md/raid0
Mar 20 04:10:29 ip-172-30-4-223.aws.awesomedemos.net mdadm[196024]: RebuildFinished event detected on md device /dev/md/raid0
Mar 20 04:13:11 ip-172-30-4-223.aws.awesomedemos.net mdadm[199724]: Rebuild72 event detected on md device /dev/md/raid0
Mar 20 04:13:34 ip-172-30-4-223.aws.awesomedemos.net mdadm[199724]: RebuildFinished event detected on md device /dev/md/raid0
Mar 20 04:22:52 ip-172-30-4-223.aws.awesomedemos.net mdadm[203274]: Rebuild72 event detected on md device /dev/md/raid0
Mar 20 04:23:15 ip-172-30-4-223.aws.awesomedemos.net mdadm[203274]: RebuildFinished event detected on md device /dev/md/raid0
Mar 20 05:08:10 ip-172-30-4-223.aws.awesomedemos.net mdadm[209608]: Rebuild72 event detected on md device /dev/md/raid0
Mar 20 05:08:33 ip-172-30-4-223.aws.awesomedemos.net mdadm[209608]: RebuildFinished event detected on md device /dev/md/raid0
```

### Thin provisioning

Typically you would monitor the file system with `df`, but with thin
provisioning, the file system is not taking up as much space as if it would if
it was on a regular block device.

You can have 5x 5GiB filesystems on a thinly provisioned device that is only
5GiB in physical storage. This is due that the amount of data stored is only as
large as that which is actually stored.

### Blockdevice

```bash
# stratis blockdev

Pool Name   Device Node              Physical Size   Tier   UUID
pool0       /dev/md127 (/dev/dm-0)        9.98 GiB   DATA   15f99708-db87-4e79-ae10-c66f2fcaaf0b
```

### Pools

```bash
# stratis pool

Name                 Total / Used / Free    Properties                                   UUID   Alerts
pool0   9.98 GiB / 600.44 MiB / 9.39 GiB   ~Ca, Cr, Op   138d56e2-bcf7-4069-b81b-36769628ef50   WS001

# stratis pool explain WS001

Every device belonging to the pool has been fully allocated. To increase the allocable space, add additional data devices to the pool.
```

In the above example, the Properties denote that `pool0` is able to be `Overprovisioned`.

### File systems
```bash
stratis filesystem

Pool    Filesystem   Total / Used / Free            Created             Device                     UUID
pool0   test1        1.86 GiB / 73 MiB / 1.79 GiB   Mar 20 2024 05:07   /dev/stratis/pool0/test1   08a57e65-006e-49c6-9df7-438b23d59dc2
```

### stratis report

You could easily feed this into an Ansible job for status and metric information.

```json
{
    "name_to_pool_uuid_map": {},
    "partially_constructed_pools": [],
    "path_to_ids_map": {},
    "pools": [
        {
            "available_actions": "fully_operational",
            "blockdevs": {
                "cachedevs": [],
                "datadevs": [
                    {
                        "blksizes": "base: BLKSSSZGET: 512 bytes, BLKPBSZGET: 512 bytes, crypt: BLKSSSZGET: 512 bytes, BLKPBSZGET: 512 bytes",
                        "in_use": true,
                        "key_description": "mykey",
                        "path": "/dev/md127",
                        "size": "20920320 sectors",
                        "uuid": "15f99708-db87-4e79-ae10-c66f2fcaaf0b"
                    }
                ]
            },
            "filesystems": [
                {
                    "name": "test1",
                    "size": "3906250 sectors",
                    "used": "76546048 bytes",
                    "uuid": "08a57e65-006e-49c6-9df7-438b23d59dc2"
                }
            ],
            "fs_limit": 100,
            "name": "pool0",
            "uuid": "138d56e2-bcf7-4069-b81b-36769628ef50"
        }
    ],
    "stopped_pools": []
}
```

## DISCLAIMER

I'm a Solutions Architect with Red Hat with a background in application
development, systems administration, and a hint of networking experience. I
introduce customers to Red Hat products, and help them gain the most value from
these products. I am not releasing this as a representative of Red Hat. The
purpose of this project is to demonstrate some capabilities of the stratis
command, cockpit, and ansible playbooks.

Use at your own risk, preferably in a development or sandbox environment.
