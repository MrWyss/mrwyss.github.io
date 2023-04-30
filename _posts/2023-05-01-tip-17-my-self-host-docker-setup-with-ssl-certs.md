---
layout: post
title: 'Tip #17: my self-host docker setup with ssl certs'
date: 2023-05-01 01:00 +0100
description: 
image: 
category:
- tips
- linux
tags: linux docker self-hosting
mermaid: false
---
## Intro

I run a few docker images at home which are local only. Some examples:

- [your_spotify](https://github.com/Yooooomi/your_spotify)
- [uptime kuma](https://github.com/louislam/uptime-kuma)
- [portainer](https://www.portainer.io/)
- [mailrise](https://github.com/YoRyan/mailrise)

Each of these services has a **DNS name** and present an **SSL certificate**. I use [Traefik](https://traefik.io/) as a reverse proxy and [Let’s Encrypt](https://letsencrypt.org/) for the SSL certificates. I have set up a DNS challenge and an A record that points to my Docker server through [Cloudflare](https://www.cloudflare.com/).

I have put together, in my optionion, a nice setup that is easy to maintain and that works well for me. Let me share that with you in this post.

## One-time setup

The DNS Setup and the reverse Proxy (Traefik) setup. Is a one time setup. For that I assume:

- you have a domain with Cloudflare and know how to get the API key
- you have a server with docker installed

### DNS

I created a subdomain for my public domain called **local**. I use this subdomain to access my services locally. I have set up a wildcard A record that points to my Docker server local IP Address through [Cloudflare](https://dash.cloudflare.com/).

Assuming `192.168.1.2` is the docker server local IP address, the DNS records look like this:

| Type | Name    | Content     | Proxy Status          |
|------|---------|-------------|-----------------------|
| A    | *.local | 192.168.1.2 | DNS only - reseved IP |
| A    | *       | 192.168.1.2 | DNS only - reseved IP |

This means that `local.yourdomain.com` or `anything.local.yourdomain.com` will point to `192.168.1.2`.

### File Folder Structure

I have a folder called **docker-compose** in my home directory. Each service has its own folder with a `docker-compose.yml` file and a `.env` file. The `.env` file contains the environment variables that are used in the `docker-compose.yml` file.

Like so:

```bash
/home
└── docker-compose
    ├── mailrise
    │   ├── docker-compose.yml
    │   └── .env
    ├── portainer
    │   ├── docker-compose.yml
    │   └── .env
    ├── traefik
    │   ├── docker-compose.yml
    │   └── .env
    └── your_spotify
        ├── docker-compose.yml
        └── .env
```
{: .nolineno }

### Reverse proxy with Traefik

I use Traefik to redirect traffic to the right docker container and to manage SSL certificates.

Traefix settings:

```yaml
version: "3.9"

networks:
  public:

services:
  traefik:
    image: "traefik:latest"
    container_name: "traefik"
    restart: always
    command:
      # Globals
      #- "--log.level=DEBUG"
      - "--api=true"
      - "--api.dashboard=true"
      - "--global.sendAnonymousUsage=false"
      # Docker
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--providers.docker.endpoint=unix:///var/run/docker.sock"
      # Entrypoints
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      # LetsEncrypt
      - "--certificatesresolvers.mydnschallenge.acme.email=${LETSENCRYPT_EMAIL}"
      - "--certificatesresolvers.mydnschallenge.acme.dnschallenge.provider=cloudflare"
      - "--certificatesresolvers.mydnschallenge.acme.storage=acme/acme.json"
      - "--certificatesresolvers.mydnschallenge.acme.dnschallenge.delaybeforecheck=0"
      - "--certificatesresolvers.mydnschallenge.acme.dnschallenge.resolvers=1.1.1.1:53,1.0.0.1:53"
      # !IMPORTANT - COMMENT OUT THE FOLLOWING LINE IN PRODUCTION!
      - "--certificatesresolvers.mydnschallenge.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"
    ports:
      - "80:80"
      - "443:443"
    environment:
      - CLOUDFLARE_EMAIL=${CF_API_EMAIL}
      - CLOUDFLARE_API_KEY=${CF_API_KEY}
    volumes:
      - "acme-data:/acme"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    labels:
      # API
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.rule=Host(`local.${DOMAIN}`) && (PathPrefix(`/api`) || PathPrefix(`/dashboard`))"
      - "traefik.http.routers.traefik.service=api@internal"
      - "traefik.http.routers.traefik.entrypoints=web,websecure"
      #- "traefik.http.services.traefik.loadbalancer.server.port=8080"
      #- "traefik.http.services.traefik.loadbalancer.server.scheme=http"
      # Wildcard cert
      - "traefik.http.routers.traefik.tls.domains[0].main=${DOMAIN}"
      - "traefik.http.routers.traefik.tls.domains[0].sans=*.${DOMAIN}"
      - "traefik.http.routers.traefik.tls.certresolver=mydnschallenge"
      # Global http-->https
      - "traefik.http.routers.http-catchall.rule=hostregexp(`{host:[a-z-.]+}`)"
      - "traefik.http.routers.http-catchall.entrypoints=web"
      - "traefik.http.routers.http-catchall.middlewares=redirect-to-https"
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
      # BasicAuth Middleware
      - "traefik.http.middlewares.auth.basicauth.users=${TRAEFIK_USER}:${TRAEFIK_PASSWORD}"
      - "traefik.http.routers.traefik.middlewares=auth"
      # Dashboard
    networks:
      - public
volumes:
  acme-data:
     name: acme-data
```
{: file="docker-compose.yml" }

> Test with the staging server first, then comment out the line: <br> `- "--certificatesresolvers.mydnschallenge.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"`<br> to use the production server instead of the staging server
{: .prompt-info }

The environment variables as follows:

```bash
DOMAIN=yourdomain.com
HOST_NAME=local.yourdomain.com
CF_API_KEY=redacted
CF_API_EMAIL=redacted
LETSENCRYPT_EMAIL=redacted
TRAEFIK_USER=redacted
TRAEFIK_PASSWORD=redacted
```
{: file=".env" : .nolineno }

To start Traefik, run:

```bash
docker-compose --env-file .env up
```
{: .nolineno }

Essentiall it will:

- expose the Traefik dashboard at `local.yourdomain.com/dashboard`, protected by basic auth (TRAEFIK_USER:TRAEFIK_PASSWORD)
- redirect all http traffic to https
- SSL certificates will be generated by Let’s Encrypt
  - wildcard SSL certificate for `yourdomain.com` and `*.yourdomain.com`
  - Cloudflare as a DNS challenge provider

## Per docker image setup

To streamline the process of creating new Docker images with reverse proxy and SSL certificate, I made a **template**:

```yaml
version: "3.9"
networks:
  traefik_public:
    name: traefik_public
services:
  web:
    image: ${SERVICE_NAME}:latest
    restart: always
    container_name: ${SERVICE_NAME}
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.${SERVICE_NAME}.rule=Host(`${SERVICE_NAME}.${SUBDOMAIN}`)"
      - "traefik.http.routers.${SERVICE_NAME}.entrypoints=websecure"
      - "traefik.http.routers.${SERVICE_NAME}.tls.certresolver=mydnschallenge"
    networks:
      - traefik_public
```
{: file="docker-compose.yml" }

The environment variables as follows:

```bash
SERVICE_NAME=yourservicename
SUBDOMAIN=local.yourdomain.com
```
{: file=".env" : .nolineno }

The `SERVICE_NAME` will be used as the **container image**, **container name** and the **host name** of the `SUBDOMAIN` that will be proxied to https.

To start the service, run:

```bash
docker-compose --env-file .env up
```
{: .nolineno }

It will:  

- create a docker container with the yourservicename
- generate a SSL certificates by Let’s Encrypt
- Serve the exposed port of the docker image at `https://yourservicename.local.yourdomain.com`

## Abstract

I have documented this setup for my own reference, but I hope it will be useful for others as well. If you have any questions, feel free to post a comment.
