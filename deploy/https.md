---
title: "Cadmus Deployment - HTTPS Configuration"
layout: default
parent: "Cadmus Deployment"
nav_order: 3
---

# HTTPS Configuration

## (A) NGINX Proxy Companion

This procedure can be used whenever you want to configure a Docker-based NGINX reverse proxy. This will:

- map each domain/subdomain to any specific container you want to expose from your Docker compose stack.
- automatically get and manage LetsEncrypt certificates for all of them.

You can place your files where you want; usually I create a `/dockers` folder and then inside it a folder for each compose script. So in this case I would have:

- `/dockers/proxy` for the NGINX proxy script.
- `/dockers/cadmus-__PRJ__` for the Cadmus script (as usual, `__PRJ__` here stands for your own Cadmus project name). This also includes `env.js` modified to point to the correct base API URL. See [Cadmus app setup](setup) for more.

▶️ (1) **Create a Docker Network**: Create a Docker network that your services and the NGINX proxy will use to communicate with each other. You can create a network with the following command:

```bash
docker network create nginx-proxy
```

To check for network: `docker network ls`.

▶️ (2) **Configure Your Services** connecting them to the newly created network and linking each service to a subdomain. In your `docker-compose.yml` file, make sure each of your services is connected to the `nginx-proxy` network and add under `environment:`a list of variables like this:

```yaml
services:
  # for each service:
  your-service:
    image: your-service-image
    restart: unless-stopped
    # ...
    # link to subdomain
    environment:
      - VIRTUAL_HOST=service-subdomain.my-domain.com
      # optionally add also the port, e.g.:
      # - VIRTUAL_PORT=8080
      - LETSENCRYPT_HOST=service-subdomain.my-domain.com
      - LETSENCRYPT_EMAIL=yourEmailAddress
    # connect the service
    networks:
      - nginx-proxy

# at the end, define the network:
networks:
  nginx-proxy:
    external: true
```

>⚠️ NOTE: you might also need to specify the container's port especially when it's not standard, i.e. 8080 instead of 80, by just adding `VIRTUAL_PORT=8080` to the environment variables. Adding this does not make any harm, so it is suggested to do so.

▶️ (3) **Set Up NGINX Proxy**: Create a new Docker Compose file for the NGINX proxy. This will use the `jwilder/nginx-proxy` image, which automatically generates reverse proxy configurations for your services. To use SSL, you can use the `jrcs/letsencrypt-nginx-proxy-companion` Docker image to automatically create and manage your SSL certificates:

```yaml
services:
  nginx-proxy:
    image: jwilder/nginx-proxy
    restart: unless-stopped
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - certs:/etc/nginx/certs
      - vhost:/etc/nginx/vhost.d
      - html:/usr/share/nginx/html
    labels:
      - "com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy"      
    networks:
      - nginx-proxy

  letsencrypt-companion:
    image: jrcs/letsencrypt-nginx-proxy-companion
    restart: unless-stopped
    container_name: letsencrypt-companion
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - certs:/etc/nginx/certs
      - vhost:/etc/nginx/vhost.d
      - html:/usr/share/nginx/html
    environment:
      - NGINX_PROXY_CONTAINER=nginx-proxy      
    networks:
      - nginx-proxy

networks:
  nginx-proxy:
    external: true

volumes:
  certs:
  vhost:
  html:
```

This setup will expose port 80 (and 443 for SSL) to the outside world, and automatically route traffic to the correct service based on the domain name used to access your server. The `jwilder/nginx-proxy` image uses Docker labels to determine which service a request should be routed to, so you'll need to add a `VIRTUAL_HOST` environment variable to each of your services with the domain name for that service, as explained above.

▶️ (4) **Start Your Services and the Proxy**: Start your services with `docker compose up -d`, then start the proxy with `docker compose -f docker-compose-proxy.yml up -d`.

### Sample Cadmus Docker Compose

Here is a full sample (passwords and other sensitive data have been replaced by `...`):

