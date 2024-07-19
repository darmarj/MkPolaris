---
date: 2024-07-10
authors: [darmarj]
description: >
  Synology + Certification
categories:
  - Synology
---

# Secure Synology NAS with a custom domain, dynamic DNS and a free SSL certification

## [ACME](https://github.com/acmesh-official/acme.sh)

| CA | valid days | ECC algorithm | max domains | wildcard | IPv4 | IPv6 | NotAfter | IDN |
| -- | ---------- | ------------- | ----------- | -------- | ---- | ---- | -------- | --- |
| Let's Encrypt | 90 | Yes | 100 | Yes | No | No | No | Yes |
| ZeroSSL | 90 | Yes | 100 | Yes | No | No | Yes | Yes |
| Google | 90 | Yes | 100 | Yes | No | No | Yes | No |
| Buypass | 180 | Yes | 5 | Paid | No | No | No | Yes |
| SSL.com | 90 | Yes | 2 | Paid | No | No | No | Yes |
| HiCA | 180 | Paid | 10 (1 if wildcard) | Yes | Paid | Paid | No | Paid |

## Pre-requisites
This is what you will need:

*   A [FreeDNS](https://freedns.afraid.org/) account to have dynamic DNS. Create an account if you don't have it.
*   A domain of your own with full control over it. If you don't have one, you can also do register.
*   acme.sh - A pure Unix shell script implementing script

## Dynamic DNS with FreeDNS
Your ISP can change your public IP without warning, and usually does it each time your router is rebooted, so you need a way to update the DNS name servers whenever that happens. __FreeDNS__ is a service that allow you to freely update DNS records so that your domain name points to your public IP all the time

### Forward your custom domain to FreeDNS servers
Change NS records on registrar(eg. GoDaddy,Namecheap,etc)
```bash
NS1.AFRAID.ORG
NS2.AFRAID.ORG
NS3.AFRAID.ORG
NS4.AFRAID.ORG
```

### Add subdomain of type A
The most important ones right now are your generic domain (eg, `whatever.com`) and the www host ( `www.whatever.com`). The IP is not that important at this point, since we will be updating it dynamically.

### Edit DDNS on Synology
*   Open Control Panel
*   Browse to `Connectivity` -> `External Access`
*   Click the DDNS tab and then click the `Add` button
*   In ***`Service Provider`***, choose __Freedns.org__
*   In Hostname, type the domain name, eg `whatever.com`
*   Fill in Username and Password with Free credentials
*   Click the Test Connection button

Waiting until the status going on `Normal` for configuration save.

## Install Docker
### Create a group for Docker
*   Open Control Panel
*   Browse to User&group
*   Click on Group and create a new group called `docker`
*   Click on Members
*   Add user to the group

## Grant docker group permission to run Docker
Issue the command to be able to use docker without having to `sudo`everything:
```bash
sudo chown root:docker /var/run/docker.sock
```

## Certificate with ZeroSSL
Browsers these days are more an more concerned about security, and while you can connect to your NAS ignoring the warnings, I prefer not having that feeling of something’s wrong by issuing my own certificate and publishing it in my Synology. __ZeroSSL__ is a service that allows you to issue a certificate for your domain at no cost.

### To come to the git service which not install on Synology.
```docker
docker run -ti --rm -v ${HOME}:/root -v $(pwd):/git alpine/git clone https://github.com/acmesh-official/acme.sh.git acmesh

cd acmesh
chmod +ux ./acme.sh
```

### Export variable for FreeDNS script for work
```bash
export FREEDNS_User="<FreeDNS username>"
export FREEDNS_Password="<FreeDNS password"
```

### Test
```bash
./acme.sh --issue --dns dns_freedns -d whatever.com -d www.whatever.com --test
```

### Register in ZeroSSL
```bash
# Replace the email address with -m option
./acme.sh --register-account -m me@whatever.com --server zerossl
```

### Issue the certificate
```bash
# The --force flag is required only if you did the --test before.
/acme.sh --issue --dns dns_freedns -d whatever.com -d www.whatever.com --force
```

After a while, the feedback will be showed up
```bash
Your cert is in: /acme.sh/whatever.com/whatever.com.cer
Your cert key is in: /acme.sh/whatever.com/whatever.com.key
The intermediate CA cert is in: /acme.sh/whatever.com/ca.cer
And the full chain certs is there: /acme.sh/whatever.com/fullchain.cer
```

## Publish the certification to Synology
To deploy the certificate in NAS. ACME has to do something called ***deploy hooks***, that take care of the required steps to publish certificate in different environments. The [Synology_dsm_hook](https://github.com/acmesh-official/acme.sh/wiki/deployhooks#20-deploy-the-cert-into-synology-dsm) will be used.

```bash
export SYNO_Username="<DSM Admin Username>"
export SYNO_Password="<DSM Admin Password>"
export SYNO_Certificate="acme.sh whatever.com ZS certificate"
export SYNO_Create=1 # default is off, means setting is not saved.  By setting to 1 we create the certificate if it's not in DSM
acme.sh --deploy -d whatever.com --deploy-hook synology_dsm
```

## Program the renewal of the cert
```bash
./acme.sh --cron
```

### Schedule a task for cron job on Synology
*   Control Panel -> Task Scheduler
*   Create -> Scheduled Task -> User-defined script
*   Give a task name eg "Renew cert" and choose the use under **docker** group
*   Set a repeat time for job loop
*   Add following script into User-defined tab:
```bash
/var/services/homes/fernando/acmesh/acme.sh --cron
```

## Change the cert in DSM as the default
*   Control Panel -> Connectivity -> Security
*   Click Certificate tab
*   The new cert should be showing up
*   Select the cert as the default for the services
*   Click OK for save.

:material-google-downasaur: [Medium](https://medium.com/@ferarias/secure-synology-nas-with-a-custom-domain-dynamic-dns-and-a-free-certificate-e61fd8e607a8)
:material-google-downasaur: [dr-b.io](https://dr-b.io/post/Synology-DSM-7-with-Lets-Encrypt-and-DNS-Challenge)
