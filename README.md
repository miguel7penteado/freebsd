# Instalando o FreeBSD

### Versão 13.x 

## VirtualBox

### Instalando pacotes de suporte a virtualização ao servidor xorg

```bash
pkg install xf86-video-vmware xf86-input-vmmouse open-vm-tools
```

### Compilando e instalando o VirtualBox

```bash
cd /usr/ports/emulators/virtualbox-ose-additions && make install clean
```

### Adicionando o seu usuário no grupo `vboxusers`

```bash
pw groupmod vboxusers -m yourusername
```

### Carregando o driver `vboxdrv`

```bash
kldload vboxdrv
```

### Carregando os drivers do virtualbox na inicialização do sistema

```bash
echo 'vboxguest_enable="YES"' >> /etc/rc.conf
echo 'vboxservice_enable="YES"' >> /etc/rc.conf
echo 'vboxservice_flags="--disable-timesync"' >> /etc/rc.conf
```

### Ativando a rede do virtualbox na inicialização do sistema

Carregando o driver de rede do virtualbox
```bash
echo 'vboxdrv_load="YES"' >> /boot/loader.conf
```
Ativando o driver de rede do virtualbox
```bash
echo 'vboxnet_enable="YES"' >> /etc/rc.conf
```

Permissionando corretamente o dispositivo de rede
```bash
chown root:vboxusers /dev/vboxnetctl
chmod 0660 /dev/vboxnetctl
```

Salvando o permissionamento de rede do dispositivo no carregamento do sistema
```bash
echo 'own     vboxnetctl root:vboxusers' >> /etc/devfs.conf
echo 'perm    vboxnetctl 0660'           >> /etc/devfs.conf
```

### Ativando o acesso do virtualbox nos dispositivos USB

Adicionando o seu usuário no grupo `operator`
```bash
pw groupmod operator -m yourusername
```

Salvando o permissionamento do grupo operator para os dispositivos USB
```bash
cat << EOF >> /etc/devfs.rules
[system=10]
add path 'usb/*' mode 0660 group operator
EOF
```

Salvando as regras do devfs na inicialização do sistema
```bash
echo 'devfs_system_ruleset="system"'  >> /etc/rc.conf
```

reinicializando o devfs
```bash
service devfs restart
```

### Ativando o acesso do virtualbox nos dispositivos DVD

```bash
cat << EOF >> /etc/devfs.conf
perm cd* 0660
perm xpt0 0660
perm pass* 0660
EOF
```

```bash
service devfs restart
```


## xorg 

### Instalando o **xorg**

```bash
pkg install xorg
```

```bash
pkg install libva-intel-driver mesa-libs mesa-dri cmrt libva libva-intel-driver libva-intel-hybrid-driver
```



### Adicionando o seu usuário no grupo `video`

```bash
pw groupmod video -m username
```

### Instalando o drm-kmod

```bash
pkg install drm-kmod
```

### Instalando suporte aos drivers intel na carga do sistema

```bash
echo "sysrc kld_list+=i915kms" >> /etc/rc.conf
```


### Arquivo de configuração do xorg: xorg.conf

```bash
pciconf -lv|grep -B4 VGA
```


