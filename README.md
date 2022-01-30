# Hosting a bitwarden server with vaultwarden

## Server Setup
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
Everything will be done in the home directory of the bitwarden user. This is assuming that the bitwarden user is used soley for the purpose of running bitwarden.

### SSL Certificates
`certbot certonly -d <domain>`
The certs will be placed in `/etc/letsencrypt/live/<domain>

### Configure docker
1. Copy the `docker-compose.yml` and `nginx.conf` files from wherever they've been stored.
2. Open `docker-compose.yml` and replace any instance of `<domain>` with your domain. The domain will be in the form `host.domain.tld`.
3. Open `nginx.conf` and replace any instance of `<domain>` with your domain, in the same form as before.

### Create admin token
[vaultwarden enabling admin](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page "vaultwarden enabling admin")
Run `openssl rand -base64 48` and copy the result. Open `docker-compose.yml` and replace `<admin_token>` with the result.

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