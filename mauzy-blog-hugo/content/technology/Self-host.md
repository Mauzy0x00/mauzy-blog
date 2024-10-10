+++
title = "Self Host Your Website without Opening Ports"
date = 2024-10-09T07:07:07+01:00
draft = false
hideToc = false
tags =  ["Web Development"]
+++

This post is a follow along to a Youtube video walk-through that I recorded. We will be setting up a home server to host a web application without opening any ports on my home network. To accomplish this I'll be using a Cloudflare tunnel.

>[!note]
> The `$` denotes a terminal command. Anything before `$` denotes the current working directory.
## 0. Prerequisites
1. Remote Server
2. Cloudflare Account
3. Reliable Internet Connection

>[!warning]
>A little disclaimer: I am not a professional; just a student. Do your own research but this should help you get up and running. For the most part I am just following Cloudflare and other documentation. I highly recommend you read through official documentation as needed. 
## 1. Configure Server
1. Make sure you have root permissions
	1. `sudo -l` should output (ALL : ALL) ALL on the current user.
2. Add current user to sudo as root
	1. `usermod -aG sudo mauzy`
3. Install OpenSSH
	1. `ufw allow OpenSSH`
	2. `ufw enable`
	3. `ufw status`
4. Install nginx 
	1. `sudo ufw allow 'Nginx HTTP'`
## 2. Add your site to Cloudflare
1. Register using Cloudflare
2. If you have an existing domain, click add site 
3. Copy the name servers from Cloudflare to your registrar
	- Ensure DNSSEC is disabled before doing this
	
>[!note]
>DNS Records:
> This is where you can show everyone on Discord how cool you are by adding your website as a connection.

## 3. Additional Cloudflare settings
###  Configure DNSSEC
1. Enable in Cloudflare
	1. DNS > Settings
	2. Enable DNSSEC
2. Find walkthrough for your registrar 
3. NameCheap
	1. Manage > Advanced DNS 
	2. Enable DNSSEC
	3. Copy values from Cloudflare
	4. Save it and wait an hour
	
4. Set SSl/TLS encryption mode to Full(strict)
	1. This ensures a secure connection between your origin server and Cloudflare.
	2. Cloudflare will validate the SSL certificate on your server. 
	3. This reduces the chances of man-in-the-middle attacks
		1. Find it in SSL/TLS > Overview
		2. Make sure the SSL/TLS recommender is on
5. Enable Always use Https
	1. SSL/TLS > Edge Certificates 
6. Set Authenticated Origin Pulls
	1. This ensures only requests from Cloudflare with a valid certificate are accepted by your origin server. It helps to protect against unauthorizes access to your server. 
	2. SSL/TLS > Origin Server
## 3. Create a tunnel and download the daemon for your Operating System
1. Go to your newly added site in Cloudflare
2. In the access tab, Launch the Zero Trust Platform. They move the location of this all of the time for no reason. 
3. Copy and paste into your server shell
## 4. Setup Firewall
1. Download the firewall setup script from my Github page
```BASH
~/$ curl https://raw.githubusercontent.com/Mauzy0x/Scripts/main/Cloudflare%20IPtable%20setup.sh >> ipScript.sh
```
3. Make the script executable
```BASH
~/$ chmod +x ipScript.sh
```
5. Run the script
```BAsh
~/$ ./ipScript.sh
```

## 5. Nginx
1. Set up server block
	1. Create a directory for your domain with the `p` flag to create any necessary parent directories 
	```BASH
		sudo mkdir -p /var/www/your_domain/html
	```
	2. Now assign ownership of the directory to the current user
	```BASH
		sudo chown -R $USER:$USER /var/www/your_domain/html
	```
	3. Ensure permissions are correct with chmod
		```BASH
		sudo chmod -R 755 /var/www/your_domain
		```
		- This uses octal notation. This recursively ensures the owner has full permissions
2. Configure Nginx server block
	1. `sudo rm /etc/nginx/sites-enabled/default`
	2. `sudo nano /etc/nginx/sites-available/your_domain`
3. Configure nginx configuration file
```bash
server {
    listen 5552;
    listen [::]:5552;
    server_name localhost;

    root /var/www/Mauzy-Site/;

    location / {
        try_files $uri /html/$uri; # Serve from /html/ if not found
    }

    location ~* \.(?:gif|css|js|)$ {
        try_files $uri =404;
    }

    location /html/ {
        index index.html index.htm;
        try_files $uri $uri/index.html $uri.html =404;
    }
    error_page 404 = /404.html;
}
```
3. Allow HTTPS traffic with Nginx Full which allows https and http
	1. `sudo ufw allow 'Nginx Full'`
	2. `sudo ufw reload`
4. Test config 
	1. `sudo nginx -t`
	2. `sudo systemctl restart nginx`
		1. In most cases you can just reload nginx to make changes so that applications don't go down. But in this case, let us restart nginx.
		
## 6. Trouble Shooting Connection
- If for whatever reason something is not working, lets troubleshoot. 
	1. Let us first make sure we are properly hosting our webpage 
		- `curl localhost:portNo`
		- If this command does not return your HTML then there is an issue with your NGINX configuration
	2. If you get an error where it is 'argo tunnel error' there is an issue where Cloudflare cannot see your service. 
		1. Is your tunnel config pointing to http://localhost:portNo ? 
		   It needs to be http and it needs to be the same port number specified in your Nginx config.