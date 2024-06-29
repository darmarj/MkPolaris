---
date: 2022-01-13
authors: [darmarj]
description: >
  vaultWarden implementation with self-signed cert(openssl) assignment
categories:
  - vaultWarden
---

# vaultBitwarden

__This is a Bitwarden server API implementation written in Rust compatible with upstream Bitwarden clients*, perfect for self-hosted deployment where running the [official](https://github.com/dani-garcia/vaultwarden) resource-heavy service might not be ideal.__

## Features
Basically full implementation of Bitwarden API is provided including:

 * Basic single user functionality
 * Organizations support
 * Attachments
 * Vault API support
 * Serving the static files for Vault interface
 * Website icons API
 * Authenticator and U2F support
 * YubiKey OTP

## Installation
Work_Dir:
``` bash
mkdir ${WORK_DIR}
```

  [__Work_Dir__]: #installation

Cert:
``` bash title="certification"
openssl req -x509 -newkey rsa:4096 -keyout ${NAME}.key \
-out ${NAME}.crt -days 720 -nodes
```

Container:
``` bash title="docker run"
# Get into the Work_Dir and run following dockers command

docker run -d --restart always --name ${CONTAINER_NAME} -e \
ROCKET_TLS='{certs="/${SSL_PATH}/${SSL_NAME}.crt",key="/${SSL_PATH}/${SSL_NAME}.key"}' \
-v /$HOME/vaultwarden/ssl/:/ssl/ \
-v /$HOME/vaultwarden/vw-data:/data/ \
-p 443:80 vaultwarden/server:latest
```

!!! Note "Parameter help"
    **-e**      set the environment

    **-v**      set the volume

    **-p**      set the port

!!! Warning
    ${SSL_PATH} should be resided at the same path in [__Work_Dir__] when invoke the docker run.

[DB Backup](https://github.com/dani-garcia/vaultwarden/issues/975):

=== ":octicons-plug-16: Systemd"
	``` toml title="systemd.service"
	[Unit]
	Description=bitwarden-watch-for-changes

	[Service]
	Type=simple
	ExecStart=/home/USER/bitwardenrs/watch-for-changes.sh
	Restart=always
	WorkingDirectory=/home/USER

	[Install]
	WantedBy=multi-user.target
	```

    ``` bash title="systemd.sh"
	sudo sed -i 's/^SELINUX=.*/SELINUX=[disabled|permissive]/g' /etc/selinux/config && sudo sestatus
	sudo reboot
	sudo cp /PATH/bitwarden-watch-for-changes.service /etc/systemd/system/multi-user.target.wants
	sudo systemctl enable/start/status bitwarden-watch-for-changes.service
	sudo journalctl			//troubleshoot
	sudo chmod +x watch-for-changes.shd
	```

=== ":octicons-telescope-16: Bash"
	``` bash title="watcher.sh"
	#!/usr/bin/env bash
	export PATH=/usr/local/inotify-tools/bin/
	path_to_watch=/home/USER/bitwardenrs/bw-data

	inotifywait -m "$path_to_watch" -e create -e moved_to -e modify |
		while read path action file; do
  		if [[ $file =~ (wal|config.json) ]]; then
    		echo '1' > /home/USER/bitwardenrs/bitwarden-folder-updated;
  		fi
		done
	```
	!!! Note "[inotify-tools](https://docs.rockylinux.org/books/learning_rsync/06_rsync_inotify/): inotify's features to be used from within shell scripts"
		``` bash
		dnf -y install autoconf automake libtool

		wget -c https://github.com/inotify-tools/inotify-tools/archive/refs/tags/3.21.9.6.tar.gz

		tar -zvxf 3.21.9.6.tar.gz -C /usr/local/src/
        ```
        ``` bash
		cd /usr/local/src/inotify-tools-3.21.9.6/
		./autogen.sh && \
			./configure --prefix=/usr/local/inotify-tools && \
			make && \
			make install
		ls /usr/local/inotify-tools/bin/
		```
	!!! Tip "[Github Reference](https://github.com/inotify-tools/inotify-tools)"

=== ":octicons-database-16: DB"
	``` bash title="db-backup.sh"
	#!/usr/bin/env bash

	set -e

	bitwarden_folder_updated=/home/USER/bitwarden/bitwarden-folder-updated
	touch $bitwarden_folder_updated

	if [[ "$(cat $bitwarden_folder_updated)" == "1" ]]; then
  	rm -f /home/USER/bw-bk.tar.gz

  	docker exec bitwarden bash -c 'mkdir -p /data/db-backup && sqlite3 /data/db.sqlite3 ".backup /data/db-backup/backup.sqlite3"'

  	cd /home/USER/bitwarden/bw-data
  	tar -czvf /home/USER/bw-bk.tar.gz \
    	config.json \
    	icon_cache \
    	db-backup/backup.sqlite3

  	cd /home/USER/bitwarden/
  	tar -czvf /home/USER/bw-scripts.tar.gz \
    	backup.sh \
    	bitwarden-watch-for-changes.service \
    	watch-for-changes.sh

	mv /home/USER/bw-bk.tar.gz /SMB,NFS.AW3.../BitwardenBak/bw-bk-$(date +"%Y-%m-%d").tar.gz
	mv /home/USER/bw-scripts.tar.gz /SMB,NFS.AW3.../BitwardenBak/bw-scripts-$(date +"%Y-%m-%d").tar.gz

  	echo "0" > /home/USER/bitwarden/bitwarden-folder-updated
	else
  	echo 'nothing to backup'
	fi
	```
	??? example "Sqlite3"

		=== ":octicons-code-square-16: [sqlite3](https://sqlite.org/index.html): curl the source and complie."

			``` bash
				# dnf install make, gcc
				# tar xvfz sqlite-autoconf-<version>.tar.gz
				# cd sqlite-autoconf-<version>
				# ./configure
				# make
				# make install
			```

		=== ":octicons-code-square-16: sqlite3 in docker env:"

			``` bash
				# docker exec -it CONTAINER bash -c 'cp /datasource/sqlite3 /usr/local/bin'
			```

=== ":octicons-megaphone-16: Crontab"
	!!! example
		``` bash
		crontab -e
		0 3 * * * $PATH/*.sh > /dev/null 2>&1	# Rsync on 3:00 AM
		0 * * * * $PATH/*.sh > /dev/null 2>&1	# Rsync every hour

		crontab -l				# List the schedule
		```
