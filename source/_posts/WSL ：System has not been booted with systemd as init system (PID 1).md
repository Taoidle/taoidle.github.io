---
layout: WSL ：System has not been booted with systemd as init system (PID 1)
title: WSL ：System has not been booted with systemd as init system (PID 1)
categories:
  - - 折腾记录
tags: 
  - wsl
  - systemd
date: 2023-01-08 13:01:23
---

## Systemd Error

```shell
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down
```

## Issue

- Release of WSL doesn't support Systemd. 

- Systemd is now officially supported in Ubuntu on many WSL systems.

## Fix

Start Ubuntu (or other Systemd) distribution under WSL

```shell
sudo vim /etc/wsl.conf
```

Add the following:

```shell
[boot]
systemd=true
```

Open Powershell:

```shell
wsl --shutdown
```

or

```shell
wsl --terminate <Distro>
```

WSL List only distributions that are currently running:

```shell
wsl --list --running
```

Start:

Run the specified distribution.

```shell
wsl --distribution <Distro>
```