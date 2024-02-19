## Overview

This repository contains my personal website and blog page.

## Tech Stack

Made with [Hugo](https://gohugo.io/) and [HugoBlox](https://hugoblox.com/).

## Build

### Preretirements

[Install hugo](https://gohugo.io/installation/).

[Install golang](https://go.dev/doc/install).

### Build the Website

- Clone the repository

```bash
git clone https://github.com/burakberkkeskin/website.git
```

- Build with hugo

```bash
hugo --gc --minify
```

- The builded static html css code is under the `./public` directory.

- You can host it anywhere you want.

- For a quick test, host it with a python web server.

- Change directory to the `public` directory.

```bash
cd public
```

- Serve it with python.

```bash
python3 -m http.server 80
```

- Open chrome and head to the http://127.0.0.1

## Developing

To develop the website, you should run the hugo development server.

```bash
hugo server -p 8080
```

This will watch the file changes and rebuild the application.
