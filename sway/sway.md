# Ricing Sway

Install fuzzel to replace wmenu. I like how it looks by default.

```
emerge fuzzel
```

I had to unmask it because it was marked as unstable.

Download the Adwaita (Gnome) cursors if you don't have them.

```
sudo emerge --ask x11-themes/adwaita-icon-themes
```

Then in your sway config add:

```
seat seat0 xcursor_theme Adwaita 20
```

to make the cursor size 20 (pixels?).

