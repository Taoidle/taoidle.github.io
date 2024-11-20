---
layout: Linux Miscellaneous
title: Linux Miscellaneous
date: 2022-10-9 02:20:53
categories:
  - - 折腾记录
tags: 
  - linux
---
### KDE

- KDE Desktop Restart

    `kquitapp5 plasmashell && kstart plasmashell`

### KVM

1. Install KVM

    ```shell
     pacman -S qemu libvirt ovmf virt-manager virt-viewer dnsmasq vdebridge-utils openbsd-netcat 
     ```

2. Start KVM

    ```shell
    sudo systemctl enable libvirtd.service
    sudo systemctl start libvirtd.service
    ```

3. Use with non root user

    ```shell
    sudo vim /etc/libvirt/libvirtd.conf
    ```

    found and delete '#'

    ```shell
    #unix_sock_group = "libvirt"
    #unix_sock_rw_perms = "0770"
    ```

4. Add new group and restart libvirt service

    ```shell
    newgrp libvirt
    sudo usermod -aG libvirt $USER
    sudo systemctl restart libvirtd.service
    ```

5. Enable Nested Virtualization

    ```shell
    echo "options kvm-intel nested=1" | sudo tee /etc/modprobe.d/kvm-intel.conf
    cat /sys/module/kvm_intel/parameters/nested
    ```

### Doker

- Use with non root user

    ```shell
    sudo usermod -aG docker $USER
    newgrp docker
    ```
- Use on Raspberrypi

    ```shell
    curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh
    sudo groupadd docker
    sudo gpasswd -a $USER docker
    newgrp docker
    ```

- Portainer

    ```shell
    docker volume create portainer_data
    docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
    ```

### PlatformIO

1. Install PlatformIO

    ```shell
    curl -fsSL https://raw.githubusercontent.com/platformio/platformio-core-installer/master/get-platformio.py -o get-platformio.py
    python3 get-platformio.py
    ```

2. Set System Environment

    ```shell
    sudo vim /etc/profile
    ```

    Insert ```append_path '$HOME/.platformio/penv/bin'```

    Save and exit

    ```shell
    source /etc/profile
    ```

### Minicom

```shell
yay -S minicom
sudo minicom -s
```

```shell
+-----[configuration]------+
| Filenames and paths      |
| File transfer protocols  |
| Serial port setup        |
| Modem and dialing        |
| Screen and keyboard      |
| Save setup as dfl        |
| Save setup as..          |
| Exit                     |
| Exit from Minicom        |
+--------------------------+
```

select "Serial port setup" then "Save setup as dfl" or "Save setup as.."

if use "Save setup as..", connect drives like ``` sudo minicom configurename```

check device : ``` sudo dmesg | grep tty ``` or ``` ls -l /dev/tty* ```

```shell
+-----------------------------------------------------------------------+
| A -    Serial Device      : /dev/ttyUSB0                              |
| B - Lockfile Location     : /var/run                                  |
| C -   Callin Program      :                                           |
| D -  Callout Program      :                                           |
| E -    Bps/Par/Bits       : 115200 8N1                                |
| F - Hardware Flow Control : No                                        |
| G - Software Flow Control : No                                        |
| H -     RS485 Enable      : No                                        |
| I -   RS485 Rts On Send   : No                                        |
| J -  RS485 Rts After Send : No                                        |
| K -  RS485 Rx During Tx   : No                                        |
| L -  RS485 Terminate Bus  : No                                        |
| M - RS485 Delay Rts Before: 0                                         |
| N - RS485 Delay Rts After : 0                                         |
|                                                                       |
|    Change which setting?                                              |
+-----------------------------------------------------------------------+
```

select "Save setup as dfl" and "Exit from Minicom"

use ``` sudo minicom ``` connect the serial device

Tips: When use minicom, use Ctrl + A Z to help, Ctrl + A X exit minicom