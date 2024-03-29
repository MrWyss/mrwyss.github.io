---
layout: post
title: 'Tip #16: Fun with Tailscale''s Funnel'
date: 2023-04-22 11:29 +0100
description: 
image: 
category:
- tips
- linux
tags: linux docker tailscale self-hosting
mermaid: false
---

## Intro

_**Tailscale Funnel** is a secure and easy-to-use tool that allows you to share your resources with others over the internet without the need for complex setup or configuration (no router port forwarding). It acts as a reverse proxy, similar to **Cloudflare Tunnel**, and features **public DNS** and **TLS termination**. This makes it a great option for quick self-hosting resources. Some use cases include:_

- _Showcasing your development website to a co-worker, client or anyone on the internet_
- _Exposing self-hosted docker images to the internet_
- _Triggering webhooks on your dev device_
- _Self-hosting a file or directory, see the idea I have at the end[^1]_
- _etc._

In the next few steps, I will show you how to get started with **Tailscale Funnel**. I guess, the guide will work mostly on any Device (Linux, Windows or macOS), but I have only tested it on a Raspberry Pi 400 with Raspberry Pi OS. The Tailscale client version was 1.38.4.

## Prerequisites

- [Tailscale](https://tailscale.com/) account (Free)

## Tailscale Admin Config

Before you begin, there are a few things that need to be activated on the [Tailscale Admin Portal](https://login.tailscale.com/admin/)

### In the DNS Settings

Enable **Magic DNS**

![Enable Magic DNS](/assets/img/tip-16/tailscale-magicdns.png)_Enable Magic DNS_

Enable **HTTPS Certificates**

![Enable HTTPS Certificates](/assets/img/tip-16/tailscale-certificates.png)_Enable HTTPS Certificates_

### In the Access Controls

Add **Funnel to policy**

![Add Funnel to policy](/assets/img/tip-16/tailscale-addfunnelpolicy.png)_Add Funnel to policy_

That should be it for the Tailscale admin settings.

## Install Tailscale

Tailscale can be installed with this one-liner:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```
{: .nolineno }

Start Tailscale:

```bash
sudo tailscale up
```
{: .nolineno }

Authenticate:

```bash
To authenticate, visit:

        https://login.tailscale.com/a/XXXXXXXXXXXXX
```
{:.nolineno }

> follow the instructions in the browser
{: .prompt-info }

## Turn Funnel On

Turing on funnel for HTTPS:

```bash
sudo tailscale funnel 443 on
```
{: .nolineno }

But we are **not serving** anything yet.

Check the status:

```bash
sudo tailscale serve status
```
{: .nolineno }

## Serve a Docker Image (whoami) on yourhost.tailnet-name.ts.net/whoami

Whoami is a Tiny Go web server that prints OS information and HTTP request to output.
Make sure you have docker installed. There are tons of [guides](https://sl.bing.net/eA6XogYFkke) out there.

### Run Docker Image

Start `whoami` docker image:

```bash
docker run -d --name funnel-whoami -p 8001:80 containous/whoami
```
{: .nolineno }

Check if it exposes `whoami` on `http://172.0.0.1:8001` on your device.

### Funnel/Serve the Docker Image

Now, as we have it locally exposed, we would like to expose it to the scary Internet on `yourhost.tailnet-name.ts.net/whoami`.

Serve `http://127.0.0.1:8001` on `yourhost.tailnet-name.ts.net/whoami`:

```bash
sudo tailscale serve https /whoami http://127.0.0.1:8001 on
```
{: .nolineno }

Check the funnel/serve status:

```bash
sudo tailscale serve status
```
{: .nolineno }

Now you should see something like this:

```text
# Funnel on:
#     - https://yourhost.tailnet-name.ts.net

https://yourhost.tailnet-name.ts.net (Funnel on)
|-- /whoami proxy http://127.0.0.1:8001
```
{: .nolineno }

Check the url + path in your browser (<https://yourhost.tailnet-name.ts.net/whoami>). It's not on publicly available and should even be TLS terminated.

Stop serving:

```bash
sudo tailscale serve https /whoami http://127.0.0.1:8001 off
```
{: .nolineno }

## Expose a static html site on yourhost.tailnet-name.ts.net

Serve a static html file on the scary internet is super simple.

Create an example [index.html](https://www.w3schools.com/html/tryit.asp?filename=tryhtml_basic_document) and save it locally e.g. ``/home/pi/tailscale-funnel/index.html``

Serve `index.html` on `yourhost.tailnet-name.ts.net/`:

```bash
sudo tailscale serve https:443 / /home/pi/tailscale-funnel/index.html on
```
{: .nolineno }

Check the funnel/serve status:

```bash
sudo tailscale serve status
```
{: .nolineno }

Now you should see something like this:

```text
# Funnel on:
#     - https://yourhost.tailnet-name.ts.net

https://yourhost.tailnet-name.ts.net (Funnel on)
|-- /       path  /home/pi/tailscale-funnel/index.html
|-- /whoami proxy http://127.0.0.1:8001
```
{: .nolineno }

browse to (<https://yourhost.tailnet-name.ts.net>). It's now publicly available and should even be TLS terminated.

Stop serving:

```bash
sudo tailscale serve https:443 / /home/pi/tailscale-funnel/index.html off
```
{: .nolineno }

## Conclusion

Once the prerequisites are installed and Tailscale is configured. It's as simple as running:

- Static Website: ``sudo tailscale serve https:443 / /home/pi/tailscale-funnel/index.html on``
- Proxy Port ``sudo tailscale serve https /anything http://127.0.0.1:8001 on``
- Directory listing ``sudo tailscale serve https /dir /home/pi/tailscale-funnel/dir on``
- Single file ``sudo tailscale serve https /notes.txt /home/pi/tailscale-funnel/notes.txt on``
- SSH ``sudo tailscale serve tcp:2222 tcp://localhost:22 on`` this will only be on the tailnet not public

> As of the time of this writing Tailscale Funnel is still beta.
{: .prompt-warning }

For my long-term self-hosting needs, I prefer using Cloudflare Tunnels. They can be set up with authentication and are managed on the backend. However, Tailscale Funnels can also be fun.

[^1]: For instance, I could create a self-hosted Pastebin by serving a file named ``pastebin.txt``. This file could be used to pipe any text into it, such as ``cowsay fun with funnels > pastebin.txt``
