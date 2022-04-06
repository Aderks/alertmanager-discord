discord-alertmanager
===

**Pull image & Clone Repo:**

```docker
cd $HOME && \
docker pull benjojo/alertmanager-discord && \
git clone https://github.com/Aderks/discord-alertmanager.git
```

**Adjust .env:**

```bash
DISCORD_WEBHOOK='<input_webhook_url_here>' && \

sudo tee /home/$USER/discord-alertmanager/.env<<EOF
LISTEN_ADDRESS=0.0.0.0:9095
DISCORD_WEBHOOK=$DISCORD_WEBHOOK
EOF
```

**Create service file:**

```bash
sudo tee <<EOF >/dev/null /etc/systemd/system/discord-alertmanager.service
[Unit]
Description=Discord Alertmanager
Requires=docker.service network-online.target
After=docker.service network-online.target

[Service]
WorkingDirectory=/home/$USER/discord-alertmanager
Type=oneshot
RemainAfterExit=yes

# Pre-start command
ExecStartPre=/usr/bin/docker-compose -f "/home/$USER/discord-alertmanager/docker-compose.yml" down

# Compose up
ExecStart=/usr/bin/docker-compose -f "/home/$USER/discord-alertmanager/docker-compose.yml" up -d

# Compose down
ExecStop=/usr/bin/docker-compose -f "/home/$USER/discord-alertmanager/docker-compose.yml" down

[Install]
WantedBy=multi-user.target
EOF
```

```bash
sudo systemctl daemon-reload && \
sudo systemctl start discord-alertmanager && \
sudo systemctl enable discord-alertmanager
```

---

Give this a webhook (with the DISCORD_WEBHOOK environment variable) and point it as a webhook on alertmanager, and it will post your alerts into a discord channel for you as they trigger:

![](/.github/demo-new.png)

## Warning

This program is not a replacement to alertmanager, it accepts webhooks from alertmanager, not prometheus.

The standard "dataflow" should be:

```
Prometheus -------------> alertmanager -------------------> alertmanager-discord

alerting:                 receivers:                         
  alertmanagers:          - name: 'discord_webhook'         environment:
  - static_configs:         webhook_configs:                   - DISCORD_WEBHOOK=https://discordapp.com/api/we...
    - targets:              - url: 'http://localhost:9094'  
       - 127.0.0.1:9093   





```

## Example alertmanager config:

```
global:
  # The smarthost and SMTP sender used for mail notifications.
  smtp_smarthost: 'localhost:25'
  smtp_from: 'alertmanager@example.org'
  smtp_auth_username: 'alertmanager'
  smtp_auth_password: 'password'

# The directory from which notification templates are read.
templates: 
- '/etc/alertmanager/template/*.tmpl'

# The root route on which each incoming alert enters.
route:
  group_by: ['alertname']
  group_wait: 20s
  group_interval: 5m
  repeat_interval: 3h 
  receiver: discord_webhook

receivers:
- name: 'discord_webhook'
  webhook_configs:
  - url: 'http://localhost:9094'
```

## Docker

If you run a fancy docker/k8s infra, you can find the docker hub repo here: https://hub.docker.com/r/benjojo/alertmanager-discord/
