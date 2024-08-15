# Conducting an orchestra
*   Apply patches and updates via yum, apt and other package manangers.
*   Check resource usage(disk space, memory, CPU, swap space, network).
*   Check log files.
*   Manage system users and groups.
*   Manage DNS settings, hosts files, etc
*   Copy files to and form server.
*   Deploy applications or run applications maintenance.
*   Reboot servers.
*   Manage cron jobs.

## Inventory file for multiple servers
```yml
[defaults]
inventory = hosts.ini
```

```ini
# hosts.ini
# Lines beginning with a # are comments, and are only included for illustration. These comments are overkill for most inventory files.

# Application servers
[app]
192.168.60.4
192.168.60.5

# Database server
[db]
192.168.60.6

# Group 'multi' with all servers
[multi:children]
app
db

# Variables that will be applied to all servers
[multi:vars]
$ ansible_user=vagrant
$ ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key
```

## First ad-hoc commands
### Discover Ansible's parallel nature
```bash
$ ansible multi -a "hostname"
```

!!! tip
    If Ansible reports **<span class="rouge">No Hosts</span>** matched or returns some other inventory-related **error**, you might not have your `ansible.conf` file in the correct directory, or it might not be the correct syntax. You can also try overriding the inventory file path by setting the **ANSIBLE_INVENTORY** environment variable explicitly: export **<span class="jade">ANSIBLE_INVENTORY=hosts.ini</span>**.

## Learning about your environment
### Disk space on the servers
```bash
$ ansible multi -a "df -h"
```
### Memory free
```bash
$ ansible multi -a "free -m"
```

## Make changes using Ansible modules
### Chrony daemon to keep time in sync
```bash
# -b = become -m = module
$ ansible multi -b -m yum -a "name=chrony state=present"
$ ansible multi -b -m service -a "name=chrony state=started enabled=yes"
$ ansible multi -b -a "chronyc tracking"
```
### Make changes to just specfic server
```bash
$ ansible app -b -a "service chrony restart" --limit "192.168.60.4"

# Regular expression[~]
$ ansible app -b -a "service ntpd restart" --limit ~".*\.4"
```

## Configure groups of servers, or individual servers
Since set up two seperate gorups in inventory file, `app` and `db`, target commands to just the servers in those groups.
### Configure the Application servers
The hypothetical web application uses Django, need to make sure Django ans its dependencies are installed. Django is not the official CentOS yum repository, using Pip for installation:
```bash
$ ansible app -b -m yum -a "name=python3-pip state=present"
$ ansible app -b -m pip -a "name=django<4 state=present"
```

Check to make sure Django is installed and working correctly:
```bash
$ ansible app -a "python -m django --version"
```

Let's install MariaDB ans start it:
```bash
$ ansible db -b -m yum -a "name:mariadb-server state=present"
$ ansible db -b -m service -a "name=mariadb state=started enabled=yes"
```

Configure the system firewall to ensure only the app servers can access the database:
```bash
$ ansible db -b -m yum -a "name=firewalld state=present"
$ ansible db -b -m service -a "name=firewalld state=started enabled=yes"
$ ansible db -b -m firewalld -a "zone=database state=present permanent=yes"
$ ansible db -b -m firewalld -a "source=192.168.60.0/24 zone=database state=enabled permanent=yes"
$ ansible db -b -m firewalld -a "port=3306tcp zone=database state=enabled permanent=yes"
```

Assorting mysql_* modules with Ansible to control MariaDB server:
```bash
$ ansible db -b -m yum -a "name=python3-PyMySQL state=present"
$ ansible db -b -m mysql_user -a "name=django host=% password=1234 priv=*.*:ALL state=present"
```

## Manage users and groups --- <span class="vividorange">user & group module</span>

### Ansible's `user` and `group` modules make things pretty simple and standard across any Linux flavor.

Add an `admin` group on the app servers
```bash
$ ansible app -b -m group -a "name=admin state=present"
```

Add the user `johndoe` to `admin` group
```bash
$ ansible app -b -m user =a "name=johndoe group=admin createhome=yes"
```

??? tip
    Additional parameters:

    1). generate_ssh_key=yes

    2). uid=[uid]

    3). gid=[gid]

    4). shell=[shell]

    5). password=[encrpyted-password]

Delete the account
```bash
$ ansible app -b -m user -a "name=johndoe state=absent remove=yes"
```

:material-google-downasaur: [Official Ansible: User module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html)

## Manage packages --- <span class="vividorange">package module</span>

A generic `package` module that can be used for easier cross-platform.
```bash
$ ansible app -b -m package -a "name=git state=present"
```

