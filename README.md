Getting Started with Nginx
=======================

Nginx is a great reverse proxy HTTP server. Let's talk about why you might want to use Nginx on your web server.

## What is a Reverse Proxy server?
A Reverse Proxy server sits between your server side code and clients that are requesting information from the web. If we were to visualize this process, you can imagine typing "google.com" into your web browser (a client). Your client makes an HTTP request over port 80 to google.com. On Google's side, this request hits an IP address and a web server. On that web server there could be installed a reverse proxy HTTP server that receives that request. The reverse proxy server could forward that request onto the application on the machine that needs to service the request. Once the response is formulated, the reverse proxy server will return the request. The client never knows the difference.

Put visually, consider a very very simple web application architecture flow:

![web-diagram-basic](https://www.lucidchart.com/publicSegments/view/5571f786-0cd4-41d8-b601-65ae0a004692/image.png)

If we had multiple Node apps running on our web server, we wouldn't be able to utilize them as only one app can listen on port 80 at any given time.

Let's consider the above example again, this time with a few Node apps running on a single web server:

![web-diagram-advanced](https://www.lucidchart.com/publicSegments/view/5571f995-0ef0-48f0-814e-57df0a00c920/image.png)

Do you see the advantage here? This is just one of the things that a reverse proxy HTTP web server like nginx can do for us. Here are some others:
* Serve HTTPS/HTTP in parallel
* Auto-forward certain requests
 * block particular kinds of requests (or from certain places)
 * redirect old resources to new resources
 * site-wide redirects
* Automatically respond to static file requests rather than passing through to express/Node to handle (nginx is must faster)

OK, so let's talk about how to get Nginx set up and working in some of these ways.

## Installing nginx

On Ubuntu, this couldn't get much easier.

```
sudo apt-get install nginx
```

## Editing the sites-available/enabled file

Nginx uses a configuration file to run. Keep in mind, this file is not written in Javascript or any other scripting langauge that you might be familiar with. The format and syntax is all nginx-specific.

By default, nginx puts a basic default config file at this path: `/etc/nginx/sites-available/default`

Let's remove that, because we're going to create our own. 

```
sudo rm /etc/nginx/sites-available/default
```

We're going to edit this file in order to get nginx ready to rock and roll. To do that, you'll need to use a command line editor like Vim. If you're not familiar with basic Vim or how to edit files with Vim, check out a tutorial like [this one](http://www.engadget.com/2012/07/10/vim-how-to/).

Let's create our configuration file:

```
sudo touch /etc/nginx/sites-available/titanium
```

"titanium" was a completely arbitrary name I selected. In your case, it might match the hostname of your droplet or instance.

We'll edit our config file in just a moment. First, we need to create a shortcut between `/etc/nginx/sites-available/titanium` and `/etc/nginx/sites-enabled/titanium`. Nginx uses config files found in `sites-enabled` to be set up. We'll link up the default file found in `/sites-available` to a shortcut in `/sites-enabled` so that whenever we edit one, we're automatically editing the other (they're the same file). We can do this with one command:

```
sudo ln -s /etc/nginx/sites-available/titanium /etc/nginx/sites-enabled/titanium
```

Again, what's happening here is that we are creating a shortcut (link) between `/sites-available/titanium` and `/sites-enabled/titanium`.

## Using nginx to set up our first web app

Now that our config file is ready to go, let's create our first server block that points to a Node/MongoDB app on our DigitalOcean droplet.

We'll assume that we have an app called "favorite-places.com" that is a Node/MongoDB app.

Open your config file (in our case titanium), and let's put in the following:

```
server {

  listen 80;

  server_name favorite-places.com;
    
  location / {
    proxy_pass         http://127.0.0.1:1337/;
    proxy_redirect     off;

    proxy_set_header   Host              $http_host;
    proxy_set_header   X-Real-IP         $remote_addr;
    proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_set_header   X-NginX-Proxy     true;
    proxy_http_version 1.1;
    proxy_cache_key   "$scheme$host$request_method$request_uri";
  }

}
```

OK, so what's going on here? You can think of the code above sort of like a JSON object. The syntax is obviously different, but the principle is the same: we have keys (directives) and values. You can probably guess what some of these do. `listen` tells nginx to listen on port 80 for this particular block. `server_name` tells nginx what domain requests will be caught by this server block. You can have multiple server blocks (we'll go over that in a second).

The `location` directive essentially redirects all traffice (hence the `/`) from the favorite-places.com domain to our Node app running at localhost (127.0.0.1:1337). The rest of the options allow our proxy server to respond to requests correctly.

## Using nginx and HTTPS

