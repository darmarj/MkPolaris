# Finalizing

## User administration

### Adding a user for daily use

|   Group   |           Description         |
|   -----   |           -----------         |
|   audio   |   Be able to access the audio devices.    |
|   cdrom   |   Be able to directly access optical devices.     |
|   floppy  |   Be able to directly access floppy devices.      |
|   games   |   Be able to play games.      |
|   portage |   Be able to access portage restricted resources.     |
|   usb     |   Be able to access USB devices.      |
|   video   |   Be able to access video capturing hardware and doing hardware acceleration.     |
|   wheel   |   Be able to use su.      |

For instance, to create a user called larry who is member of the wheel, users, and audio groups, log in as root first (only root can create users) and run useradd:
```shell
Login:root

Password: (Enter the root password)
```

```shell
root #useradd -m -G users,wheel,audio -s /bin/bash larry
root #passwd larry

Password: (Enter the password for larry)
Re-enter password: (Re-enter the password to verify)
```

## Disk cleanup
### Removing tarballs
With the Gentoo installation finished and the system rebooted, if everything has gone well, we can now remove the downloaded stage3 tarball from the hard disk. Remember that they were downloaded to the / directory.
```shell
root #rm /stage3-*.tar.*
```
