---
layout: Linux杂记
title: Linux杂记
date: 2022-10-8 02:20:53
categories:
  - - 折腾记录
tags: 
  - linux
---
### KDE

- KDE重启

    `kquitapp5 plasmashell && kstart plasmashell`

- KVM

    

- Minicom

    ```bash
    yay -S minicom
    sudo minicom -s
    ```

    ```
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

    select "Serial port setup"

    check device : ``` sudo dmesg | grep tty ``` or ``` ls -l /dev/tty* ```

     ```
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