# The hardware, software, services and configuration used for website

## Software

### [OpnSense Router](https://opnsense.org)

#### Running version: 22.7.9_3-amd64

### [Fedora Server](https://getfedora.org/en/server/)

#### Running version: Fedora 37

* SSH password login has been disabled
* SSH key login has been enabled
* Cockpit service enabled and podman plugin added
* Username: pcontainers

Setup running systemd services on boot for pcontainers.

Run command as root:
```
loginctl enable-linger pcontainers
```

Check if loginctl is set for user:
```
ls /var/lib/systemd/linger/
```

It should show `pcontainers` in the directory.

### [Caddy Server](https://caddyserver.com)

#### Files and locations

/home/pcontainers/bin/caddy

/home/pcontainers/.config/caddy/[Caddyfile](#Caddyfile)

/home/pcontainers/secrets/caddy.env

/home/pcontainers/.config/systemd/user/[caddy.service](#caddyservice)

/data/sites/melzak_site

/data/assets/melzak

##### Caddyfile
```
{
        admin off
}
```
**REDACTED LINES**
```
*.melzak.duckdns.org, melzak.duckdns.org {
        tls {
                dns duckdns {env.DUCKDNS_API_TOKEN}
        }

        log {
                output file /data/logs/caddy/melzak.duckdns.org-access.log
        }

        @main {
                host melzak.duckdns.org
        }
        handle @main {
                encode zstd gzip
                root * /data/sites/melzak_site
                file_server
        }

        @assets {
                host assets.melzak.duckdns.org
        }
        handle @assets {
                encode zstd gzip
                root * /data/assets/melzak
                file_server
        }

        handle {
                abort
        }
}
```

##### caddy.service
```
[Unit]
Description=Caddy
Documentation=https://caddyserver.com/docs/
After=network.target network-online.target
Wants=network-online.target

[Service]
EnvironmentFile=/home/pcontainers/secrets/caddy.env
Type=notify
ExecStart=/home/pcontainers/bin/caddy \
        run \
        --environ \
        --config /home/pcontainers/.config/caddy/Caddyfile
ExecReload=/home/pcontainers/bin/caddy
        reload \
        --config /home/pcontainers/.config/caddy/Caddyfile
TimeoutStopSec=5s
LimitNOFILE=1048576
LimitNPROC=512

[Install]
WantedBy=default.target
```

##### Commands

* Set Caddy to run on ports 80 & 443

    `sudo setcap cap_net_bind_service=+ep /home/pcontainer/bin/caddy`

* Install the caddy.service:

    `systemctl --user enable /home/pcontainers/.config/systemd/user/caddy.service`

* Start the caddy.service:

    `systemctl --user start caddy.service`

* Restart the service after a change to caddy.service file:

    ```
    systemctl --user daemon-reload
    systemctl --user restart caddy.service
    ```

* Restart the service after a change to Caddyfile file:

    `systemctl --user restart caddy.service`

* Stop the caddy.service:

    `systemctl --user stop caddy.service`

* Remove the caddy.service:

    `systemctl --user disable caddy.service`

## Services

### [Duck DNS](http://www.duckdns.org)

Registered sub-domain: melzak

IPv4 and IPv6 have been set
