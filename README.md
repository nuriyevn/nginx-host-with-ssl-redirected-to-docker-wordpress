Based on 

https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/
https://stackoverflow.com/questions/63520827/redirect-nginx-to-wordpress-docker-container
https://stackoverflow.com/questions/63135042/wordpress-on-docker-behind-nginx-reverse-proxy-using-ssl

# Redirect nginx (host) to wordpress docker container with wordpress with apache (container)

I've a webserver nginx on the host (dedicated server) with a wordpress site as a docker container (it-cluster.com.ua).
I want to redirect an endpoint of this webserver(0.0.0.0:80) to a docker container with wordpress at port 998. The endpoint must be "/".
Webserver must run SSL and redirect http traffic to https.


# 0. Prerequisites

Please make sure that all required ports(80, 443) are open

I personally prefer firewall-cmd

    firewall-cmd --list-all
    firewall-cmd --permanent --add-port=80\tcp --zone=public
    firewall-cmd --permanent --add-port=443\tcp --zone=public
    firewall-cmd --reload
    firewall-cmd --list-all

# 1. Download the Let’s Encrypt Client

First, download the Let’s Encrypt client, certbot.

As mentioned just above, we tested the instructions on Ubuntu 16.04, and these are the appropriate commands on that platform:

    $ apt-get update
    $ sudo apt-get install certbot
    $ apt-get install python-certbot-nginx

With Ubuntu 18.04 and later, substitute the Python 3 version:

    $ apt-get update
    $ sudo apt-get install certbot
    $ apt-get install python3-certbot-nginx

# 2. Set Up NGINX
certbot can automatically configure NGINX for SSL/TLS. It looks for and modifies the server block in your NGINX configuration that contains a server_name directive with the domain name you’re requesting a certificate for. In our example, the domain is www.it-cluster.com.ua

Assuming you’re starting with a fresh NGINX install, use a text editor to create a file in the /etc/nginx/conf.d directory named domain‑name.conf (so in our example, it-cluster.com.ua.conf).

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


Remove **/etc/nginx/site-enabled/default**  file (if you haven't added anything there, do it on your own risk, better copy that as a backup to be 100% sure) , because it might interfere with your newly added config, particularly with     

    listen 80 default_server;
    listen [::]:80 default_server;


# 3. Obtain the SSL/TLS Certificate

    $ sudo certbot --nginx -d it-cluster.com.ua -d www.it-cluster.com.ua
During execution it might ask 


1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):


You may choose 1 if you don't want everything to be redirected to https,
but in case if you change your mind just add the following redirect:

    # Redirect non-https traffic to https
    if ($scheme != "https") {
        return 301 https://$host$request_uri;
    } # managed by Certbot

to the final .conf file.

Final it-cluster.com.ua.conf file could be found in this repo within the foled etc/nginx/conf.d/


# 4. Automatically Renew Let’s Encrypt Certificates
Let’s Encrypt certificates expire after 90 days. We encourage you to renew your certificates automatically. Here we add a cron job to an existing crontab file to do this.

Open the crontab file.

    $ crontab -e

Add the certbot command to run daily. In this example, we run the command every day at noon. The command checks to see if the certificate on the server will expire within the next 30 days, and renews it if so. The --quiet directive tells certbot not to generate output.

    0 12 * * * /usr/bin/certbot renew --quiet


Save and close the file. All installed certificates will be automatically renewed and reloaded.


