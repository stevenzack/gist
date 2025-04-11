jWM Setup

- [Debian netinst](#debian-netinst)
- [Power consumption](#power-consumption)

---

# Debian netinst

```sh
sudo apt install xorg jwm xfce4-terminal gdebi vim git thunar fonts-noto-cjk pulseaudio pavucontrol sudo
```

/etc/X11/xorg.conf.d/40-libinput.conf
```
Section "InputClass"
        Identifier "libinput touchpad catchall"
        MatchIsTouchpad "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
        Option "Tapping" "on"
EndSection
```

/etc/X11/xorg.conf.d/30-touchpad.conf
```
Section "InputClass"
        Identifier "libinput touchpad catchall"
        MatchIsTouchpad "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
        Option "NaturalScrolling" "true"
        Option "Tapping" "on"
EndSection
```
$HOME/.jwmrc
[Reference](https://notabug.org/adnan360/jwm-config)
```xml
<?xml version="1.0"?>
<JWM>
        <Key mask="4" key="W">exec:google-chrome-stable</Key>
        <Key mask="4" key="E">exec:thunar</Key>
        <Key mask="CA" key="T">exec:xfce4-terminal</Key>
        <Key mask="CA" key="C">exec:code</Key>

        <Key mask="A" key="Tab">nextstacked</Key>
        <Key mask="A" key="F4">close</Key>

        <Key mask="A" key="F10">maximize</Key>

        <Key mask="CA" key="Right">rdesktop</Key>
        <Key mask="4" key="Tab">rdesktop</Key>
        <Key mask="CA" key="Left">ldesktop</Key>
        <Key mask="CA" key="Up">udesktop</Key>
        <Key mask="CA" key="Down">ddesktop</Key>
        <Key mask="CA" key="Delete">exec:sudo reboot</Key>

        <MoveMode>opaque</MoveMode>
        <ResizeMode>opaque</ResizeMode>
        <FocusModel>click</FocusModel>


        <!-- Virtual Desktops -->
        <Desktops>
                <Background type="image">/home/asd/Pictures/wallpapers/montain.png</Background>
        </Desktops>

</JWM>
```

# sudoers
/etc/sudoers
```
asd ALL=(ALL:ALL) ALL
```

# Monitor Display Settings
[Link](https://www.baeldung.com/linux/monitor-brightness-change)

# Brightness
```sh
brightnessctl set 80%
```

# Volume Control
```sh
pavucontrol
```

# Power Consumption

```sh
n1=$(cat /sys/class/power_supply/BAT0/charge_now) n2=$(cat /sys/class/power_supply/BAT0/charge_full) && echo $(expr $n1 \* 100 / $n2) %
```
