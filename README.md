# VzBorg utility
## **Deduplicated, encrypted backups for the [Proxmox Virtual Environment](https://pve.proxmox.com/ )**

With VzBorg you can:
- Backup, restore, delete, list and maintain your PVE backups in a flexible and efficient way.
- Use default retention settings to keep your desired number of hourly, daily, weekly, monthly and yearly backups.
- Set up automated backups to remote repositories, simply setting appropriate ssh keys.

VzBorg uses vzdump and [BorgBackup](https://www.borgbackup.org/) to allow deduplicated, compressed backups of Proxmox guests, in an optionally encrypted repository.

## Requirements
You need a proxmox 5.x or 6.x server with a suitable BorgBackup package installed. 

If you are running proxmox 6.x you can install BorgBackup with:

`apt install borgbackup`

If you are running proxmox 5.x you must enable stretch-backports repository:

`echo "deb http://ftp.debian.org/debian stretch-backports main" >>/etc/apt/sources.list`

 Then install BorgBackup from it:

`apt update`

`apt install -t stretch-backports borgbackup`

If you want to use a remote repository, you need BorgBackup installed on the remote computer, with the same or a greater version, than the one in your proxmox server. If your remote repository is on another proxmox server, you can also install VzBorg on it.

## Installation
As user root in your Proxmox server run:

`wget -O - https://raw.githubusercontent.com/g3492/vzborg/master/install_and_update_vzborg.sh | bash`

## Usage:
`vzborg <COMMAND> [OPTIONS]`

After the \<COMMAND\> VzBorg only uses options. Specified, but not used options, are ignored.

### Commands:

VzBorg accepts one of the following commands:

| Command   | Description                        |
|:----------|:-----------------------------------|
|  backup   |Perform a backup job.               |
|  delete   |Delete a specific backup.           |
|  discard  |Discard all backups of given guests.|
|  getdump  |Recreate a dump file from a backup. |
|  help     |Show VzBorg help.                   |
|  info     |Show repository or backup info      |
|  list     |List backups in a repository.       |
|  prune    |Prune (purge) a repository.         | 
|  restore  |Restore backup from a repository.   |
|  version  |Show VzBorg, Borg and PVE versions. |
            
Examples:

`vzborg list`

List backups in repository

`vzborg backup -i 121`

Backup guest with ID 121

### Additional options
| Option          | Value      |Use with commands       |Description                             |
|:----------------|:-----------|:-----------------------|:---------------------------------------|
|-b (--backup)    |BACKUP_NAME |delete//info/restore    |Name of an existing backup (archive)    |
|-c (--config)    |CONFIG_NAME |all except help/version |Name of a config file in /etc/vzborg dir|
|-d (--dry-run)   |            |backup/prune            |Just perform a simulation               |
|-f (--force)     |            |restore                 |Force overwrite of existing VM/CT       |
|-h (--help)      |            |all except help/version |Display command help. Requires -c option|
|-i (--id)        |VM_ID       |backup/discard/restore  |PVE VM/CT ID or list of PVE VM/CT IDs   |
|-k (--keep)      |RETENTION   |prune                   |List of retention settings              |
|-m (--mode)      |MODE        |backup                  |vzdump mode (default = snapshot)        |
|-r (--repository)|REPOSITORY  |all except help/version |Borg repository                         |
|-s (--storage)   |STORAGE     |getdump/restore         |Proxmox storage (default = local)       |

### Configuration file:

 `/etc/vzborg/default`

Edit before using VzBorg, to customize defaults values for repository, encryption, etc..

You can have different config files in /etc/vzborg for managing different repositories or environments.
You can instruct vzborg to read an additional configuration file with the -c option. Ex:

`vzborg backup -c remoterepo -i 121`

Backup guest with ID 121 reading additional configuration file /etc/vzborg/remoterepo.


### Backup names

VzBorg creates backups encoding the guest ID and the backup time into the name, for example:

`vzborg-104-2020_04_02-17_12_34.tar` for an lxc container

or

`vzborg-105-2020_04_02-17_15_11.vma` for a virtual machine

When recreating a Proxmox backup file (getdump command), VzBorg will use the default PVE naming convention, for example:

`vzdump-lxc-104-2020_04_02-17_12_34.tar.lzo` for an lxc container

or

`vzdump-qemu-105-2020_04_02-17_15_11.vma.lzo` for a virtual machine


### Examples
`vzborg restore -h`

Show help about restore command.

`vzborg backup -i '101 102 307'`

Backup guests 101, 102 and 307 with default mode snapshot, to the default repository using default keep settings (rettention).

`vzborg restore -b vzborg-300-2020_03_20-13_11_46.vma -i 1300 -s local_lvm`

Restore VM from backup with name vzborg-300-2020_03_20-13_11_46.vma as VM with ID 1300 to storage local_lvm.

`vzborg list`

List all backups in the default repository.

`vzborg list -i 303 -r ssh://example.com:22/mnt/remote_borg_repo`

List all backups of guest with ID 303 existing on remote repository ssh://example.com:22/mnt/remote_borg_repo

`vzborg list -i '1230 1040 2077' -r /mnt/vzborg`

List all backups of guests with IDs 1230, 1040 and 2077 existing in local repository /mnt/vzborg


`vzborg getdump -b vzborg-13998-2020_03_20-13_08_35.tar -s backups`

Recreate from backup name vzborg-13998-2020_03_20-13_08_35.tar an lxc dump file in PVE storage backups (the file will be recreated as the compressed file vzdump-lxc-13998-2020_03_20-13_08_35.tar.lzo)

`vzborg prune -i '101 102 307'`

Prune or purge backups of guests with IDs 101, 102 and 307, on default repository, using default retentions.

`vzborg prune -i '101 102 307' -k '--keep-weekly=4 --keep-monthly=6 --keep-yearly=2'`

Prune or purge backups of guests with IDs 101, 102 and 307 on default repository, keeping 4 weekly, 6 monthly an 2 yearly backups.

## License
Licensed under GNU Affero General Public License, version 3.

## Feedback, bug-reports, requests, ...
They are welcome [here!](https://github.com/g3492/vzborg/issues)

## Important note
VzBorg is under development. Use it at your own risk