---
authors:
  - Burak Berk
title: Optimize Dockerfile By Yarn Prune
description: Optimize Dockerfile and your container images by equivalent.
date: 2023-08-17
tags:
  - 'docker'
  - 'container'
  - 'nodejs'
  - 'yarn'
  - 'ci-cd'
categories:
  - 'DevOps'
  - 'System Administration'
  - 'Containers'
---

## Overview

In this tutorial, you will see how to test SMTP settings with a tool named `gomtp` on Linux and macOS cli.

`gomtp` is a open-source and free software to test SMTP settings. You can see the source code on the [Github page](https://github.com/safderun/gomtp).

## Install gomtp

To install the `gomtp`, you can simply run the command below:

```bash
sudo curl -L -o /usr/local/bin/gomtp "https://github.com/safderun/gomtp/releases/latest/download/gomtp-$(uname -s)-$(uname -m)" && \
sudo chmod +x /usr/local/bin/gomtp
```

## SMTP configuration.

To test SMTP settings, you first need to configure a yaml file.

- Create a file named `gomtp.yaml`.

```bash
vim gomtp.yaml
```

❗The file name must be `gomtp.yaml` by default.

- Configure the SMTP settings for your needs.

```yaml
##### Gmail Example #####
username: 'from@gmail.com'
password: 'appPassword'
from: 'from@gmail.com'
to: 'to@example.com'
host: 'smtp.gmail.com'
port: 587
ssl: false
tls: true
auth: 'LOGIN'
subject: 'Testing Email'
body: |
  this is line 1
  This is line 2
```

ℹ You can find some other example configurations under [the repository](https://github.com/safderun/gomtp/blob/master/gomtp.yml).

## Test the Configuration

Now it's time to test the configuration.

- Be sure that you are in the same directory with the `gomtp.yaml` file you have just created previously.

- Run the `gomtp` directly with no argument.

```bash
gomtp
```

- If the configuration is valid, you will see this output.

```shell
$ gomtp
Email sent successfully!
```

- If the configuration is invalid, you will see an error message that describe the problem.

```shell
$ gomtp
2023/12/21 11:44:33 535 5.7.8 Error: authentication failed: Invalid user or password! 1703148273-WiJ7lX77XKo0
```
