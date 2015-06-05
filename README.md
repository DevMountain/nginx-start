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

  server_name favorite-places.com localhost;
    
  location / {
    proxy_pass         http://127.0.0.1:1337/;
    proxy_redirect     off;

    proxy_set_header   Host             $host;
    proxy_set_header   X-Real-IP        $remote_addr;
    proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
  }

}
```

## Using nginx and HTTPS

## Using nginx to serve static assets

## Using nginx to host multiple apps on one server



[DigitalOcean tutorial on nginx](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-virtual-hosts-server-blocks-on-ubuntu-12-04-lts--3)
