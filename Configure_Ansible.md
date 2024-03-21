# Automate all the things!

In the [playbooks](/playbooks) directory are the playbooks and roles used to
perform the shell steps, but codified as an ansible playbook. It also contains
some logic to be idempotent and some sanity checking with the variables.

## Prerequisite

* Do not run as root
* Create a non-root user with sudo access
* Requires rhel system roles.

```bash
sudo dnf install rhel-system-roles
```

* Requires `ansible.posix` collection:

```bash
ansible-galaxy collection install -r playbooks/requirements.yml -v
```

* edit and modify the variables under `playbooks/group_vars/storage/public.yml`

```yaml
---
raid:
  devices:
    - nvme1n1
    - nvme2n1
  level: 1
  name: raid0

stratis:
  name: pool0
  state: present
  overprovision: true
  devices:
    - "/dev/md/{{ raid.name }}"
  encryption: "{{ stratis_encryption_key }}"
  file_systems:
    - name: test1
      size: "{{ 1000 * 1000 * 1000 * 2 }}"
      mount_point: /mnt/test1
```

## Running the playbook

chdir into the playbooks directory and run the playbook. You will be asked the
vault password. Here's a hint.

> "12345? Amazing. I have the same combination on my luggage." -- President Skroob, Spaceballs (1987), Movie.

```bash
cd playbooks
ansible-playbook -i inventory stratis.yml --ask-vault-password
```

You can also execute portions of the playbook using [tags][tags].

[tags]: https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_tags.html

### Defined tags

* raid - only the tasks used to create the software RAID device
* pool - only the tasks needed to create the stratis pool and encryption key
* fs - the tasks used to create and mount a stratis file system

## That's it!

You've now created a Stratis filesystem on a RAID1 blockdevice using Ansible!

[Head back to the README](/README.md)
