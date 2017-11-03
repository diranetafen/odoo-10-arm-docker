
[![Docker Stars](https://img.shields.io/docker/stars/fiercely/odoo-10-arm-docker.svg)]

[![Docker Pulls](https://img.shields.io/docker/pulls/fiercely/odoo-10-arm-docker.svg)]

# Latest fixes
Latest fix ( September 1st 2017) :
Fix "Could not execute command 'lessc'" for missing dependencies

# odoo-10-arm-docker

This repository contains the dockerfile required to build an odoo v.10 image for use in ARM devices, tested in raspberry pi 3.

The image built is based on Alpine image and uses armhf Alpine repositories.

**X86 devices** should refer to the Dockerfile-x86 . 

**Note**: The image built uses [Testing Repo](https://wiki.alpinelinux.org/wiki/Aports_what_is_edge) for the [wkhtmltopdf](https://pkgs.alpinelinux.org/package/edge/testing/x86/wkhtmltopdf) package. This means that there is no support for this package yet, and errors might be present. This package is required by odoo. This is valid for both ARM and X86. 

## How to build odoo for arm from this dockerfile

```
docker build -f /path/to/this/Dockerfile . 
```
This might take a long time on your first build, depending on a few conditionals, the write speed of your RPI sd card, and network.

Current Dockerfile defaults to version 10.0.20170101 from Odoo source repository
#
## Building a new image from a newer odoo source

[Odoo source code daily builds](https://nightly.odoo.com/10.0/nightly/src/)

After selecting the odoo build version you wish to use:  **in tar.gz format**

Replace the default dockerfile for you selected version.

Example:
```
Dockerfile line 27:    && wget https://nightly.odoo.com/10.0/nightly/src/odoo_10.0.20170101.tar.gz \
Dockerfile line 28:    && tar -xzf odoo_10.0.20170101.tar.gz -C /opt \
Dockerfile line 29:    && rm odoo_10.0.20170101.tar.gz \
Dockerfile line 30:    && cd /opt/odoo-10.0-20170101 \
Dockerfile line 33:    && rm -r /opt/odoo-10.0-20170101


Replace with latest for instance:

Dockerfile line 27:    && wget https://nightly.odoo.com/10.0/nightly/src/odoo_10.0.latest.tar.gz \
Dockerfile line 28:    && tar -xzf odoo_10.0.latest.tar.gz -C /opt \
Dockerfile line 29:    && rm odoo_10.0.latest.tar.gz \
Dockerfile line 30:    && cd /opt/odoo_10.0.latest \
Dockerfile line 33:    && rm -r /opt/odoo_10.0.latest
```
And change odoo.conf file to your new odoo directory.
```
[options]
addons_path = /mnt/extra-addons,/usr/lib/python2.7/site-packages/odoo-10.0.post20170101-py2.7.egg/odoo/addons
```
to

```
[options]
addons_path = /mnt/extra-addons,/usr/lib/python2.7/site-packages/odoo-10.0.postlatest-py2.7.egg/odoo/addons
```

Build your new image.
#

## How to use this image 
This image has the same usage as the ofiicial Odoo.


This image requires a running PostgreSQL server. Be aware that for the ARM version, you should run an PostgreSQL image for arm and version 9.4 and above. (i.e. e1ee1e11/postgresql_armhf:9.4.8-1)
#
### Start a PostgreSQL server
```
docker run -d -e POSTGRES_USER=odoo -e POSTGRES_PASSWORD=odoo --name db e1ee1e11/postgresql_armhf:9.4.8-1
```

Change db image  e1ee1e11/postgresql_armhf:9.4.8-1 for whichever image you might use for postgres in arm
#
### Start an Odoo instance
```
docker run -p 8069:8069 --name odoo --link db:db -t odoo
 ```
Change -t odoo for the name of the odoo arm image you built previously.

The alias of the container running Postgres must be db for Odoo to be able to connect to the Postgres server.
#

### Run Odoo with a custom configuration

The default configuration file for the server (located at /etc/odoo/openerp-server.conf) can be overriden at startup using volumes. Suppose you have a custom configuration at /path/to/config/openerp-server.conf, then

```
docker run -v /path/to/config:/etc/odoo -p 8069:8069 --name odoo --link db:db -t odoo
 ```
 Change -t odoo for the name of the odoo arm image you built previously.

#
### Mount custom addons
You can mount your own Odoo addons within the Odoo container, at /mnt/extra-addons
```
docker run -v /path/to/addons:/mnt/extra-addons -p 8069:8069 --name odoo --link db:db -t odoo
```
 Change -t odoo for the name of the odoo arm image you built previously.

#
### Using with Docker Compose 

### ARM example
```
version: '2'
services:
  web:
    build: 
      context: .
      dockerfile: Dockerfile
      - db
    ports:
      - "8069:8069"
  #postgres database with persisting volume for data, so that when container is destroyed data is persisted
  db:
    image: e1ee1e11/postgresql_armhf:9.4.8-1
    environment:
    - POSTGRES_USER=odoo
    - POSTGRES_PASSWORD=myodoo
    volumes:
    - db-data:/var/lib/postgresql/data
```
#
### X86 example
```
  web:
    build: 
      context: .
      dockerfile: Dockerfile-x86
    depends_on:
      - db
    ports:
      - "8069:8069"
    environment:
    - HOST=db
    - USER=odoo
    - PASSWORD=myodoo
    # passing custom addons to odoo.
    volumes: 
    - ./addons:/mnt/extra-addons
    
  #postgres database using official postgres image, with persisting volume for data, so that when container is destroyed data is persisted
  db:
    image: postgres:9.4.3
    environment:
    - POSTGRES_USER=odoo
    - POSTGRES_PASSWORD=myodoo
    volumes:
    - db-data:/var/lib/postgresql/data
```    
  
