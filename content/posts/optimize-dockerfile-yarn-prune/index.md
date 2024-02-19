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

We all have at least one nodejs application that is dockerized. Typically, we search `nodejs dockerfile yarn` and add the first result to our repository.

But how effective are these dockerfiles?

In this tutorial, I will give a example dockerfile for server side nodejs applications.

In this tutorial, we'll discuss how to improve Node.js programs that use the yarn package manager.

## Background of problem

Normally, `npm install` and `yarn install` commands install all dependencies listed in package.json into the node_modules folder. This includes "dependencies" as well as "devDependencies".

However, once you've finished developing and building the application, you no longer require dev dependencies.

So npm handles that by `prune` command. The command `npm prune` goes through node_modules and removes any packages not listed as dependencies in package.json.

Also the --production flag in the `npm prune --production`command tells prune to only remove packages not in "dependencies"

After all, you don't have unnecessary dev and test packages in your production ready docker image.

The reduced size of the optimized image not only improves the overall performance but also minimizes the storage requirements, making it more efficient for deployment and distribution. Additionally, by addressing the vulnerabilities in the not-optimized image, the optimized version enhances security and reduces potential risks for cyberattacks.

## The problem

If you decided to use yarn as it is faster than npm on installing packages, but also want to use prune command, you will notice that yarn package manager has no equivalent command for the "npm prune --production". But we have a trick for that.

## Optimized Dockerfile

First, lets see the full Dockerfile. Then I will explain it.

```Dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json yarn.lock ./
RUN yarn install --frozen-lockfile
COPY . .
RUN yarn run build
RUN yarn install --production --ignore-scripts --prefer-offline --frozen-lockfile

FROM node:18-alpine AS runner
RUN apk add --no-cache tini
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/build ./
COPY --from=builder /app/package.json ./
ENTRYPOINT [ "/sbin/tini", "--" ]
CMD [ "node", "build/server.js" ]
```

### Let's break this dockerfile into pieces and try to understand it

#### Base Image

> FROM node:18-alpine AS builder

You should select alpine version so your builded image will be smaller than normal one.

#### Building Application

> COPY package.json yarn.lock ./
> RUN yarn install --frozen-lockfile
> COPY . .
> RUN yarn run build

You are building the image here. The key here is that using cache mechanism of docker while installing npm modules. Unless you don't change the package.json, you don't have to re-install all npm modules again and again.

#### Pruning Dev Dependencies After Build 👑

> RUN yarn install --production --ignore-scripts --prefer-offline --frozen-lockfile

This command will prune the dev dependencies after the building the image. You have already seen the advantages of removing dev-dependencies.

#### We do not reinstall dependencies 👑

As you can see from the dockerfile, there is no second installation for node modules on the runner stage. This shortens the build time.

#### Using tini

Tini is an additional method for optimizing Node.js apps. Node is not an excellent tool for dealing with system signals or zombie processes. For example, if you press control + c, the node program will not stop since node lacks a single handling method. You may solve these issues by using tini (the inverse of init).

## Conclusion

Let's look at some of the benefits mentioned above.
The docker image I'm comparing is the same as the optimized one, but without the 'yarn prune' command.

### Security

As a part of conclusion, let's check get the security check results of optimized image and not optimized image:

Optimized image has only 10 vulnerability. 4 medium, 6 high.

![Optimized Image Security Scan](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/csryi8b88gafof6dcjk8.png)

Not-optimized image has 33 vulnerability! 13 medium and 20 high.

![Not Optimized Image Security Scan](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fnvzthaxc8lw4396wjge.png)

### Image Size

Also the optimized image's size is less than half of the unoptimized image. Let's check it out:

![Docker Image Size Comparison](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lcrozqk49tjfx3zdiw78.png)
