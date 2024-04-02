---
title: electron-media-compressor
date: 2024-03-31 10:48:30 +0530
categories: [NodeJS, Electron]
tags: [desktop, app, nodejs, reactjs]     # TAG names should always be lowercase
# mermaid: true
pin: true
---

## Download
[CompressMe](https://github.com/bangeras/electron-media-compressor/releases){:target="_blank"}{:rel="noopener noreferrer"}

## Overview
A lightweight, desktop app built with Electron to compress media files (images, videos, etc). Multiple media files can be selected & compressed.
Output files are currently stored in the local filesystem and the user can view the content via the app.

The application utilizes native desktop OS capabilities (file dialog, native file viewer). The frontend of the app is built using ReactJS. The main process handling is performed using NodeJS

![Desktop View](/assets/img/electron-media-compressor/app.png){: width="700" height="400" }

## Architecture
![App flow](/assets/img/electron-media-compressor/architecture.png){: width="1000" height="500" }

## Packaging & Distribution
Electron Forge is the all-in-one tool used for packaging and distributing across multiple platforms.

### Packaging
```shell
"scripts": {
    "start": "electron-forge start",
    "package": "electron-forge package",
    "package-win": "electron-forge package --platform=win32",
    "make": "electron-forge make",
    "make-win": "electron-forge make --platform=win32",
    "publish": "electron-forge publish",
    "publish-win": "electron-forge publish --platform=win32"
}
```
{: file="package.json" }

```shell
$ npm run make
```

### Publishing

```shell
$ npm run publish
```


## Electron Showcase
https://www.electronjs.org/apps
