# Index

- [ssh](#ssh)
- [fedora](#fedora)

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

china mirror
```sh
sed -e 's|^metalink=|#metalink=|g' \
    -e 's|^#baseurl=http://download.example/pub/fedora/linux|baseurl=https://mirrors.tuna.tsinghua.edu.cn/fedora|g' \
    -i.bak \
    /etc/yum.repos.d/fedora.repo \
    /etc/yum.repos.d/fedora-updates.repo
```

Download Edge browser [https://packages.microsoft.com/yumrepos/edge/Packages/m/microsoft-edge-stable-135.0.3179.73-1.x86_64.rpm](https://packages.microsoft.com/yumrepos/edge/Packages/m/microsoft-edge-stable-135.0.3179.73-1.x86_64.rpm)

Download VS Code rpm [https://packages.microsoft.com/yumrepos/vscode/Packages/c/code-1.99.2-1744250112.el8.x86_64.rpm](https://packages.microsoft.com/yumrepos/vscode/Packages/c/code-1.99.2-1744250112.el8.x86_64.rpm)
