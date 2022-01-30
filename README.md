# Hosting a bitwarden server with vaultwarden on Ubuntu

## Server Setup
### Change ssh port
In `/etc/ssh/sshd_config`, add `Port <new_port_number>` (or modify the port if the line already exists). This will prevent random attempts at accessing the server through ssh.
Run `sudo systemctl restart ssh` to restart the ssh service. Next time you ssh in, it will be on the new port.

### Create user and group
[Original bitwarden guide](https://bitwarden.com/help/install-on-premise-linux/#create-bitwarden-local-user-and-directory). This is where the user and group setup came from.
```
sudo apt install docker.io docker-compose certbot apache2-utils
sudo adduser bitwarden
sudo groupadd docker
sudo usermod -aG docker bitwarden
su bitwarden
cd ~
```

### SSL Certificates
`certbot certonly -d <domain>`
The certs will be placed in `/etc/letsencrypt/live/<domain>`

### Configure docker
Everything from here on out will assume you are logged in as the user `bitwarden` and starting in the home directory.
1. `mkdir vaultwarden`
2. Copy the `docker-compose.yml` and `nginx.conf` files into `~/vaultwarden`.
3. Open `vaultwarden/docker-compose.yml` and replace any instance of `<domain>` with your domain. The domain will be in the form `host.domain.tld`.
4. Open `vaultwarden/nginx.conf` and replace any instance of `<domain>` with your domain, in the same form as before.
5. `mkdir vaultwarden/vw-data`. This needs to be done because vaultwarden will be run as the bitwarden user. If the `vw-data` directory is not created manually, docker will create it using the root user and vaultwarden will not be able to write to that directory.

### Setup admin page
[vaultwarden enabling admin](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page "vaultwarden enabling admin")

First, generate a token to be used to access the admin page:
`openssl rand -base64 48`
 Copy the result, open `docker-compose.yml` and replace `<admin_token>` with the result.

Next is to set a username and password to access the admin site. This is a bit redundant, but gives two layers of security.
`htpasswd -cB vaultwarden/.htpasswd <username>`

### Setup SMTP Relay with sendgrid.com
[vaultwarden SMTP docs](https://github.com/dani-garcia/vaultwarden/wiki/SMTP-Configuration "vaultwarden SMTP docs")
1. Log into your account on sendgrid.com. Create an account if you don't already have one.
2. Authenticate your domain. If you have already done this, skip to step 3.
a. On the left pane expand `Settings` and select `Sender Authentication`.
b. Under `Sender Identity` then `Domain Authentication` click `Authenticate Your Domain`.
c. Follow the steps provided.
3. Setup an SMTP Relay. If you have already done this skip to step 4, but make sure you have the password (api key).
a. On the left pane expand `Email API` and select `Integration Guide`.
b. Choose SMTP Relay
c. Follow the steps provided and copy the generated password (api key) to someplace secure.
d. To verify that the SMTP relay is working, run the vaultwarden server and create a user. Login as that user and verify the email address. This should send an email and sendgrid.com can use that email as verification.
4. Open `docker-compose.yml` and replace `<sendgrid_apikey>` with the send grid apikey.

## Running
### Starting the server
Switch to the bitwarden user if needed (`su bitwarden`) and navigate to the bitwarden home directory (`cd ~`). Then run:
`docker-compose up -d`

### Stopping the server
`docker-compose down`

## Updating
[vaultwarden updating docs](https://github.com/dani-garcia/vaultwarden/wiki/Updating-the-vaultwarden-image)
```
docker-compose stop
docker-compose pull
docker-compose start
```