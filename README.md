# freebsd-traefik
How To Install Traefik with Dashboard on FreeBSD 13

# Prerequisites
- FreeBSD 13
- Sudo user privileges

Traefik is included in the default FreeBSD repositories.

The installation is pretty straightforward. On FreeBSD systems, the Traefik package and the service is called traefik.

Run the following commands to update the package index and install Traefik:

```
pkg update
pkg install -y traefik
```

To enable Traefik as a service, add traefik_enable="YES" to the /etc/rc.conf file. You can us the sysrc command to do just that:

```
sysrc traefik_enable="YES"
```

Remove traefik.yml and traefik.toml and paste the following script:

```
rm /usr/local/etc/traefik.yml
rm /usr/local/etc/traefik.toml
```

```
vi /usr/local/etc/traefik.yml
```

```
global:
  checkNewVersion: true
  sendAnonymousUsage: true
entryPoints:
  web:
    address: :80
  websecure:
    address: :443
api:
  insecure: true
  dashboard: true
providers:
  file:
    filename: routes.yml
    directory: /etc/traefik
    watch: true
certificatesResolvers:
  le:
    acme:
      email: admin@example.com
      storage: /etc/traefik/acme.json
      httpChallenge:
        entryPoint: web
```

Lets touch routes.yml and acme.json in /etc/traefik

```
mkdir -p /etc/traefik
touch /etc/traefik/routes.yml
touch /etc/traefik/acme.json
```

And change chmod 600 for acme.json

```
chmod 600 /etc/traefik/acme.json
```

# Traefik Dashboard

We will use File provider for our Traefik routing in FreeBSD. Let’s create our dashboard in routes.yml

```
vi /etc/traefik/routes.yml
```

```
http:
  routers:
    dashboard-http:
      entryPoints:
      - web
      rule: "Host(`traefik.example.com`)"
      service: api@internal
    dashboard-https:
      entryPoints:
      - websecure
      rule: "Host(`traefik.example.com`)"
      service: api@internal
      tls:
        certResolver: le
```

# Crontab

Traefik does not start by default in FreeBSD. We use crontab to auto start our Traefik script

```
crontab -e
```

Paste the following:

```
@reboot /usr/local/bin/traefik --configFile=/usr/local/etc/traefik.yml
```

Finally, hit reboot to test

```
reboot
```
