# nginx-ssl


#1. Download the Let’s Encrypt Client

First, download the Let’s Encrypt client, certbot.

As mentioned just above, we tested the instructions on Ubuntu 16.04, and these are the appropriate commands on that platform:

$ apt-get update
$ sudo apt-get install certbot
$ apt-get install python-certbot-nginx

With Ubuntu 18.04 and later, substitute the Python 3 version:

$ apt-get update
$ sudo apt-get install certbot
$ apt-get install python3-certbot-nginx

#2. Set Up NGINX
certbot can automatically configure NGINX for SSL/TLS. It looks for and modifies the server block in your NGINX configuration that contains a server_name directive with the domain name you’re requesting a certificate for. In our example, the domain is www.example.com.

Assuming you’re starting with a fresh NGINX install, use a text editor to create a file in the /etc/nginx/conf.d directory named domain‑name.conf (so in our example, www.example.com.conf).

Specify your domain name (and variants, if any) with the server_name directive:

server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html;
    server_name www.it-cluster.com.ua it-cluster.com.ua;
}
Save the file, then run this command to verify the syntax of your configuration and restart NGINX:

$ nginx -t && nginx -s reload
or
$ service nginx restart


Remove /etc/nginx/site-enabled/default  file , because it might interfere with your newly added config, particularly with      
listen 80 default_server;
listen [::]:80 default_server;


#3. Obtain the SSL/TLS Certificate

$ sudo certbot --nginx -d example.com -d www.example.com



Final it-cluster.com.ua.conf file could be found in this repo within the foled etc/nginx/conf.d/
