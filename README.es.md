# Dotfiles & Configs

![Qtile](.screenshots/qtile.png)

Idioma
🇪🇸
[🇺🇸](https://github.com/antoniosarosi/dotfiles)

# Table of Contents

- [Resumen](#resumen)
- [Instalación de Arch Linux](#instalación-de-arch-linux)
- [Inicio de sesión y gestor de ventanas](#inicio-de-sesión-y-gestor-de-ventanas)
- [Configuración básica de Qtile](#configuración-básica-de-qtile)
- [Utilidades básicas del sistema](#utilidades-básicas-del-sistema)
  - [Fondo de pantalla](#fondo-de-pantalla)
  - [Fuentes](#fuentes)
  - [Audio](#audio)
  - [Monitores](#monitores)
  - [Almacenamiento](#almacenamiento)
  - [Redes](#redes)
  - [Systray](#systray)
  - [Xsession](#xsession)
- [Otras configuraciones y herramientas](#otras-configuraciones-y-herramientas)
  - [AUR helper](#aur-helper)
  - [Media Transfer Protocol](#media-transfer-protocol)
  - [Explorador de archivos](#explorador-de-archivos)
  - [Tema de GTK](#tema-de-gtk)
  - [Multimedia](#multimedia)
  - [Empieza a hackear](#empieza-a-hackear)
- [Galería](#galería)
  - [Qtile](#qtile)
  - [Spectrwm](#spectrwm)
  - [Openbox](#openbox)
  - [Xmonad](#xmonad)
  - [Dwm](#dwm)
- [Atajos de teclado](#atajos-de-teclado)
  - [Ventanas](#ventanas)
  - [Apps](#apps)
- [Software](#software)

# Resumen

Esta guía te llevará por todos los pasos necesarios para construir un entorno de
escritorio a partir de una instalación limpia basada en Arch Linux. Voy a asumir
que te manejas bien con sistemas operativos basados en Linux y sus líneas de
comandos. Ya que estás leyendo esto, asumiré también que has visto algunos
vídeos de "*tiling window managers*" en Youtube, porque ahí es donde empieza el
sinfín. Puedes elegir el gestor de ventanas que quieras, pero aquí usaremos
Qtile, dado que fue con el que empecé yo. Esta guía es básicamente una
descripción de cómo he construido mi entorno de escritorio desde 0.

# Instalación de Arch Linux

El punto de partida de esta guía es justo después de una instalación basada en
Arch completa y limpia. La
**[Wiki de Arch](https://wiki.archlinux.org/index.php/Installation_guide)**
no te dice qué hacer después de establecer la contraseña del superusuario,
sugiere instalar un cargador de arranque, pero antes de eso yo me aseguraría de
tener internet:

```bash
pacman -S networkmanager
systemctl enable NetworkManager
```

Ahora puedes instalar un cargador de arranque y probarlo de forma "segura", así
es como se haría en hardware moderno,
[suponiendo que has montado la partición efi en /boot](https://wiki.archlinux.org/index.php/Installation_guide#Example_layouts):

```bash
pacman -S grub efibootmgr os-prober
grub-install --target=x86_64-efi --efi-directory=/boot
os-prober
grub-mkconfig -o /boot/grub/grub.cfg
```

Ahora puedes crear tu usuario:

```bash
useradd -m username
passwd username
usermod -aG wheel,video,audio,storage username
```

Para tener privilegios de superusuario necesitamos sudo:

```bash
pacman -S sudo
```

Edita **/etc/sudoers** con nano o vim y descomenta esta línea:

```bash
## Uncomment to allow members of group wheel to execute any command
# %wheel ALL=(ALL) ALL
```

Ahora ya puedes reiniciar:

```bash
# Sal de la imagen ISO, desmontala y extráela
exit
umount -R /mnt
reboot
```

Después de haber iniciado sesión, el internet debería funcionarte sin problema,
pero eso solo aplica si tu ordenador está conectado por cable. Si estás en un
portátil que no tiene puertos Ethernet, probablemente hayas usado
**[iwctl](https://wiki.archlinux.org/index.php/Iwd#iwctl)**
durante la instalación, pero este programa ya no está disponible a no ser que
lo hayas instalado explícitamente. Sin embargo, tenemos
**[NetworkManager](https://wiki.archlinux.org/index.php/NetworkManager)**,
así que no hay problema, para conectarte a una red inalámbrica con este software
solo debes hacer esto:

```bash
# Lista las redes disponibles
nmcli device wifi list
# Conéctate a tu red
nmcli device wifi connect TU_SSID password TU_CONTRASEÑA
```

Échale un vistazo a
[esta página](https://wiki.archlinux.org/index.php/NetworkManager#nmcli_examples)
para otras opciones proporcionadas por *nmcli*. Lo último que tenemos que hacer
antes de pensar en entornos de escritorio es instalar
**[Xorg](https://wiki.archlinux.org/index.php/Xorg)**:

```bash
sudo pacman -S xorg
```

# Inicio de sesión y gestor de ventanas

Primero, necesitamos una forma de iniciar sesión y abrir programas como
navegadores y terminales, así que empezaremos instalando
**[lighdm](https://wiki.archlinux.org/index.php/LightDM)**
y **[qtile](https://wiki.archlinux.org/index.php/Qtile)**.
*lightdm* no funcionará si no instalamos también un
**[greeter](https://wiki.archlinux.org/index.php/LightDM#Greeter)**.
También necesitamos
**[xterm](https://wiki.archlinux.org/index.php/Xterm)**
porque esa es la terminal que *qtile* abre por defecto, hasta que lo cambiemos
en el archivo de configuración. Para editar archivos de configuración
necesitaremos también un editor de texto, puedes usar
**[vscode](https://wiki.archlinux.org/index.php/Visual_Studio_Code)**
o directamente
**[neovim](https://wiki.archlinux.org/index.php/Neovim)**
si tienes experiencia previa, si no no lo recomiendo. Por último necesitamos un
navegador.

```bash
sudo pacman -S lightdm lightdm-gtk-greeter qtile xterm code firefox
```

Activa el servicio de *lightdm* y reinicia el ordenador, deberías poder iniciar
sesión en Qtile a través de *lightdm*.

```bash
sudo systemctl enable lightdm
reboot
```
# Configuración básica de Qtile

Ahora que estás dentro de Qtile, deberías conocer algunos de los atajos de
teclado que vienen por defecto.

| Atajo                | Acción                              |
| -------------------- | ----------------------------------- |
| **mod + enter**      | abrir xterm                         |
| **mod + k**          | ventana siguiente                   |
| **mod + j**          | ventana anterior                    |
| **mod + w**          | cerrar ventana                      |
| **mod + [asdfuiop]** | ir al espacio de trabajo [asdfuiop] |
| **mod + ctrl + r**   | reiniciar qtile                     |
| **mod + ctrl + q**   | cerrar sesión                       |

Antes de hacer nada, si no la distribución del teclado en inglés, deberías
cambiarla usando *setxkbmap*. Abre *xterm* con **mod + enter**, y cambia la
distribución a español:

```bash
setxkbmap es
```

Ten en cuenta que este cambio no es permanente, si reinicias el PC tendrás que
esribir el comando anterior de nuevo. Consulta [esta sección](#xsession) para
hacer cambios permanentes o sigue el orden natural de esta guía si tienes
tiempo suficiente.

Por defecto, no hay menú, tienes que lanzar programas a través de *xterm*. En
este punto puedes instalar otro emulador de terminal si lo prefieres:

```bash
# Instala otro de tu preferencia
sudo pacman -S alacritty
```

Abre el archivo de configuración de Qtile:

```bash
code ~/.config/qtile/config.py
```

Al principio, después de los imports, encontrarás una lista llamada *keys*, que
contiene la línea siguiente:

```python
Key([mod], "Return", lazy.spawn("xterm")),
```

Edítala para lanzar el emulador de terminal que has instalado:

```python
Key([mod], "Return", lazy.spawn("alacritty")),
```

Instala un menú como
**[dmenu](https://wiki.archlinux.org/index.php/Dmenu)**
o **[rofi](https://wiki.archlinux.org/index.php/Rofi)**:

```bash
sudo pacman -S rofi
```

Después añade atajos de teclado para el menú:

```python
Key([mod], "m", lazy.spawn("rofi -show run")),
Key([mod, 'shift'], "m", lazy.spawn("rofi -show")),
```

Reinicia Qtile con **mod + control + r**. Deberías poder abrir el menú y el
emulador de terminal usando los atajos de teclado que acabas de definir. Si has
instalado *rofi*, puedes cambiar su tema:

```bash
sudo pacman -S which
rofi-theme-selector
```

Eso es todo en cuanto a Qtile, puedes empezar a trastear con su configuración y
personalizarlo. Écha un vistazo a mi configuración
[aquí](https://github.com/antoniosarosi/dotfiles/tree/master/.config/qtile).
Pero antes de eso recomiendo configurar utilidades básicas como audio, batería,
montaje de unidades de almacenamiento, etc.

# Utilidades básicas del sistema

En esta sección vamos a ver algunos programas que casi todo el mundo necesita en
su sistema. Pero recuerda que los cambios que haremos no son permanentes,
[esta sección](#xsession) describe cómo conseguir que lo sean.

## Fondo de pantalla

Lo primero es lo primero, tu pantalla se ve negra y vacía, así que probablemente
quieras un fondo más bonito para no sentirte tan deprimido. Puedes abrir
*firefox* usando *rofi* y descargar un fondo de pantalla. Después instala
**[feh](https://wiki.archlinux.org/index.php/Feh)** o
**[nitrogen](https://wiki.archlinux.org/index.php/Nitrogen)**
y pon tu fondo:

```bash
sudo pacman -S feh
feh --bg-scale ruta/al/fondo/de/pantalla
```

## Fuentes

Las fuentes en Arch son básicamente un meme, antes de que te den problemas
puedes simplemente instalar estos paquetes:

```bash
sudo pacman -S ttf-dejavu ttf-liberation noto-fonts
```

Para listar todas las fuentes disponibles:

```bash
fc-list
```

## Audio

En este punto, no hay audio, necesitamos
**[pulseaudio](https://wiki.archlinux.org/index.php/PulseAudio)**.
Recomiendo instalar un programa gráfico para manejar el audio como
**[pavucontrol](https://www.archlinux.org/packages/extra/x86_64/pavucontrol/)**,
porque todavía no tenemos atajos de teclado para ello.

```bash
sudo pacman -S pulseaudio pavucontrol
```

En Arch,
[pulseaudio está activado por defecto](https://wiki.archlinux.org/index.php/PulseAudio#Running),
pero puede que tengas que reiniciar para que *pulseaudio* arranque. Después de
reiniciar, abre *pavucontrol* usando *rofi*, activa el audio (porque está en
"mute") y debería estar todo correcto.

Ahora puedes establecer atajos de teclado para *pulseaudio*, abre el archivo de
configuración de Qtile y añade esto:


```python
# Volumen
Key([], "XF86AudioLowerVolume", lazy.spawn(
    "pactl set-sink-volume @DEFAULT_SINK@ -5%"
)),
Key([], "XF86AudioRaiseVolume", lazy.spawn(
    "pactl set-sink-volume @DEFAULT_SINK@ +5%"
)),
Key([], "XF86AudioMute", lazy.spawn(
    "pactl set-sink-mute @DEFAULT_SINK@ toggle"
)),
```

Aunque para una mejor experiencia en la línea de comandos, recomiendo usar
**[pamixer](https://www.archlinux.org/packages/community/x86_64/pamixer/)**:

```bash
sudo pacman -S pamixer
```

Con ello puedes convertir los atajos de teclado en:

```python
# Volumen
Key([], "XF86AudioLowerVolume", lazy.spawn("pamixer --decrease 5")),
Key([], "XF86AudioRaiseVolume", lazy.spawn("pamixer --increase 5")),
Key([], "XF86AudioMute", lazy.spawn("pamixer --toggle-mute")),
```

Reinicia Qtile con **mod + control + r** y prueba los atajos. Si estás en un
portátil, probablemente también necesites controlar el brillo de tu pantalla,
para ello recomiendo
**[brightnessctl](https://www.archlinux.org/packages/community/x86_64/brightnessctl/)**:

```bash
sudo pacman -S brightnessctl
```

Puedes añadir estos atajos y volver a reiniciar Qtile:

```python
# Brillo
Key([], "XF86MonBrightnessUp", lazy.spawn("brightnessctl set +10%")),
Key([], "XF86MonBrightnessDown", lazy.spawn("brightnessctl set 10%-")),
```

## Monitores

Si tienes múltiples monitores, seguramente quieras usarlos todos. Así es como
funciona **[xrandr](https://wiki.archlinux.org/index.php/Xrandr)**:

```bash
# Lista todas las salidas y resoluciones disponibles
xrandr
# Formato común para un portátil con monitor extra
xrandr --output eDP-1 --primary --mode 1920x1080 --pos 0x1080 --output HDMI-1 --mode 1920x1080 --pos 0x0
```

Es necesario especificar la posición de cada salida, si no se utilizará 0x0, y
todas las salidas estarán solapadas. Ahora bien, si no quieres calcular píxeles
y demás necesitas una interfaz gráfica como
**[arandr](https://www.archlinux.org/packages/community/any/arandr/)**:

```bash
sudo pacman -S arandr
```

Ábrela con *rofi*, ordena las pantallas como quieras, y después puedes guardar
la disposición de las mismas, lo cual simplemente te dará un script con el
comando exacto de *xrandr* que necesitas. Guarda ese script, pero todavía no
le des al botón de aplicar.

Para un sistema con múltiples monitores deberías crear una instancia de *Screen*
por cada uno de ellos en la configuración de Qtile.

Encontrarás una lista llamada *screens* que contiene solo un objeto inicializado
con una barra en la parte de abajo. Dentro de esa barra puedes ver los widgets
con los que viene por defecto.

Añade tantas pantallas como necesites y copia-pega los widgets, más adelante
podrás personalizarlos. Ahora puedes volver a *arandr*, darle click en "apply"
y reiniciar el gestor de ventanats.

Con esto tus monitores deberían funcionar.

## Almacenamiento

Otra utilidad básica que podrías necesitar es montar de forma automática
unidades de almacenamiento externas. Para ello uso
**[udisks](https://wiki.archlinux.org/index.php/Udisks)**
y **[udiskie](https://www.archlinux.org/packages/community/any/udiskie/)**.
*udisks* es una dependencia de *udiskie*, así que solo instalaremos este
último. Instala también el paquete
**[ntfs-3g](https://wiki.archlinux.org/index.php/NTFS-3G)**
para leer y escribir en discos NTFS:

```bash
sudo pacman -S udiskie ntfs-3g
```

## Redes

Hemos configurado la red a través de *nmcli*, pero un programa gráfico es más
cómodo. Yo uso
**[nm-applet](https://wiki.archlinux.org/index.php/NetworkManager#nm-applet)**:

```bash
sudo pacman -S network-manager-applet
```

## Systray

Por defecto, tenemos una "bandeja del sistema" en Qtile, pero no hay nada
ejecutándose en ella. Puedes lanzar los programas que acabamos de instalar así:

```bash
udiskie -t &
nm-applet &
```

Ahora deberías ver unos iconos en la barra, puedes clicar en ellos para
configurar la red y discos. Puedes instalar también iconos para la batería y
el volumen:

```bash
sudo pacman -S volumeicon cbatticon
volumeicon &
cbatticon &
```

## Xsession

Como he mencionado antes, estos cambios no son permanentes. Para que lo sean
necesitamos un par de cosas. Primero instala
**[xinit](https://wiki.archlinux.org/index.php/Xinit)**:

```bash
sudo pacman -S xorg-xinit
```

Ahora puedes usar *~/.xprofile* para lanzar programas antes de que se ejecute
el gestor de venatnas:

```bash
touch ~/.xprofile
```

Por ejemplo, si escribes esto en tu *.xprofile*:

```bash
xrandr --output eDP-1 --primary --mode 1920x1080 --pos 0x1080 --output HDMI-1 --mode 1920x1080 --pos 0x0 &
setxkbmap es &
nm-applet &
udiskie -t &
volumeicon &
cbatticon &
```

Cada vez que inicias sesión tendrás los iconos de la bandeja del sistema, tu
distribución de teclado y monitores configurados.

# Otras configuraciones y herramientas

## AUR helper

Ahora que ya tienes un poco de software que te permite usar tu PC sin perder
la paciencia, es hora de hacer cosas más interesantes. Primero, instala un
**[AUR helper](https://wiki.archlinux.org/index.php/AUR_helpers)**, yo uso
**[yay](https://github.com/Jguer/yay)**:

```bash
sudo pacman -S base-devel git
cd /opt/
sudo git clone https://aur.archlinux.org/yay-git.git
sudo chown -R username:username yay-git/
cd yay-git
makepkg -si
```

Ahora que tienes un *Arch User Repository helper*, puedes instalar prácticamente
todo el software de este planeta que haya sido pensado para correr en Linux.

## Media Transfer Protocol

Si quieres conectar tu teléfono usando un cable USB, necesitarás una
implementación de MTP y alguna interfaz de línea de comandos como
[esta](https://aur.archlinux.org/packages/simple-mtpfs/):

```bash
sudo pacman -S libmtp
yay -S simple-mtpfs

# Lista todos los dispositivos conectados
simple-mtpfs -l
# Monta el primer dispsitivo de la lista anterior
simple-mtpfs --device 1 /mount/point
```

## Explorador de archivos

Hasta ahora siempre hemos manejado los ficheros a través de la terminal, pero
puedes instalar explorador de archivos. Para uno gráfico, recomiendo
**[thunar](https://wiki.archlinux.org/index.php/Thunar)**
y para uno basado en terminal,
**[ranger](https://wiki.archlinux.org/index.php/Ranger)**, aunque este último
está pensado para usuarios de vim, usalo solo si sabes moverte en vim.

## Tema de GTK

El momento que has estado esperando ha llegado, finalmente vas a instalar un
tema oscuro. Yo uso *Material Black Colors*, puedes descargar una versión
[aquí](https://www.gnome-look.org/p/1316887/), con sus respectivos iconos
[aquí](https://www.pling.com/p/1333360/).

Sugiero empezar con
*Material-Black-Blueberry* y *Material-Black-Blueberry-Suru*. Puedes encontrar
otros temas para GTK
[en esta página](https://www.gnome-look.org/browse/cat/135/).
Una vez tengas descargados los temas, puedes hacer esto:

```bash
# Asumiendo que has descargado Material-Black-Blueberry
cd Downloads/
sudo pacman -S unzip
unzip Material-Black-Blueberry.zip
unzip Material-Black-Blueberry-Suru.zip
rm Material-Black*.zip

# Haz tu tema visible a GTK
sudo mv Material-Black-Blueberry /usr/share/themes
sudo mv Material-Black-Blueberry-Suru /usr/share/icons
```

Ahora edita **~/.gtkrc-2.0** y **~/.config/gtk-3.0/settings.ini** añdiendo
estas líneas:

```ini
# ~/.gtkrc-2.0
gtk-theme-name = "Material-Black-Blueberry"
gtk-icon-theme-name = "Material-Black-Blueberry-Suru"

# ~/.config/gtk-3.0/settings.ini
gtk-theme-name = Material-Black-Blueberry
gtk-icon-theme-name = Material-Black-Blueberry-Suru
```

La próxima vez que inicies sesión verás los cambios aplicados. Puedes instalar
también un tema de cursor distinto, para ello necesitas
**[xcb-util-cursor](https://www.archlinux.org/packages/extra/x86_64/xcb-util-cursor/)**
El tema que yo uso es
[Breeze](https://www.gnome-look.org/p/999927/), descargalo, y después:

```bash
sudo pacman -S xcb-util-cursor
cd Downloads/
tar -xf Breeze.tar.gz
sudo mv Breeze /usr/share/icons
```

Edita **/usr/share/icons/default/index.theme** añadiendo esto:

```ini
[Icon Theme]
Inherits = Breeze
```

Ahora vuelve a editat **~/.gtkrc-2.0** y **~/.config/gtk-3.0/settings.ini**:

```ini
# ~/.gtkrc-2.0
gtk-cursor-theme-name = "Breeze"

# ~/.config/gtk-3.0/settings.ini
gtk-cursor-theme-name = Breeze
```

Asegurate de escribir bien los nombres de los temas e iconos, deben ser
exactamente los nombres de los directorios donde se encuentran, los que
ofrece esta salida:

```bash
ls /usr/share/themes
ls /usr/share/icons
```

Recuerda que solo verás los cambios si inicias sesión de nuevo. También hay
herramientas gráficas para cambiar temas, yo simplemente prefiero la forma
tradicional de editar ficheros, pero puedes usar
**[lxappearance](https://www.archlinux.org/packages/community/x86_64/lxappearance/)**,
que es un programa independiente del entorno de escritorio para realizar esta
tarea, y te permie previsualizar los temas.

```bash
sudo pacman -S lxappearance
```

Finalmente, si quieres transparencia y demás azúcal visual instala un compositor:

```bash
sudo pacman -S picom
# Pon esot en ~/.xprofile
picom &
```

## Multimedia

Hay docenas de programas para multimedia, consulta
[esta página](https://wiki.archlinux.org/index.php/List_of_applications/Multimedia)
(Enlazaré algunos más adelante).

## Empieza a hackear

Con todo lo que has hecho hasta ahora ya tienes todas las herramientas para
empezar a trastear con las configuraciones y hacer de tu entorno de escritorio,
bueno, *tu* entorno de escritorio. Lo que recomiendo es empezar añadiendo
atajaos de teclado para programas típicos como firefox, un editor de texto,
explorador de archivos, etc.

Una vez te sientas cómo con Qtile, puedes instalar otros gestores de ventanas
y tendrás más sesiones disponibles al iniciar sesión con *lightdm*.

Aqui tienes una lista con las configuraciones de mis gestores de ventanas,
cada uno tiene su documentación propia:

- [Qtile](https://github.com/antoniosarosi/dotfiles/tree/master/.config/qtile)
- [Spectrwm](https://github.com/antoniosarosi/dotfiles/tree/master/.config/spectrwm)
- [Openbox](https://github.com/antoniosarosi/dotfiles/tree/master/.config/openbox)
- [Xmonad](https://github.com/antoniosarosi/dotfiles/tree/master/.xmonad)
- [Dwm](https://github.com/antoniosarosi/dotfiles/tree/master/.dwm)

# Galería
## Qtile
![Qtile](.screenshots/qtile.png)

## Spectrwm
![Spectrwm](.screenshots/spectrwm.png)

## Openbox
![Spectrwm](.screenshots/openbox.png)

## Xmonad
![Spectrwm](.screenshots/xmonad.png)

## Dwm
![Spectrwm](.screenshots/dwm.png)

# Atajos de teclado

Estos son algunos atajos de teclado comunes a todos mis gestores de ventanas:


## Ventanas

| Key                     | Action                                       |
| ----------------------- | -------------------------------------------- |
| **mod + j**             | siguiente ventana                            |
| **mod + k**             | ventana previa                               |
| **mod + shift + h**     | aumentar master                              |
| **mod + shift + l**     | decrementar master                           |
| **mod + shift + j**     | mover ventana abajo                          |
| **mod + shift + k**     | mover ventana arriba                         |
| **mod + shift + f**     | pasar ventana a flotante                     |
| **mod + tab**           | cambiar la disposición de las ventanas       |
| **mod + [1-9]**         | cambiar al espacio de trabajo N (1-9)        |
| **mod + shift + [1-9]** | mandar ventana al espacio de trabajo N (1-9) |
| **mod + punto**         | Enfocar siguiente monitor                    |
| **mod + coma**          | Enfocar monitor previo                       |
| **mod + w**             | cerrar ventana                               |
| **mod + ctrl + r**      | reiniciar gestor de ventana                  |
| **mod + ctrl + q**      | cerrar sesión                                |

Los siguientes atajos de teclado funcionarán solo si instalas los programas que
lanzan:

```bash
sudo pacman -S rofi thunar firefox alacritty redshift scrot
```

Para configurar *rofi*,
[lee este README](https://github.com/antoniosarosi/dotfiles/tree/master/.config/rofi),
y para *alacritty*, [este](https://github.com/antoniosarosi/dotfiles/tree/master/.config/alacritty).

## Apps

| Key                 | Action                                 |
| ------------------- | -------------------------------------- |
| **mod + m**         | lanzar rofi                            |
| **mod + shift + m** | navegación (rofi)                      |
| **mod + b**         | lanzar navegador (firefox)             |
| **mod + e**         | lanzar explorador de archivos (thunar) |
| **mod + return**    | lanzar terminal (alacritty)            |
| **mod + r**         | redshift                               |
| **mod + shift + r** | parar redshift                         |
| **mod + s**         | captura de pantalla (scrot)            |

# Software