```yml
services:
  # MongoDB
  cadmus-PRJ-mongo:
    image: mongo
    container_name: cadmus-PRJ-mongo
    restart: unless-stopped
    environment:
      - MONGO_DATA_DIR=/data/db
      - MONGO_LOG_DIR=/dev/null
    command: mongod --logpath=/dev/null
    ports:
      - 27017:27017
    volumes:
      - mongo-vol:/data/db
    networks:
      - nginx-proxy

  # PostgreSQL
  cadmus-PRJ-pgsql:
    image: postgres
    container_name: cadmus-PRJ-pgsql
    restart: unless-stopped
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    ports:
      - 5432:5432
    volumes:
      - pgsql-vol:/var/lib/postgresql/data
    networks:
      - nginx-proxy

  # Biblio API (remove if you don't use external bibliography)
  cadmus-biblio-api:
    image: vedph2020/cadmus-biblio-api:7.0.1
    container_name: cadmus-biblio-api
    restart: unless-stopped
    ports:
      - 60058:8080
    depends_on:
      - cadmus-PRJ-mongo
      - cadmus-PRJ-pgsql
    environment:
      - ASPNETCORE_URLS=http://+:8080
      - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus-PRJ-mongo:27017/{0}
      - CONNECTIONSTRINGS__AUTH=Server=cadmus-PRJ-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include 
      - CONNECTIONSTRINGS__BIBLIO=Server=cadmus-PRJ-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
      - SEED__BIBLIODELAY=50
      - SEED__ENTITYCOUNT=5
      - SERILOG__CONNECTIONSTRING=mongodb://cadmus-PRJ-mongo:27017/{0}-log
      - STOCKUSERS__0__PASSWORD=...
      - JWT__SECUREKEY=...
      - ALLOWEDORIGINS__0=https://cadmus-PRJ.YOURDOMAIN
      - VIRTUAL_HOST=cadmus-biblio-api.YOURDOMAIN
      - VIRTUAL_PORT=8080
      - LETSENCRYPT_HOST=cadmus-biblio-api.YOURDOMAIN
      - LETSENCRYPT_EMAIL=...
    networks:
      - nginx-proxy

  # Cadmus PRJ API
  cadmus-PRJ-api:
    image: vedph2020/cadmus-PRJ-api:0.0.1
    container_name: cadmus-PRJ-api
    restart: unless-stopped
    ports:
      - 5003:8080
    depends_on:
      - cadmus-PRJ-mongo
      - cadmus-PRJ-pgsql
      - cadmus-biblio-api
    environment:
      - ASPNETCORE_URLS=http://+:8080
      - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus-PRJ-mongo:27017/{0}
      - CONNECTIONSTRINGS__AUTH=Server=cadmus-PRJ-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include 
      - CONNECTIONSTRINGS__INDEX=Server=cadmus-PRJ-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
      - SERILOG__CONNECTIONSTRING=mongodb://cadmus-PRJ-mongo:27017/{0}-log
      - STOCKUSERS__0__PASSWORD=...
      - JWT__SECUREKEY=...
      - ALLOWEDORIGINS__0=https://cadmus-PRJ.YOURDOMAIN
      - SEED__DELAY=25
      - SEED__ITEMCOUNT=5
      - MESSAGING__APIROOTURL=https://cadmus-PRJ-api.YOURDOMAIN/
      - MESSAGING__APPROOTURL=http://cadmus-PRJ.YOURDOMAIN/
      - MESSAGING__SUPPORTEMAIL=...
      - VIRTUAL_HOST=cadmus-PRJ-api.YOURDOMAIN
      - LETSENCRYPT_HOST=cadmus-PRJ-api.YOURDOMAIN
      - LETSENCRYPT_EMAIL=...
    networks:
      - nginx-proxy

  # Cadmus PRJ App
  cadmus-app:
    image: vedph2020/cadmus-PRJ-app:0.0.1
    container_name: cadmus-PRJ-app
    ports:
      - 4200:80
    depends_on:
      - cadmus-PRJ-api
    environment:
      - VIRTUAL_HOST=cadmus-PRJ.YOURDOMAIN
      - LETSENCRYPT_HOST=cadmus-PRJ.YOURDOMAIN
      - LETSENCRYPT_EMAIL=...
    volumes:
      - /dockers/cadmus-PRJ/env.js:/usr/share/nginx/html/env.js
    networks:
      - nginx-proxy

volumes:
  mongo-vol:
  pgsql-vol:

networks:
  nginx-proxy:
    external: true
```

### Defining A-Records