```bash
cat << EOF > /tmp/xorg.conf
Section "Files"
        ModulePath   "/usr/local/lib/xorg/modules"
        FontPath     "/usr/local/share/fonts/misc"
        FontPath     "/usr/local/share/fonts/cyrillic"
        FontPath     "/usr/local/share/fonts/100dpi/:unscaled"
        FontPath     "/usr/local/share/fonts/75dpi/:unscaled"
        FontPath     "/usr/local/share/fonts/Type1"
        FontPath     "/usr/local/share/fonts/100dpi"
        FontPath     "/usr/local/share/fonts/75dpi"
        FontPath     "built-ins"
		FontPath     "/usr/local/share/fonts/urwfonts/"
		FontPath     "/usr/local/share/fonts/TrueType"
EndSection

Section "Module"
	Load  "freetype"
EndSection

Section "InputDevice"
        Identifier "Teclado0"
        Driver "kbd"
EndSection

Section "InputDevice"
        Identifier "Mouse0"
        Driver "vboxmouse"
EndSection

Section "Device"
        Identifier "Placa0"
        Driver "vboxvideo"
        VendorName "InnoTek Systemberatung GmbH"
        BoardName "VirtualBox Graphics Adapter"
		BusID     "pci0:0:2:0"
EndSection


Section "Monitor"
        Identifier "Monitor0"
        Option "VendorName" "HP"
        Option "ModelName" "Monitor genérico"
        Option "DPMS" "true"
        Modeline "1280x800_60.00"   83.50  1280 1352 1480 1680   800  803  809  831 -hsync +vsync #(49.7 kHz UP)
    Modeline "1280x800_59.08"   83.50  1280 1352 1480 1680   800  803  809  831 -hsync +vsync #(49.7 kHz UP)
    Modeline "952x480_59.09"    35.24   952  976 1064 1176   480  483  493  500 -hsync +vsync #(30.0 kHz P)
    Modeline "2560x1600_60.0"  348.50  2560 2752 3032 3504  1600 1603 1609 1658 -hsync +vsync #(99.5 kHz e)
    Modeline "2560x1600_60.0"  268.50  2560 2608 2640 2720  1600 1603 1609 1646 +hsync -vsync #(98.7 kHz e)
    Modeline "1920x1440_60.0"  234.00  1920 2048 2256 2600  1440 1441 1444 1500 -hsync +vsync #(90.0 kHz e)
    Modeline "1856x1392_60.0"  218.25  1856 1952 2176 2528  1392 1393 1396 1439 -hsync +vsync #(86.3 kHz e)
    Modeline "1792x1344_60.0"  204.75  1792 1920 2120 2448  1344 1345 1348 1394 -hsync +vsync #(83.6 kHz e)
    Modeline "2048x1152_60.0"  162.00  2048 2074 2154 2250  1152 1153 1156 1200 +hsync +vsync #(72.0 kHz e)
    Modeline "1920x1200_59.9"  193.25  1920 2056 2256 2592  1200 1203 1209 1245 -hsync +vsync #(74.6 kHz e)
    Modeline "1920x1200_60.0"  154.00  1920 1968 2000 2080  1200 1203 1209 1235 +hsync -vsync #(74.0 kHz e)
    Modeline "1920x1080_60.0"  148.50  1920 2008 2052 2200  1080 1084 1089 1125 -hsync -vsync #(67.5 kHz e)
    Modeline "1600x1200_60.0"  162.00  1600 1664 1856 2160  1200 1201 1204 1250 +hsync +vsync #(75.0 kHz e)
    Modeline "1680x1050_60.0"  146.25  1680 1784 1960 2240  1050 1053 1059 1089 -hsync +vsync #(65.3 kHz e)
    Modeline "1680x1050_59.9"  119.00  1680 1728 1760 1840  1050 1053 1059 1080 +hsync -vsync #(64.7 kHz e)
    Modeline "1400x1050_60.0"  121.75  1400 1488 1632 1864  1050 1053 1057 1089 -hsync +vsync #(65.3 kHz e)
    Modeline "1400x1050_59.9"  101.00  1400 1448 1480 1560  1050 1053 1057 1080 +hsync -vsync #(64.7 kHz e)
    Modeline "1600x900_60.0"   108.00  1600 1624 1704 1800   900  901  904 1000 +hsync +vsync #(60.0 kHz e)
    Modeline "1280x1024_60.0"  108.00  1280 1328 1440 1688  1024 1025 1028 1066 +hsync +vsync #(64.0 kHz e)
    Modeline "1440x900_59.9"   106.50  1440 1520 1672 1904   900  903  909  934 -hsync +vsync #(55.9 kHz e)
    Modeline "1440x900_59.9"    88.75  1440 1488 1520 1600   900  903  909  926 +hsync -vsync #(55.5 kHz e)
    Modeline "1280x960_60.0"   108.00  1280 1376 1488 1800   960  961  964 1000 +hsync +vsync #(60.0 kHz e)
    Modeline "1366x768_59.8"    85.50  1366 1436 1579 1792   768  771  774  798 +hsync +vsync #(47.7 kHz e)
    Modeline "1366x768_60.0"    72.00  1366 1380 1436 1500   768  769  772  800 +hsync +vsync #(48.0 kHz e)
    Modeline "1360x768_60.0"    85.50  1360 1424 1536 1792   768  771  777  795 +hsync +vsync #(47.7 kHz e)
    Modeline "1280x800_59.8"    83.50  1280 1352 1480 1680   800  803  809  831 -hsync +vsync #(49.7 kHz e)
    Modeline "1280x800_59.9"    71.00  1280 1328 1360 1440   800  803  809  823 +hsync -vsync #(49.3 kHz e)
    Modeline "1280x768_59.9"    79.50  1280 1344 1472 1664   768  771  778  798 -hsync +vsync #(47.8 kHz e)
    Modeline "1280x768_60.0"    68.25  1280 1328 1360 1440   768  771  778  790 +hsync -vsync #(47.4 kHz e)
    Modeline "1280x720_60.0"    74.25  1280 1390 1430 1650   720  725  730  750 +hsync +vsync #(45.0 kHz e)
    Modeline "1024x768_60.0"    65.00  1024 1048 1184 1344   768  771  777  806 -hsync -vsync #(48.4 kHz e)
    Modeline "800x600_60.3"     40.00   800  840  968 1056   600  601  605  628 +hsync +vsync #(37.9 kHz e)
    Modeline "800x600_56.2"     36.00   800  824  896 1024   600  601  603  625 +hsync +vsync #(35.2 kHz e)
    Modeline "848x480_60.0"     33.75   848  864  976 1088   480  486  494  517 +hsync +vsync #(31.0 kHz e)
    Modeline "640x480_59.9"     25.18   640  656  752  800   480  490  492  525 -hsync -vsync #(31.5 kHz e)
        Option "PreferredMode" "1280x800_60.00"
EndSection


Section "Screen"
        Identifier "Tela0"
        Device     "Placa0"
        Monitor    "Monitor0"
        SubSection "Display"
                Viewport   0 0
                Depth     1
                    Modes "1280x800_60.00" "1280x800_59.08" "952x480_59.09" "2560x1600_60.0" "2560x1600_60.0" "1920x1440_60.0" "1856x1392_60.0" "1792x1344_60.0" "2048x1152_60.0" "1920x1200_59.9" "1920x1200_60.0" "1920x1080_60.0" "1600x1200_60.0" "1680x1050_60.0" "1680x1050_59.9" "1400x1050_60.0" "1400x1050_59.9" "1600x900_60.0" "1280x1024_60.0" "1440x900_59.9" "1440x900_59.9" "1280x960_60.0" "1366x768_59.8" "1360x768_60.0" "1280x800_59.8" "1280x800_59.9" "1280x768_59.9" "1280x768_60.0" "1280x720_60.0" "1024x768_60.0" "800x600_60.3" "800x600_56.2" "848x480_60.0" "640x480_59.9"
        EndSubSection
        SubSection "Display"
                Viewport   0 0
                Depth     4
                    Modes "1280x800_60.00" "1280x800_59.08" "952x480_59.09" "2560x1600_60.0" "2560x1600_60.0" "1920x1440_60.0" "1856x1392_60.0" "1792x1344_60.0" "2048x1152_60.0" "1920x1200_59.9" "1920x1200_60.0" "1920x1080_60.0" "1600x1200_60.0" "1680x1050_60.0" "1680x1050_59.9" "1400x1050_60.0" "1400x1050_59.9" "1600x900_60.0" "1280x1024_60.0" "1440x900_59.9" "1440x900_59.9" "1280x960_60.0" "1366x768_59.8" "1360x768_60.0" "1280x800_59.8" "1280x800_59.9" "1280x768_59.9" "1280x768_60.0" "1280x720_60.0" "1024x768_60.0" "800x600_60.3" "800x600_56.2" "848x480_60.0" "640x480_59.9"
        EndSubSection
        SubSection "Display"
                Viewport   0 0
                Depth     8
                    Modes "1280x800_60.00" "1280x800_59.08" "952x480_59.09" "2560x1600_60.0" "2560x1600_60.0" "1920x1440_60.0" "1856x1392_60.0" "1792x1344_60.0" "2048x1152_60.0" "1920x1200_59.9" "1920x1200_60.0" "1920x1080_60.0" "1600x1200_60.0" "1680x1050_60.0" "1680x1050_59.9" "1400x1050_60.0" "1400x1050_59.9" "1600x900_60.0" "1280x1024_60.0" "1440x900_59.9" "1440x900_59.9" "1280x960_60.0" "1366x768_59.8" "1360x768_60.0" "1280x800_59.8" "1280x800_59.9" "1280x768_59.9" "1280x768_60.0" "1280x720_60.0" "1024x768_60.0" "800x600_60.3" "800x600_56.2" "848x480_60.0" "640x480_59.9"
        EndSubSection
        SubSection "Display"
                Viewport   0 0
                Depth     15
                    Modes "1280x800_60.00" "1280x800_59.08" "952x480_59.09" "2560x1600_60.0" "2560x1600_60.0" "1920x1440_60.0" "1856x1392_60.0" "1792x1344_60.0" "2048x1152_60.0" "1920x1200_59.9" "1920x1200_60.0" "1920x1080_60.0" "1600x1200_60.0" "1680x1050_60.0" "1680x1050_59.9" "1400x1050_60.0" "1400x1050_59.9" "1600x900_60.0" "1280x1024_60.0" "1440x900_59.9" "1440x900_59.9" "1280x960_60.0" "1366x768_59.8" "1360x768_60.0" "1280x800_59.8" "1280x800_59.9" "1280x768_59.9" "1280x768_60.0" "1280x720_60.0" "1024x768_60.0" "800x600_60.3" "800x600_56.2" "848x480_60.0" "640x480_59.9"
        EndSubSection
        SubSection "Display"
                Viewport   0 0
                Depth     16
                Modes "1280x800_60.00" "1280x800_59.08" "952x480_59.09" "2560x1600_60.0" "2560x1600_60.0" "1920x1440_60.0" "1856x1392_60.0" "1792x1344_60.0" "2048x1152_60.0" "1920x1200_59.9" "1920x1200_60.0" "1920x1080_60.0" "1600x1200_60.0" "1680x1050_60.0" "1680x1050_59.9" "1400x1050_60.0" "1400x1050_59.9" "1600x900_60.0" "1280x1024_60.0" "1440x900_59.9" "1440x900_59.9" "1280x960_60.0" "1366x768_59.8" "1360x768_60.0" "1280x800_59.8" "1280x800_59.9" "1280x768_59.9" "1280x768_60.0" "1280x720_60.0" "1024x768_60.0" "800x600_60.3" "800x600_56.2" "848x480_60.0" "640x480_59.9"
        EndSubSection
        SubSection "Display"
                Viewport   0 0
                Depth     24
                Modes "1280x800_60.00" "1280x800_59.08" "952x480_59.09" "2560x1600_60.0" "2560x1600_60.0" "1920x1440_60.0" "1856x1392_60.0" "1792x1344_60.0" "2048x1152_60.0" "1920x1200_59.9" "1920x1200_60.0" "1920x1080_60.0" "1600x1200_60.0" "1680x1050_60.0" "1680x1050_59.9" "1400x1050_60.0" "1400x1050_59.9" "1600x900_60.0" "1280x1024_60.0" "1440x900_59.9" "1440x900_59.9" "1280x960_60.0" "1366x768_59.8" "1360x768_60.0" "1280x800_59.8" "1280x800_59.9" "1280x768_59.9" "1280x768_60.0" "1280x720_60.0" "1024x768_60.0" "800x600_60.3" "800x600_56.2" "848x480_60.0" "640x480_59.9"
        EndSubSection
EndSection
EOF
```

### Instalando suporte a fontes True Type1

Instalar o pacote `urwfonts`
```bash
pkg install urwfonts
```

Acrescentar o caminho a seguir junto às outras fontes no arquivo xorg.conf

```bash
Section "Files"
# ...
		FontPath     "/usr/local/share/fonts/urwfonts/"
# ...
EndSection
```

Instalar o pacote `mkfontscale`
```bash
pkg install mkfontscale
```

Acrescentar o caminho a seguir junto às outras fontes no arquivo xorg.conf

```bash
Section "Files"
# ...
		FontPath     "/usr/local/share/fonts/TrueType"
# ...
EndSection
```


