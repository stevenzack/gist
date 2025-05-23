# Index

- [ssh](#ssh)
- [fedora](#fedora)
- [wireguard](#wireguard)
- [webdav rclone](#webdav-rclone)

# ssh

Github Actions
```yaml
# This workflow will build a golang project
# For more information see: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-go

name: Go

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:

  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Set up Go
      uses: actions/setup-go@v4
      with:
        go-version: '1.22'

    - name: Build
      run: make b
    - name: UPX compression
      uses: crazy-max/ghaction-upx@v3
      with:
        version: latest
        files: |
          ./bin/main
        args: -fq

    - name: SSH
      run: |
        mkdir -p ~/.ssh 
        echo $'Host *\n    StrictHostKeyChecking no' > ~/.ssh/config
        echo "${{ secrets.SSH_PUB }}" >> ~/.ssh/id_rsa.pub
    - uses: webfactory/ssh-agent@v0.9.0
      with:
        ssh-private-key: ${{ secrets.SSH_PRIV }}
    - name: Upload
      run: |
        sftp root@8.220.204.169:/root/apps/memo/ <<< $'put -r *'
    - name: Deploy
      run: ssh root@8.220.204.169 "kill \$(ps -e | grep [m]emo | awk '{print \$1}') &"
    - name: Deploy2
      run: ssh root@8.220.204.169 "(cd /root/apps/memo && mv ./bin/main ./bin/memo && ./bin/memo) &>/dev/null &"
```

Upload 

```sh
sftp root@localhost:/var/www/ <<< $'put -r html'
```

Download
```sh
sftp root@8.220.204.169:/var/www/html/index.html index.html
```

# fedora

Better use Xfce4!!

## china mirror
```sh
sed -e 's|^metalink=|#metalink=|g' \
    -e 's|^#baseurl=http://download.example/pub/fedora/linux|baseurl=https://mirrors.tuna.tsinghua.edu.cn/fedora|g' \
    -i.bak \
    /etc/yum.repos.d/fedora.repo \
    /etc/yum.repos.d/fedora-updates.repo
```

Download Edge browser [https://packages.microsoft.com/yumrepos/edge/Packages/m/microsoft-edge-stable-135.0.3179.73-1.x86_64.rpm](https://packages.microsoft.com/yumrepos/edge/Packages/m/microsoft-edge-stable-135.0.3179.73-1.x86_64.rpm)

Download VS Code rpm （Using wget would be better） [https://packages.microsoft.com/yumrepos/vscode/Packages/c/code-1.99.2-1744250112.el8.x86_64.rpm](https://packages.microsoft.com/yumrepos/vscode/Packages/c/code-1.99.2-1744250112.el8.x86_64.rpm)

## Chinese Input Method

references: [https://blog.csdn.net/codestore/article/details/132255059](https://blog.csdn.net/codestore/article/details/132255059)

```sh
sudo dnf remove -y ibus
sudo dnf install -y fcitx-table fcitx-gtk3 fcitx-table-chinese fcitx fcitx-data fcitx-configtool fcitx-pinyin
```
`vim ~/.bashrc`
```bash
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

`sudo vim /etc/profile.d/fcitx.sh`
```sh
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

now install default fcitx packages
```sh
sudo dnf install -y fcitx-{ui-light,qt{4,5},table,gtk{2,3},table-{extra,other,chinese},configtool}
```

add `fcitx` to your Xfce4 auto start options list.

Start `fcitx` config a pinyin method, and done

# wireguard

On fedora
```sh
sudo dnf update -y
sudo dnf install wireguard-tools -y
sudo modprob wireguard
sudo tee /etc/modules-load.d/wireguard.conf <<< "wireguard"
```
```sh
wg-quick up client
```

# webdav rclone

rclone
```sh
sudo dnf install rclone rclone-fuse
rclone config
```


```sh
mkdir -p ~/.config/systemd/user/
vim ~/.config/systemd/user/rclone-webdav.service
```

`rclone-webdav.service`
```service
[Unit]
Description=Rclone WebDAV Mount (User Mode)
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/rclone mount jianguoyun:/ /home/asd/nutstore \
    --vfs-cache-mode full \
    --allow-other \
    --no-modtime \
    --umask 000 \
    --dir-cache-time 5m
ExecStop=/bin/fusermount -u /home/asd/nutstore
Restart=on-failure
RestartSec=10

[Install]
WantedBy=default.target
```

```sh
systemctl --user daemon-reload
systemctl --user enable rclone-webdav.service
systemctl --user start rclone-webdav.service
```
