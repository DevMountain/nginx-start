Getting Started with Nginx
=======================

Nginx is a great reverse proxy HTTP server. Let's talk about why you might want to use Nginx on your web server.

## What is a Reverse Proxy server?
A Reverse Proxy server sits between your server side code and clients that are requesting information from the web. If we were to visualize this process, you can imagine typing "google.com" into your web browser (a client). Your client makes an HTTP request over port 80 to google.com. On Google's side, this request hits an IP address and a web server. On that web server there could be installed a reverse proxy HTTP server that receives that request. The reverse proxy server could forward that request onto the application on the machine that needs to service the request. Once the response is formulated, the reverse proxy server will return the request. The client never knows the difference.

Put visually, consider a very very simple web application architecture flow:

```
Client (Browser)     --------- (req) ---->     Web server (Node.js)    --------- (res) ---->     Client
```

If we had multiple Node apps running on our web server, we wouldn't be able to utilize them as only one app can listen on port 80 at any given time.

Let's consider the above example again, this time with two Node apps running on a single web server:

```

Client (Browser)     ----- (req) -->     Reverse proxy HTTP (nginx) ---/ Node app A
Client (Browser)     ----- (req) -->     Reverse proxy HTTP (nginx) ---/ Node app B
```
