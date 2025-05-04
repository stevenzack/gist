# C

- [vscode-setup](#vscode-setup)
- [meson](#meson)

# Vscode Setup

Install extensions:
- C/C++
- Meson

Install your dependencies:
```sh
sudo dnf install -y gcc gtk4-devel
```

Get your libs
```sh
pkg-config --cflags gtk4
```

Copy and paste your output libs into include configurations:

- In VS Code, `Ctrl-Shift P` -> `C/C++: Edit Configurations (UI) `
- Go to the "include path" section, and paste the output libs

```
${workspaceFolder}/**
/usr/include/gtk-4.0
/usr/include/pango-1.0
/usr/include/fribidi
/usr/include/harfbuzz
/usr/include/gdk-pixbuf-2.0
/usr/include/webp
/usr/include/cairo
/usr/include/libxml2
/usr/include/freetype2
/usr/include/libpng16
/usr/include/pixman-1
/usr/include/graphene-1.0
/usr/lib64/graphene-1.0/include
/usr/include/glib-2.0
/usr/lib64/glib-2.0/include
/usr/include/libmount
/usr/include/blkid
/usr/include/sysprof-6
```
 

# Meson

Installation
```sh
sudo dnf install meson -y
```

Create a `meson.build` file in your root directory of your project folder.
```meson
project('gtkdemo', 'c')

executable('main', [
    'src/main.c',
    'src/core/core.c'
],dependencies: [
    dependency('gtk4')
])
```

demo project tree
```sh
.
├── meson.build
└── src
    ├── core
    │   ├── core.c
    │   └── core.h
    └── main.c
```

Setup Meson for Ninja
```sh
meson setup bin
```

Compile
```sh
cd bin/
meson compile # or just run `ninja`
```

Run your executable
```sh
./main
```
