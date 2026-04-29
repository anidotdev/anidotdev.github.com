+++
date = '2026-04-29T16:00:00+05:30'
draft = false
title = 'Proxy Servers'
+++

# Proxy Servers

We hear all these words like Docker, Redis, Kubernetes etc. But if you zoom out, the entire stack of backend is responsible for only one thing, that is, to receive requests from the client, process it and then send the response back to the client, as efficiently, fast and cheaply as possible. **That's it**.

And a proxy server is one of those concepts that I was learning about yesterday so I thought to myself, why not write about it?
So today I will try to guide you on how to create a proxy server.

Proxy servers are intermediaries that lie between the client and the backend. It might look like an extra overhead, because why will we want to add another step between client and the backend?

But the real role of proxy is much more important, for example, instead of your backend handling every request, the proxy can cache responses or distribute traffic across multiple servers. It can improve security, route traffic and many other things.

For this article, I am more interested to talk about how we can setup our own proxy server.

The following is how the flow of a request and response happens between client and server via a proxy -

{{< figure src="Untitled-2026-03-23-0026.svg" alt="Proxy Server Flow" >}}


There are many different tools that we can use to setup a proxy server, like Nginx, Apache, Envoy etc.
I will use Nginx to create a proxy server.


## Nginx (Engine-x)

Nginx ( spelled as Engine-x) is an open source software that we can use to make web servers, proxies and load balancers etc.

Before we start, make sure you have Nginx installed on your device. I will be using Linux ( Ubuntu ).

##### **Architecture of Nginx -**

Nginx runs two processes, a main / master process and many worker processes. The main process works as a central controller and controls other worker processes and worker processes on the other hand are responsible for handling the client requests.

To start Nginx, just run the executable file and the process will start.

Then to control the starting, stopping and reloading of Nginx, use the following command,

`nginx -s signal`

where signal can be any of the following -

`stop` - to stop nginx
`reload` - to reload nginx
`quit` - to quit nginx
`reopen` - to reopen nginx

### Folder Structure of Nginx

`/etc/nginx/`
`├── nginx.conf`
`├── sites-available/`
`├── sites-enabled/`
`├── conf.d/`

- `nginx.conf` stores the main config
- `sites-available/` stores all other configs
- `sites-enabled/` stores active configs (symlinked)
- `conf.d/` stores extra configs

For this tutorial, we won't be touching the main config file, so let it remain as it is. We will make our own config inside of the `conf.d/` directory

Let's start by making a new file inside of `conf.d/` named `myproxy.conf` using the following command
```bash
sudo touch /etc/nginx/conf.d/myproxy.conf
```

Then lets make an HTML file that we will serve using Nginx, keep it inside of `/var/www/myproxy`

Lets make the directory first, by using the following command,
```bash
sudo mkdir -p /var/www/myproxy
```

and then create a new HTML file using,
```bash
sudo nano /var/www/myproxy/index.html
```

You can write anything that you desire in that HTML file based on what you want to display in your browser.
```html
<h1>Hello Animesh</h1>
<p>Your Proxy is working.</p>
```

Save it and then give it the right permissions so that Nginx would be able to access the files, use the following command to do that
```bash
sudo chown -R www-data:www-data /var/www/myproxy
```

Now let's go inside the Nginx config that we created,
```bash
sudo nano /etc/nginx/conf.d/myproxy.conf
```

and write this inside and then save the config file,
```nginx
server {
	listen 80 default_server;
	
	location / {
		proxy_pass http://127.0.0.1:3000;
	}
}

server {
	listen 127.0.0.1:3000;
	
	location / {
		root /var/www/myproxy;
		index index.html;
	}
}
```

Then we need to test and reload the config file for the changes to be implemented. Use the following commands to do that,
```bash
sudo nginx -t
sudo systemctl reload nginx
```

Let me explain what this piece of code is doing, an Nginx config is made up of directives, blocks, and contexts.

Directives are single lines of code that ends with a `;` like for example `listen 80 default_server;` is a directive.

Now when directives are written inside `{ }` it is known as a block, for example, this is a block
```nginx
location / {
	root /var/www/myproxy;
	index index.html;
}
```

contexts define where certain directives or blocks are allowed to be used, for example, this is a context
```nginx
server {
	listen 80 default_server;
	
	location / {
		proxy_pass http://127.0.0.1:3000;
	}
}
```

The `server { }` block has certain rules, like defining which port to listen to. Here we are listening to port `80` which is the default port for HTTP.

So that's what an Nginx config is made up of. Now let's go back to our code and break it down.
```nginx
server {
	listen 80 default_server;
	
	location / {
		proxy_pass http://127.0.0.1:3000;
	}
}

server {
	listen 127.0.0.1:3000;
	
	location / {
		root /var/www/myproxy;
		index index.html;
	}
}
```

For simplicity, we are using Nginx itself as the backend server here, instead of something like Node.js. One will act like a proxy and the other one as a normal backend server which will serve the HTML file that we created earlier.

Here we are making two servers using the `server { }`  block, one is listening to port `80` and the other is listening to port `3000` and we do that by using the `listen` directive inside of the `server` block.
We have made the server listening to port `80` as our default server by using the command `default_server`,

what it means is that if the client sends a request to an IP:PORT combination that does not match any server block, then the request will be sent to the server with the `default_server`. In real setups, we use domain names instead of `default_server`.

Then we use the `location` directive to define how Nginx should handle requests for a specific path or route.

In our case, we are using `location /`, which means this block will match all of the incoming requests.

For example, in the first server,
```nginx
location / {
    proxy_pass http://127.0.0.1:3000;
}
```

we are telling Nginx to forward any incoming request to our backend server running on port `3000` using the `proxy_pass` directive.
So now, any incoming request that hits port `80` will be forwarded to our backend server.

**And this is how we have created our own small proxy server.**

Now let's take a look at what our backend server is doing...
```nginx
server {
    listen 127.0.0.1:3000;
    
    location / {
        root /var/www/myproxy;
        index index.html;
    }
}
```

The backend server is listening to `127.0.0.1:3000` which means it is only accessible within the same machine and not from the outside.

Here we are again using the `location /`  directive to match all the incoming requests. But instead of forwarding the request like we did in our proxy server, here we will use the `root` directive to directly serve the files, which in our case is inside of `/var/www/myproxy`.

And the `index` directive tells Nginx which file to serve by default, for us it will be the `index.html` file.
So whenever a request reaches this server, it simply returns the `index.html` file that we created earlier.

And that's it.

We have successfully created a proxy server in Nginx that will forward any incoming request to our backend server and then the backend server will serve the file to the client via the proxy.

So now the full flow looks like this,  
`Client → Nginx (port 80) → Backend (port 3000) → Nginx → Client`

We can do a lot more cool stuff but I think this is good enough for today.

Happy Learning :D
