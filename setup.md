# docker n8n + ollama setup

## 1. create instance & ssh

https://lightsail.aws.amazon.com/ls/webapp/create/instance?region=eu-central-1

- name: ubuntu-n8n-training
- settings: 16 GB RAM, 4 vCPUs, 320 GB SSD, 84 USD/month

```sh
KEY=<YOUR_KEY_PATH>.pem
sudo chmod 400 $KEY
IP=<YOUR_IP>
REMOTE_USER=ubuntu
ssh -i $KEY $REMOTE_USER@$IP
```

## 2. update system

```sh
sudo su
apt update && apt upgrade -y && apt autoremove && apt clean
```

## 3. install docker

```sh
curl -fsSL https://get.docker.com -o get-docker.sh
sh ./get-docker.sh
```

## 4. docker compose

```sh
mkdir n8n && cd n8n
nano docker-compose.yml
```

Choose one of the compose files:
- **n8n only:** [`n8n-compose/docker-compose.yml`](n8n-compose/docker-compose.yml)
- **n8n + ollama:** [`n8n+ollama-compose/docker-compose.yml`](n8n+ollama-compose/docker-compose.yml)

Update `<YOUR_DOMAIN>` in the chosen file, then:

```sh
docker compose up -d
docker ps -a
```

Do not forget to review the files and edit the placeholders:

  - <YOUR_DOMAIN>

Default values can also be updated:

  - GENERIC_TIMEZONE: Europe/Berlin
  - TZ: Europe/Berlin


### useful commands

```sh
docker compose down && docker compose up -d   # redeploy (pull_policy: always)
docker images
docker volume ls
docker network ls
docker system prune
```

## 5. init ollama models

```sh
docker exec -it ollama ollama pull deepseek-r1:1.5b
docker exec -it ollama ollama pull llama3.2
docker exec -it ollama ollama pull nomic-embed-text
docker exec -it ollama ollama run llama3.2 "hi"
docker exec -it ollama ollama list
```

## 6. AWS networking

- Open port 443 for IPv4 and IPv6 (HTTPS)
- Create and attach a static IP
- DNS: add `<YOUR_DOMAIN>` pointing to the static IP
  - https://eu-central-1.lightsail.aws.amazon.com/ls/webapp/domains/

## 7. install apache2 + certbot TLS

### install and enable modules

```sh
apt install apache2 -y
systemctl enable apache2
systemctl start apache2
a2enmod proxy proxy_http proxy_wstunnel ssl headers rewrite
```

### harden apache

Edit `/etc/apache2/conf-enabled/security.conf` and set:

```apache
ServerTokens Prod      # hide Apache version in HTTP headers (default: OS)
ServerSignature Off    # hide Apache version in error pages (default: On)
```

### step 1: create HTTP vhost (pre-certbot)

Do not forget to update the placeholders.
  - Define DOMAIN <YOUR_DOMAIN>
  - Define BACKEND_PORT 5678


Copy the HTTP vhost config from [`apache2/n8n.conf`](apache2/n8n.conf).
Update the `Define DOMAIN` and `Define BACKEND_PORT` values at the top of the file.

```sh
nano /etc/apache2/sites-available/<YOUR_DOMAIN>.conf
```

```sh
a2dissite 000-default.conf # deactivate default server
a2ensite <YOUR_DOMAIN>.conf # activate our server
apachectl configtest
```

### step 2: run certbot

Certbot will auto-generate a minimal `-le-ssl.conf` for the HTTPS vhost:

```sh
apt install certbot python3-certbot-apache -y
systemctl stop apache2
certbot --apache -d <YOUR_DOMAIN>
```

- Email: <YOUR_EMAIL>

### step 3: replace certbot SSL config with full config

Certbot's auto-generated `<YOUR_DOMAIN>-le-ssl.conf` is missing **WebSocket proxy rules**, which are **required** for n8n Chat Trigger to work.

Replace it with the full version from [`apache2/n8n-le-ssl.conf`](apache2/n8n-le-ssl.conf).
Update the `Define DOMAIN` and `Define BACKEND_PORT` values at the top of the file.

```sh
nano /etc/apache2/sites-available/<YOUR_DOMAIN>-le-ssl.conf
```

> **⚠️ Critical:** Without the WebSocket rewrite rules, n8n does not work

```sh
apachectl configtest
systemctl start apache2
systemctl status apache2
```

## 8. n8n setup

### owner account

Activate license: https://<YOUR_DOMAIN>/settings/usage

You will need:
- Email: <YOUR_EMAIL>
- License key: <YOUR_LICENSE_KEY>

### API key

- https://<YOUR_DOMAIN>/settings/api

## 9. credentials in n8n

### Azure OpenAI

Configure in n8n UI with Azure Foundry credentials.

### Ollama

Use the Docker internal DNS name (both containers share ai_network):

```
http://ollama:11434
```