## Manage files and direcoties
### Get information about a file --- <span class="vividorange">stat module</span>
If you need to check a file's permission, MD5, or owner, use Ansible's `stat` module:
```bash
$ ansible multi -m stat -a "path=/etc/environment"
```
### Copy a file to the servers --- <span class="vividorange">copy module</span>
Most file copy operations can be completed with Ansible's `copy` module:
```bash
$ ansible multi -m copy -a "src=/etc/hosts dest=/tmp/hosts"
```
!!! tip
    The <span class="rouge">copy</span> module is perfect for single-file copies, and works very well with small directories. When you want to copy hundreds of files, especially in very deeply-nested directory structures, you should consider either copying then expanding an archive of the files with Ansible's <span class="rouge">unarchive</span> module, or using <span class="jade">synchronize</span> or <span class="jade">rsync</span> modules.

### Retrieve --- <span class="vividorange">fetch module</span>
```bash
$ ansible multi -b -m fetch -a "src=/etc/hosts des=/tmp"
```
!!! tip
    The parameter <span class="carrot">flat</span> could fetch the files directly into the `tmp` directory. However, filenames must be unique for working. This is only purpose for single host from.

### Create directories and files --- <span class="vividorange">file module</span>
Use the `file` module to create files and directories(like touch), manage permissions and ownership, modify
```bash
$ ansible multi -m file -a "dest=/tmp/test mode=644 state=directory"
```
Symlink(set state=link):
```bash
$ ansible multi -m file -a "src=/src/file dest=/dest/symlink state=link"
```

## Run operations in the background
It's more helpful when managing many servers.
*   -B <seconds>: the maximum amount of time(in seconds) to let the job run.
*   -P <seconds>: the amount of time(in seconds) to wait between polling the servers for an updated job status.
!!! warning
    As of **Ansible 2.0**, asynchronous polling on the command line(via the -P flag) no longer displays output in real time. [Resolution](https://github.com/ansible/ansible/issues/14681)

```bash
$ ansible multi -b -B 3600 -P 0 -a "yum -y update"
```

Check on the status elsewhere using Ansible's <span class="vividorange">async_status</span>
```bash
$ ansible multi -b -m async_status -a "jid="
```
!!! tip
    For tasks don't track remotely, it's usually a good idea to log the progress of the task _somewhere_, and also send some sort of alert on failure - especially, for example, when running backgrounded tasks that perform backup operations, or when running business-critical database mainternance task.

## Check log files
1.  Operations that continuously monitor a file, like `tail -f`, won't work via Ansible, because only displays output after the operation is complete, and won't be able to send teh Control-C command to stop following the file. Someday, the async module might have this feature.
2.  It's not good idea to run a command that returns a huge amount of data via stdout via Ansible. If going to cat a file larger than a few KB, you should probably log into the server(s) individually.
3.  If redirect and filter output from a command run via Ansible, need to use the `shell` module instead of Ansible's defualt `command` module (add -m `shell` to your commands).

A simple example:
```bash
$ ansible multi -b -a "tail /var/log/messages"
```
### Filter the messages log with something like grep, **can't** use default `command` module, but instead, shell:
```bash
$ ansible multi -b -m shell -a "tail /var/log/messages | grep ansible-command | wd -l"
```

## Manage cron jobs --- <span class="vividorange">cron module</span>
If want to run a shell script on all the servers every day at 4 a.m., add the cron job with:
```bash
$ ansible multi -b -m cron -a "name='daily-cron-all-servers' hour=4 job='/path/to/daily-script.sh'"
```
Ansible will assume * for all values that don't specify(valid values are `day, hour, minute, month and weekday`). Also specify special time values like `reboot, yearly or monthly` using <span class="jade">special_time=[value]</span>.

Furthermore, can set the user the job will run under via <span class="jade">user=[user]</span>, and ceate a backup of the current crontab by passing `backup=yes`.

Delete the cron job with:
```bash
$ ansible multi -b -m cron -a "name='daily-cron-all-server' state=absent"
```
### Customize crontab files
By specify the location to the cron file with:`cron_file=cron_file_name` (where corn_file_name is a cron file located in /etc/cron.d)

## Deploy a version-controlled application --- <span class="vividorange">git module</span>
Ansible's `git` module lets to specify a branch, tag, or even a specific commit with the `version` parameter (which also works for `branch name`, like `pro`)
```bash
$ ansible app -b -m git -a "repo=git://example.com/path/to/repo.git dest=/opt/myapp update=yes version=1.2.4"
```
!!! info
    Error message `Failed to find required executable git`, need to install **Git** on the server. `ansible app -b -m package -a "name=git state=present"`.

    Error message `unknown hostkey`, ass the option `accept_hostkey=yes` to the command, or add the hostkey to your server's `known_hosts` file before running the command.