To use HTTPS on your domain, there are a few steps that you'll need to go through:
1. Purchase a SSL certificate from a place like [Namecheap](http://www.namecheap.com/?aff=87614). What this means is that a third party (in this case, Namecheap) is going to be the trusted party that will tell the client (your customer + his/her browser) that you are to be trusted and that communications between him/her and your service will be encrypted. 
2. Install the SSL certificate. This process requires a few steps:
  1. Generate a CSR and a private key. Follow [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-an-ssl-certificate-from-a-commercial-certificate-authority), starting with the section entitled "Generate a CSR and Private Key." *Keep in mind, this must be done from your server, not your local machine*.
  2. Validate your domain. With Namecheap, they will want to contact someone registered on your domain that is authorized to validate the SSL certificate for your domain. This is usually an adminstrative contact, and you might have to edit the publicly viewable information on your domain records so that it will contact the correct person.
  3. Activate your SSL certificate. This is usually done on the certificate authority's website (in our example, Namecheap). You'll paste the CSR you created a couple steps ago into the web interface for Namecheap (or your authority) and it will be ready to go.
  4. Download and install your certificate. You should end up with an email at the end of this process that contains a zip file with everything you need. At this point, follow [this tutorial](https://aralbalkan.com/scribbles/setting-up-ssl-with-nginx-using-a-namecheap-essentialssl-wildcard-certificate-on-digitalocean/) starting with the section entitled "Create a certificate bundle" so that you combine the materials given to you from Namecheap in a way that your server and nginx understand. You'll want to upload the resulting files to your server *before* you unzip and combine them as described in this step. You can do this with scp:

```
scp /my/local/path/to/zip root@122.522.355.345:/destination/path
```

Once you have that ready, here's how your config might look to support HTTPS:

```
server {
  listen 80;

  server_name favorite-places.com;
  return 301 https://favorite-places.com$request_uri;

}

server {

  listen 443 ssl;
  server_name favorite-places.com;

  ssl on;
  ssl_certificate /certs/bundle.crt; #bundle created from step above
  ssl_certificate_key /certs/favorite-places.com.key; #key generated from server in steps above

  #enables SSLv3/TLSv1, but not SSLv2 which is weak and should no longer be used.
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  #Disables all weak ciphers
  ssl_ciphers ALL:!aNULL:!ADH:!eNULL:!LOW:!EXP:RC4+RSA:+HIGH:+MEDIUM;

  location / {
    proxy_pass         http://127.0.0.1:3001;
    proxy_redirect     off;

    proxy_set_header   Host              $http_host;
    proxy_set_header   X-Real-IP         $remote_addr;
    proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_set_header   X-NginX-Proxy     true;
    proxy_http_version 1.1;
    proxy_cache_key   "$scheme$host$request_method$request_uri";
  }
}
```

So, this is obviously a lot more configuration than before. Most of this are just default options for setting up SSL.

One thing to note in the configuration above is that we are now redirecting all non-HTTPS traffic to HTTPS only (see the 301 redirect in the first server block). This is because duplicating content on HTTP and HTTPS can cause problems with search engines. For example, Google can penalize you if you have the same content on HTTP as you do on HTTPS, because they consider them separate sites. Dumb, I know.

Some additional resources:
* [How to create an SSL on Nginx for Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-create-a-ssl-certificate-on-nginx-for-ubuntu-12-04)
* [How To Install an SSL Certificate from a Commercial Certificate Authority](https://www.digitalocean.com/community/tutorials/how-to-install-an-ssl-certificate-from-a-commercial-certificate-authority)
* [Setting up SSL with nginx (DigitalOcean)](https://aralbalkan.com/scribbles/setting-up-ssl-with-nginx-using-a-namecheap-essentialssl-wildcard-certificate-on-digitalocean/)

## Using nginx to serve static assets

Let's talk about how we can add configuration to our server blocks (HTTPS or HTTP) to bypass Express and serve static files directly.

```
server {
  listen 80;

  server_name favorite-places.com;

  location ~* \.(gif|jpg|png|js|css)$ {
    root `/home/root/code/public;
  }
  
  location / {
    proxy_pass         http://127.0.0.1:1337/;
    proxy_redirect     off;

    proxy_set_header   Host              $http_host;
    proxy_set_header   X-Real-IP         $remote_addr;
    proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_set_header   X-NginX-Proxy     true;
    proxy_http_version 1.1;
    proxy_cache_key   "$scheme$host$request_method$request_uri";
  }
  
}
```
The relevant new directives here is the new `location` directive.

`location` in this case uses regular expression (REGEX) to detect if the requesting URL has a filepath that matches an asset (images, Javascript or CSS). You could add any other file extensions to this list (PDF, .move, etc). If nginx sees that a client is requesting a resource that has one of these extensions, it will automatically look in the speficied folder (in this case `/home/root/code/public`) for the matching file before moving on. 

Doing the above can result in faster static file serving and can reduce the load on your Node application (makes your app more scalable).

## Using nginx to host multiple apps on one server

It's not very difficult to get nginx ready to handle multiple domains. Check out the config below:

```

server {

  listen 80;

  server_name favorite-places.com;
    
  location / {
    proxy_pass         http://127.0.0.1:1337/;
    proxy_redirect     off;

    proxy_set_header   Host              $http_host;
    proxy_set_header   X-Real-IP         $remote_addr;
    proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_set_header   X-NginX-Proxy     true;
    proxy_http_version 1.1;
    proxy_cache_key   "$scheme$host$request_method$request_uri";
  }

}

server {

  listen 80;

  server_name awesome-dating.com;
    
  location / {
    proxy_pass         http://127.0.0.1:1338/;
    proxy_redirect     off;

    proxy_set_header   Host              $http_host;
    proxy_set_header   X-Real-IP         $remote_addr;
    proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_set_header   X-NginX-Proxy     true;
    proxy_http_version 1.1;
    proxy_cache_key   "$scheme$host$request_method$request_uri";
  }

}
```

Really, all we had to change in the config was to add a new server block with a different server_name and a different port. We can't have two Node applications listening on the same port.

For the above to work, make sure you've done the following:
* Purchase both domains and pointed their nameservers to the same droplet or server instance (IP address)
* Register DNS records on your host to watch for both domains. (In DigitalOcean this means creating two domains and pointing them to the same droplet.)


## Some additional resources

[DigitalOcean tutorial on nginx](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-virtual-hosts-server-blocks-on-ubuntu-12-04-lts--3)
[How To Set Up a Host Name with DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-host-name-with-digitalocean)


## Copyright

© DevMountain LLC, 2016. Unauthorized use and/or duplication of this material without express and written permission from DevMountain, LLC is strictly prohibited. Excerpts and links may be used, provided that full and clear credit is given to DevMountain with appropriate and specific direction to the original content.