In the context of Docker and NGINX reverse proxy, the `VIRTUAL_HOST` environment variable in your Docker Compose configuration is used to route requests to the correct container based on the hostname in the HTTP request. This means that if a request comes in for `service1.your-domain.com`, it will be routed to the `service1` container, and if a request comes in for `service2.your-domain.com`, it will be routed to the `service2` container.

This works well for local testing and development, but for it to work on the public internet, you need to set up DNS records for each subdomain, pointing to the public IP address of your server. This is because when someone tries to visit `service1.your-domain.com`, their browser needs to know which server to send the request to, and it gets this information from the DNS records for that subdomain.

So, in summary, while you can freely define subdomains in your Docker Compose configuration, you would still need to set up corresponding DNS records for those subdomains to be accessible over the internet.

#### Configuring DNS

To setup the DNS A records, you must use the dashboard provided by your service.

For instance, if you registered your domain with Namecheap, login there and open the dashboard to manage your domain like `my-domain.com` (under Advanced DNS) and add the subdomain portion only as an A record pointing to your server VM IP, e.g.:

- A record: host=`service-subdomain` (without `.my-domain.com`), value=your VM IP, TTL=automatic.

If you use your VM provider also as your registrar, e.g. with SSDNodes or similar services, you would use their dashboard or control panel to define A records. In SSDNodes you will find a DNS zone corresponding to the registered domain, and you will add A-records there. For example, if your server’s public IP address is 203.0.113.0, and you want to create a subdomain `service1.your-domain.com`, you would create an A record that points `service1.your-domain.com` to 203.0.113.0. This way, when someone visits `service1.your-domain.com`, their request is directed to your server at 203.0.113.0. Your NGINX reverse proxy then routes the request to the correct Docker service based on the `VIRTUAL_HOST` environment variable.

>Note that DNS takes time to propagate. You can use an online DNS propagation checker like <https://www.whatsmydns.net/> to check if the DNS has propagated worldwide. Once you see that DNS has propagated, just do `docker compose down` (add `-v` if you want to remove also volumes) and then `up` for your NGINX proxy script.

You can check your DNS propagation with `nslookup` in Ubuntu, e.g.:

```bash
nslookup YOURDOMAIN
nslookup cadmus-PRJ.YOURDOMAIN
nslookup cadmus-PRJ-api.YOURDOMAIN
nslookup cadmus-biblio-api.YOURDOMAIN
```

`nslookup` is part of the `dnsutils` package. You can install it using the following command:

```bash
sudo apt-get update
sudo apt-get install dnsutils
```

## (B) Alternative Method

