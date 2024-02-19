---
authors:
  - Burak Berk
title: Creating Fullchain SSL Certificate With Crtforge
description: Become a local CA and handle certificates with 3 commands.
date: 2023-12-21
tags:
  - 'ssl'
  - 'ca'
  - 'self-signed'
categories:
  - 'DevOps'
  - 'System Administration'
---

## Overview

I wrote about how to create a fullchain ssl certificate manually in a [previous post](https://burakberk.dev/posts/fullchain-ssl-certificate-with-bash/). Today, I will show you how you can create fullchain ssl cert with only 3 commands.

We will use a tool named crtforge which is free and open-source application that you can find on [Github](https://github.com/burakberkkeskin/crtforge).

## Install `crtforge`

To install `crtforge`, you need Linux or macOS. You can also use the `crtforge` on Windows with wsl.

```bash
sudo curl -L -o /usr/bin/crtforge https://github.com/burakberkkeskin/crtForge/releases/latest/download/crtforge-$(uname -s)-$(uname -m) && \
sudo chmod +x /usr/bin/crtforge
```

These two command will install crtforge binary to your system.

## Create a fullchain cert.

You can directly run `crtforge` with a appname and domains that app needs.

```bash
crtforge websiteApp myapp.com app.myapp.com
```

Congratulations 🎉, you will have your fullchain with 3 commands in total.

You can get the certs from the `$HOME/.config/crtforge/default/websiteApp` directory.

For detailed usage like custom root CA, custom intermediate CA and trusting the root CAs, you can check the readme of the [repository](https://github.com/burakberkkeskin/crtForge).
