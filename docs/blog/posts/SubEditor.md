---
date: 2022-06-28
authors: [darmarj]
description: >
  SublimeEditor
categories:
  - Sublime
---

# SublimE (Unlimited) for Linux User

* bash "sublime_text" is in /opt/sublime_text/sublime_text
* Go to https://hexed.it/
* Open and input "sublime_text"
* Search for "80 78 05 00 0F 94 C1"
* Change into "C6 40 05 01 48 85 C9"
* Do export

```bash
sudo mv /opt/sublime_text/sublime_text /opt/sublime_text/sublime_text.old
```

```bash
sudo mv $HOME/Downloads/sublime_text /opt/sublime_text/sublime_text
```

```bash
sudo chmod 755 /opt/sublime_text/sublime_text
```

```bash
sudo chown root:root /opt/sublime_text/sublime_text
```
