---
layout: use gpg&smart card
title: use gpg&smart card
date: 2022-06-23 21:17:40
categories:
  - - 折腾记录
tags: 
  - gpg
  - yubikey
  - canokeys
---

## Import Key

```shell
gpg --import <key_path>
```
![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/gpg_import(1).png)

## List Keys

### list keys
```shell
gpg -k
```

### list secret keys
```shell
gpg -K
```

![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/gpg_import(2).png)


## Change The Ownertrust

```shell
gpg --edit-key <id>
```

```shell
gpg> trust

5

y

gpg> quit
```

![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/gpg_import(3).png)


## Import To Smart Card

```shell
gpg --expert --edit-key <id>
```

### Select key 1 and import to card

```shell
gpg> key 1

gpg> keytocard

2
```

![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/gpg_key2card(1).png)

### Unselect key 1,Select key 2 and import to card

```shell
gpg> key 1

gpg> key 2

gpg> keytocard

1
```

![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/gpg_key2card(2).png)

### Unselect key 2,Select key 3 and import to card

```shell
gpg> key 2

gpg> key 3

gpg> keytocard

3

gpg> quit

y
```

![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/gpg_key2card(3).png)

## Print Card Status

```shell
gpg --card-status
```

![](https://cdn.assets.taoidle.com/gh/taoidle/taoidle.github.io@master/assets/images/gpg_card_status.png)

## Bind Smart Card

```shell
gpg-connect-agent "scd serialno" "learn --force" /bye
```
