# Rofi

![Rofi](./rofi.png)

***Language***
- [🇪🇸 Español](./README.es.md)
- 🇺🇸 English

Dependencies:

```bash
sudo pacman -S papirus-icon-theme
yay -S nerd-fonts-ubuntu-mono
git clone https://github.com/davatorium/rofi-themes.git
sudo cp rofi-themes/User\ Themes/onedark.rasi /usr/share/rofi/themes
```

Delete this line in **/usr/share/rofi/themes/onedark.rasi**

```css
font: "Knack Nerd Font 14";
```

If you are using my window manager configs, **mod + m** will launch
*rofi -show drun* (menu) and **mod + shift + m** will launch *rofi -show* (window navigation).
