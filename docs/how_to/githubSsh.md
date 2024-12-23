---
layout: default
title: githubSsh
parent: How To
nav_order: 2
last_modified_date:   2024-12-23 23:16:02
---

# github ssh authentication
1. generate key
    ```bash

    ssh-keygen -t ed25519 -C "your_email@example.com"

    ```
1. add pub key to github: setting -> SSH and GBG keys
1. add config file to .ssh
    ```bash
    Host github.com
        User git
        Hostname github.com
        PreferredAuthentications publickey
        IdentityFile C:\Users\imgri\.ssh\id_ed25519
    ```
1. make sure repository url is ssh
    ```bash
    git remote -v
    ```

