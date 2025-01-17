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
dashboard_url: 

# For MongoDB
connection_uri: 
database: Dashactyl-v2
```

The start of the settings file; The `port` is where Dashactyl will be running. `ssl` is dependant on wether you run dashactyl with SSL or not. This has been known to cause issues however if enabled with ssl, so it is recomended to keep it as false. The `secret` is a randomly genererated password that you must keep secret as it it is what the dashboards sessions are encrypted with. `environment` and `debug` are only for developers, so you don't need to worry about them. `dashboard_url` is the URL of your dashboard. This MUST be correct otherwise the frontend will not be able to send requests to the backend. This is because this setting sets the cors origin policy. `connection_uri` is the URI for your mongodb database.

You may ask how do I create a database? Well.. It's fairly simple, just follow the steps below. Remember, you can change the database name to whatever you would like, however you just need to change it in the webconfig.yml file.

![create database](/img/create_database.png)

Once you have created the database, you just need to start the application, **(MAKE SURE THE CONNECTION URI AND DATABASE IS CORRECT IN THE WEBCONFIG.YML FILE)**

Dashactyl will automatically create the necessary collections for you, and insert all the required data into those collections.

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
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header Host $host;
    proxy_buffering off;
    proxy_set_header X-Real-IP $remote_addr;
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

Once you have started Dashactyl, head to the Dashbard URL and start with the installation screen!
