---
sidebar_position: 3
---

# Configuration
This page goes over the `webconfig.yml` configuration and setting up the Nginx webserver for Dashactyl.

## Configuring your Settings
Because the `webconfig.yml` file is so large, this page will break down and explain each individual section.

```yaml
port: 3000
ssl: false
secret: 
environment: production
debug: false

# For MongoDB
connection_uri: 
database: Dashactyl-v2
```

The start of the settings file; The `port` is where Dashactyl will be running. `ssl` is dependant on wether you run dashactyl with SSL or not. This has been known to cause issues however if enabled with ssl, so it is recomended to keep it as false. The `secret` is a randomly genererated password that you must keep secret as it it is what the dashboards sessions are encrypted with. `environment` and `debug` are only for developers, so you don't need to worry about them. `connection_uri` is the URI for your mongodb database.

You may ask how do I create a database? Well.. It's fairly simple, just run the commands below. Remember you can change the username and/or the database name, but you'll need to set them on the settings.yml to whatever you've set the user/database name to.
```sql
mysql -u root -p

# After you've got that setup, let's go into the next step. Remember to change 'YourPasswordHere' with a secure password.
CREATE USER 'dashboard'@'127.0.0.1' IDENTIFIED BY 'YourPasswordHere'; 
CREATE DATABASE dashactyl;
GRANT ALL PRIVILEGES ON dashactyl.* TO 'dashboard'@'127.0.0.1' WITH GRANT OPTION;
quit;
```
This section is for setting up the MySQL/MariaDB database for Dashactyl.

```yaml
api:
  apicodepassword:
    user info: true
    blacklist user: true
    unblacklist user: true
    set coins: true
    set package: true
    set resources: true
    create coupon: true
    revoke coupon: true
```

This section is for managing the Dashactyl API endpoints. Each option toggles whether the endpoint can be used publicly.

```yaml
locations:
  "1":
    name: "Location Name"
    enabled: true
    package: null

    # package:
    # - default
    # - another_package_name

    renewal: true
```

This section is for the location used for creating the servers on Pterodactyl.

`"1"` is the location ID, you can find it on your Pterodactyl Panel, inside the admin area. `Locations -> Search your location, and you should see the ID on the left.`
`name` is the location name that you want to show while creating a server on Dashactyl.
`enabled` is pretty self-explanatory. It means if the location is enabled or not inside Dashactyl.
`package` checks if a user has a package on his account. If the user doesn't have the specified package in Dashactyl, the user won't be able to create the server in that location. 


```yaml
eggs:
  paper:
    display: "Paper"
    minimum:
      memory: 100
      disk: 100
      cpu: 10
    maximum:
      memory: null
      disk: null
      cpu: null
    info:
      egg: 3
      docker_image: quay.io/pterodactyl/core:java
      startup: java -Xms128M -Xmx{{SERVER_MEMORY}}M -Dterminal.jline=false -Dterminal.ansi=true -jar {{SERVER_JARFILE}}
      environment:
        SERVER_JARFILE: 'server.jar'
        BUILD_NUMBER: 'latest'
      feature_limits:
        databases: 1
        backups: 1
```

This section is for the server configuration eggs in Pterodactyl. When creating a server through Dashactyl, the package associated with this egg will be used to create it. You can set this to your liking, and/or remove the default egg to change it with another one.

```yaml
packages:
  default: "default"
  list:
    default:
      display: "The package name."
      memory: 1024
      disk: 1024
      cpu: 100
      servers: 1
    pro:
      display: "Pro Package"
      memory: 2048
      disk: 2048
      cpu: 200
      servers: 2
```

The packages displayed when a user is creating a server through Dashactyl. These packages can be customised and used with Dashactyl's currency system. The options shown are the specifications for the server(s) that will be given to the user in that package.

```yaml
store:
  memory:
    enabled: true
    cost: 10
    per: 10

  disk:
    enabled: true
    cost: 10
    per: 10

  cpu:
    enabled: true
    cost: 10
    per: 10

  servers:
    enabled: true
    cost: 10
    per: 10
```

This section is for the Dashactyl store configuration. `enabled` is whether the item should be purchasable in the shop. `cost` is how much the item costs and `per` is the amount of that item to give per purchase.

```yaml
afk:
  domain_lock: 
    - localhost:4440
  redirect_on_attempt_to_steal_code: https://www.youtube.com/watch?v=dQw4w9WgXcQ

  everywhat: 60
  gaincoins: 1

renewal:
  renewal_time: 6.048e+8
  deletion_time: 8.64e+7

  renew_fee: 10
```
This section is for the AFK page and the server renewal. More information is provided below.

AFK page:

`domain_lock` (WIP)
`redirect_on_attempt_to_steal_code` (WIP)
`everywhat` means how much time must go by before a user is able to get a coin inside the AFK page. The time is in seconds.
`gaincoins` this means how many coins will the user gain after the time you specified goes by. The time is in seconds.

Renewal:
`renewal_time` this is the time that specifies when the server must be renewed.
`deletion_time` deletion time is the time that is given to the user to renew the server after it gets suspended. If the user doesn't renew the server on the specified time after it gets suspended, the server will be deleted. The time is in seconds.
`renew_fee` is the renew fee users will need to use when renewing a server, also known as using coins. The time is in seconds.

## Setting Up Nginx
The Nginx web server will allow us to use a custom domain name and apply SSL to it.

First, we need to make sure you have `Nginx` and `certbot` installed:
```bash
sudo apt install nginx
sudo apt install certbot
sudo apt install -y python3-certbot-nginx
```

Now you can install your SSL certificate:
```bash
systemctl start nginx
certbot certonly --nginx -d <DASHACTYL_DOMAIN>
```

Make sure to replace `<DASHACTYL_DOMAIN>` with your domain name. If you have done this correctly you should see something similar to the following:
```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/your.dashactyl.domain/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/your.dashactyl.domain/privkey.pem
   Your cert will expire on date. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```
If what you saw isn't similar to what you saw in your server, we recommend you ask for support on our [Discord Server](https://discord.gg/wwpRNvkMHA).

Next, if everything's going correctly, you need to go to the Nginx sites directory and create a configuration file:
```bash
cd /etc/nginx/sites-available
nano dashactyl.conf
```

Now paste the following into the file. Make sure to replace `<DOMAIN>` and `<PORT>` with your Dashactyl domain and the port Dashactyl is running on.
```conf
server {
  listen 80;
  server_name <DOMAIN>;
  return 301 https://$server_name$request_uri;
}
server {
  listen 443 ssl http2;

  server_name <DOMAIN>;
  ssl_certificate /etc/letsencrypt/live/<DOMAIN>/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/<DOMAIN>/privkey.pem;
  ssl_session_cache shared:SSL:10m;
  ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers  HIGH:!aNULL:!MD5;
  ssl_prefer_server_ciphers on;

  location / {
    proxy_pass http://localhost:<PORT>/;
    proxy_buffering off;
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  location /afkwspath {
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_pass "http://localhost:<PORT>/afkwspath";
  }
}
```
After we've setup the main config file, we'll need to symlink it to sites-enabled:
```bash
sudo ln -s /etc/nginx/sites-available/dashactyl.conf /etc/nginx/sites-enabled/dashactyl.conf
```

  Once you have edited, saved, and symlinked your configuration file, restart Nginx with `systemctl restart nginx` and restart Dashactyl. You should see it running on that domain with SSL!
  
## Starting Dashactyl

First we need to install pm2:
```
npm install pm2 -g
```
Now you need to go to the dashboard directory and use:
```
pm2 start index.js
```
