# nginx
This is an nginx reverse proxy designed to front microservices with valid HTTPS certificates for free, using [letsencrypt](https://letsencrypt.org/).

## Automatic certificate generation
When the container boots, if no certificates are found, it will do the following:

  - First create a self signed certificate for the domain in question (so we can start nginx, and letsencrypt can do it's host checks).
  - Use [simp_le](https://github.com/zenhack/simp_le) to generate, or update the letsencrypt certificates for the domain.

It's important that the letsencrypt servers can contact your selected domain in order to do validation, and this container is running on the server that hosts that doman.  Basically, this is the flow of events:
```
Container Boots
  -> Self Signed certificates generated for given domain(s)
  -> Triggers LetsEncrypt for given domain(s)
  -> LetsEncrypt servers try and talk to yourdomain.net/.well-known/<secret>
  -> LetsEncrypt servers return certificate
  -> Self signed replaced with LetsEncrypt certs
  -> Container restarts NGINX
```

## Multiple domains, no configuration
You can host multiple domains on the same NGINX:443 host (see the example below).

The default server part is important, as we're hosting multiple SSL certificates on the same IP, Nginx will use [SNI](https://en.wikipedia.org/wiki/Server_Name_Indication) to serve up the relevant endpoint.  If the client doesn't support SNI (for example, my curl client on macosx?!) then you'll get the default server.

```
version: '2'

services:
  nginx:
    image: stono/docker-nginx-letsencrypt
    restart: always
	volumes:
	  - ./certs:/etc/letsencrypt/live
    environment:
      - HOST_WEBSITE1=website1.yourdomain.net,localhost:9000,default_server
      - HOST_WEBSITE2=website2.yourdomain.net,localhost:9001
      - LETSENCRYPT_EMAIL=youremail@yourdomain.com
      - LETSENCRYPT=true
    ports:
      - 443
      - 80
```

## Redirecting
You can redirect domains too, for example - www.karlstoney.com to karlstoney.com by using `redirect` at the end of your host declaration:

```
version: '2'

services:
  nginx:
    image: stono/docker-nginx-letsencrypt
    restart: always
	volumes:
	  - ./certs:/etc/letsencrypt/live
    environment:
      - HOST_WEBSITE1=website1.yourdomain.net,localhost:9000,default_server
      - HOST_WEBSITE2=website2.yourdomain.net,website1.yourdomain.net,redirect
      - LETSENCRYPT_EMAIL=youremail@yourdomain.com
      - LETSENCRYPT=true
    ports:
      - 443
      - 80
```

## Volumes
You need to persist your certificates, so mount the `/etc/letsencrypt` folder!
