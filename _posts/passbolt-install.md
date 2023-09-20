---
title: Password Manager - Passbolt
date: 2023-09-17 12:00:00 -0700
categories:
  - homelab
tags:
  - hosting
---
## Password Manager Install
```bash
sudo apt update && upgrade
sudo apt install docker.io docker-compose -y
```


```bash
mkdir passbolt
```
site to start at
https://www.passbolt.com/ce/docker

commands to run
```bash
wget "https://download.passbolt.com/ce/docker/docker-compose-ce.yaml"

wget "https://github.com/passbolt/passbolt_docker/releases/latest/download/docker-compose-ce-SHA512SUM.txt"

sha512sum -c docker-compose-ce-SHA512SUM.txt && echo "Checksum OK" || (echo "Bad checksum. Aborting" && rm -f docker-compose-ce.yaml)

docker-compose -f docker-compose-ce.yaml up -d


'''ensure you fix the app url in the compose file'''
docker-compose -f docker-compose-ce.yaml exec passbolt su -m -c
"/usr/share/php/passbolt/bin/cake passbolt register_user -u brian.biddle.w@gmail.com -f Brian -l Biddle -r admin" -s /bin/sh www-data
```

documentation site for docker install
https://help.passbolt.com/hosting/install/ce/docker.html


edit the compose file
```yaml
APP_FULL_BASE_URL: HTTPS://DOMAIN.TLD
```

Start containers
```bash
docker-compose -f docker-compose-ce.yaml up -d
```

 
## Adding SSL with traefik and cloudflare
---

add the following to the compose file under passbolt section
```yaml
found at the following url.
https://help.passbolt.com/configure/https/ce/docker/auto.html
under "add traefik service to handle https"
```

and under volumes in the traefik section add the following
```yaml
environment:
	- CF_API_EMAIL=email
	- CF_API_KEY=apikey
```
make sure to remove the ***ports*** section under the ***passbolt*** section so that their is no conflicts

create a **traefik.yaml** config file. (change email for your email.) change teh exposedby default variable to false
change the cert resolver form letsengrypt to cloudflare
and the httpChallenge to dnsChallenge

```yaml
certificatesResolvers:
	cloudflare:
		acme:
			email:brian.biddle.w@gmail.com
			...
			dnsChallenge
				provider: cloudflare
				resolvers:
					- "1.1.1.1:53"
					- "1.0.0.1:53"
			tlsChallenge: {}
```

```yaml
found at the following url.
https://help.passbolt.com/configure/https/ce/docker/auto.html
under "configuration files"
```

now create a config directory
```bash
mkdir conf
```

create 2 files under the directory
```bash
touch headers.yaml
touch tls.yaml
```

add the required info from the docs.found at the following url.
https://help.passbolt.com/configure/https/ce/docker/auto.html
under "configuration files"

now under "handle passbolt with traefik" section
add the required labels under the ***passbolt*** section. changing email 