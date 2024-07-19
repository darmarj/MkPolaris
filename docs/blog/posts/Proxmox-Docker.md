---
date: 2023-12-23
authors: [darmarj]
description: >
  Setup Docker on Proxmox host
categories:
  - Proxmox
  - Container
---

# Installing Docker

## Install Docker on Proxmox
We’re installing Docker along with Docker Compose from the official Docker repository according to the [docs](https://docs.docker.com/engine/install/debian/):

Update:
```bash
apt update
```

Install the required packages to access the Docker repository via HTTPS:
```bash
apt install ca-certificates curl gnupg lsb-release
```

Add Docker’s GPG key:
```bash
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Add Docker’s stable repository:
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Update and install Docker:
```bash
apt update
apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

Verify that Docker is installed correctly by running an image that prints a message and exits:
```bash
systemctl status docker
docker run --rm hello-world
```

## Configuring Docker

### <span class="vividgreen">Log Rotation</span>
By default, log rotation is disabled ([docs](https://docs.docker.com/config/containers/logging/configure/)). To enable log rotation, create a Docker config file /etc/docker/daemon.json with the following content:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

Restart the Docker daemon (`service docker restart`). To verify, run `docker info`.

### <span class="vivigreen">Docker Data Directory</span>
On the Proxmox host, create Docker data directories:
```bash
mkdir /rpool/data/docker
mkdir /rpool/encrypted/docker
```

## Troubleshooting Docker
### <span class="vividgreen">Inspecting Container Logs</span>
Docker captures stdout and stderr output from within containers and writes it to log files. To inspect, use docker compose logs (docker logs works, too), e.g., like this:

```bash
docker compose logs <container> --tail 30 --timestamps
```

:material-google-downasaur: [Helge Klein --- Docker Host on Proxmox](https://helgeklein.com/blog/installing-proxmox-as-docker-host-on-intel-nuc-home-server/)

:material-google-downasaur: [Gurucomputing --- Installing a Docker environment](https://blog.gurucomputing.com.au/homelabbing-with-proxmox/installing-a-docker-environment/)

:material-google-downasaur:[Unprivileged LXC containers](https://pve.proxmox.com/wiki/Unprivileged_LXC_containers)