To use HTTPS, you need to make some changes to the previous configuration to use a [certificate](https://en.wikipedia.org/wiki/Public_key_certificate) and change the URIs protocol from HTTP to HTTPS. Of course, there are several other methods, but the one illustrated here just continues the above configuration.

### Getting a Certificate

First you need a **certificate**. Get it from somewhere (e.g. <https://letsencrypt.org>), and prepare it in these formats:

- CER with KEY.
- PFX: you can convert a CER (or other format) certificate with its key file into a PFX file using the Linux `openssl` tool, like this:

```bash
openssl pkcs12 -export -out certificate.pfx -inkey <NAME>.key -in <NAME>.cer
```

Here your mileage may vary according to the format you get for the certificate. See the [openssl manual](https://www.openssl.org/docs/manmaster/man1/) for more, e.g. to convert PEM into CRT and KEY:

```bash
openssl x509 -outform der -in fullchain.pem -out docker_it.crt
openssl rsa -outform der -in privkey.pem -out docker_it.key
```

Then, place the 3 files under some folder in your host, e.g.:

- my CER and KEY files are under `opt/cadmus/cert`.
- my PFX file is under `opt/cadmus/nginx`.

This is required because we are going to share these certificates with the services running inside the Docker containers via a Docker volume. Using a volume is the suggested way of making a certificate available to what's inside a Docker image, without having to build an image with the certificate inside it, which would not be secure, nor practical.

### Passing Certificates to the API

- reference: [ASP.NET CORE SSL in Docker](https://docs.microsoft.com/en-us/aspnet/core/security/docker-compose-https).

The next step is passing our certificates to the API service inside the Cadmus Docker stack.

>As an ASP.NET API, the Cadmus API service uses [Kestrel](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel).

(1) in the API service of the Docker compose script add a **volume** to share the PFX certificate file:

```yml
volumes:
  - /opt/cadmus/cert/certificate.pfx:/app/Infrastructure/Certificate/certificate.pfx
```

This instruction means that the file `certificate.pfx` under folder `/opt/cadmus/cert` in our host machine will be shared via a Docker volume pointing to file `/app/Infrastructure/Certificate/certificate.pfx` in the Docker image file system. This location is where Kestrel (the web server used by Cadmus API) will be instructed to find the certificate.

(2) continuing in the Docker compose script, add these environment variables in the same API service section of the Docker script, to tell Kestrel about the certificate:

```yml
environment:
  - ASPNETCORE_URLS=https://+:443;http://+:80
  - ASPNETCORE_Kestrel__Certificates__Default__Path=/app/Infrastructure/Certificate/certificate.pfx
  - ASPNETCORE_Kestrel__Certificates__Default__Password=YOURCERTPASSWORD
```

Of course, replace `YOURCERTPASSWORD` with the password used for your certificate.

These variables tell Kestrel that the HTTPS port is 443, the HTTP port is 80, the certificate is found at the specified path, and its password is that specified.

>💥 Mind the number of underscore (`_`) characters in these variable names! Please note that `ASPNETCORE_` ends with 1 underscore, whereas all the other underscores in the variable name come with 2 of them.

(3) still in this `environment` section, be sure to add an HTTPS (rather than HTTP) allowed origin for CORS, e.g.:

```yml
  - ALLOWEDORIGINS__0=https://docker.somewhere.it
```

>The suffix `__0` here is just the index in the array of allowed origins. So here it happens to be `0`, because it's the first origin I put in my list. Should you have more entries, change this index accordingly, remembering that it's 0-based (so the first origin you add is `ALLOWEDORIGINS__0`, the second `ALLOWEDORIGINS__1`, etc.).

This is enough for the API service: all what we did was sharing our PFX certificate file with Kestrel running in the Docker container, and telling it where to find this certificate.

Also, given that our frontend web app will be served in HTTPS, we ensured that its URI is among the allowed origins for this API.

### Passing Certificates to the Frontend

The frontend app uses [NGINX](https://www.nginx.com/) to serve the web app. You can find the default NGINX configuration for it among the source files of the Cadmus web app, in its GitHub repository.

(1) rather than modifying this configuration and rebuild the image, we're going to replace it with a new one, using our CER and KEY certificates. In the new NGINX configuration, the `server` section is replaced with this one:

```nginx
server {
  listen 443 ssl;
  listen 80;
  server_name localhost;
  ssl_certificate docker_it.cer;
  ssl_certificate_key docker_it.key;

  gzip on;
  gzip_min_length 1000;
  gzip_proxied expired no-cache no-store private auth;
  gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;

  location / {
    root /usr/share/nginx/html;
    index index.html index.htm;
    try_files $uri $uri/ /index.html;
  }

  error_page 500 502 503 504 /50x.html;
  location = /50x.html {
    root /usr/share/nginx/html;
  }
}
```

The relevant changes are:

- `listen 443 ssl` for HTTPS.
- the `ssl_certificate` lines to define the CER and KEY certificates to be used.

The rest of the section is unchanged. Save this new `nginx.conf` configuration file somewhere in your host, e.g. in `/opt/cadmus/nginx`.

(2) in the Docker compose script, head to the `cadmus-app` service section, and make these changes:

- ensure you have changed the image name to reference your production version for the app. The default image name in the GitHub source refers to the development version, which targets API in localhost. Your production version should change the `env.js` file so that API are located at their HTTPS URIs.
- ensure that both ports 80 and 443 are exposed.
- add volumes to pass the NGINX configuration and the CER and KEY files to the Docker container.

Here is a sample:

```yml
cadmus-app:
  restart: always
  image: vedph2020/cadmus-app:0.0.10-prod
  ports:
    - 80:80
    - 443:443
  depends_on:
    - cadmus-api
    - cadmus-db
  networks:
    - cadmus-network
  volumes:
    # https://www.techrepublic.com/article/how-to-enable-ssl-on-nginx/
    # overwrite nginx.conf to use SSL with certificates from the host
    - /opt/cadmus/nginx/nginx.conf:/etc/nginx/nginx.conf
    - /opt/cadmus/nginx/docker_it.cer:/etc/nginx/docker_it.cer
    - /opt/cadmus/nginx/docker_it.key:/etc/nginx/docker_it.key
```
