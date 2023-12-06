---
date: 2022-03-21
authors: [darmarj]
description: >
  Transparent Gnome
categories:
  - Gnome
---

# Transparent Gnome Shell

## To use opacity/transparency occasionally - it is better use xprop command.

``` bash
xprop -format _NET_WM_WINDOW_OPACITY 32c -set _NET_WM_WINDOW_OPACITY 0x7FFFFFFF
```

- `0x7FFFFFFF` - 50% opacity
- `0xFFFFFFFF` - 100% opacity

!!! quote "[Forum Reference](https://askubuntu.com/questions/1028/how-to-make-a-window-transparent-in-gnome)"

## One tiny shell script for reference:

``` bash
#!/bin/bash

read -p "Set transparency percentage ? [Enter for 100%]" mydectrans
# only accept 10 to 99, rest is considered 100
[[ "$mydectrans" != [1-9][0-9] ]] && mydectrans=100
# Convert decimal to 32bit hex representation
my32bhextrans=$(printf 0x%x $((0xffffffff * $((mydectrans)) / 100)))
# Execute the action
xprop -f _NET_WM_WINDOW_OPACITY 32c -set _NET_WM_WINDOW_OPACITY $my32bhextrans
```

!!! quote "[Forum Reference](https://askubuntu.com/questions/1283783/is-it-possible-to-set-transparency-per-window-in-ubuntu-20-04/1320458#1320458)"
