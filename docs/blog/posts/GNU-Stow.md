---
date: 2024-06-29
authors: [darmarj]
description: >
  GNU-Stow helps on dotfile backup throughout different client
categories:
  - GNU Project
---

# Intro
One painful part of keeping a dotfiles directory is managing symbolic links between your configuration files and the locations where they need to go in your home directory.

# What's is GNU-Stow
[GNU Stow](https://www.gnu.org/software/stow/) is a symlink farm manager which takes distinct packages of software and/or data located in separate directories on the filesystem, and makes them appear to be installed in the same place.

# How to install
```bash
# For Arch
sudo pacman -Sy stow
```

# Basic Usage
Let’s say we’ve got our configuration files stored in a directory under the home directory called .dotfiles. We can easily create symbolic links to the files in this directory to the equivalent locations in the home directory using the following command:

```bash title="💁‍♀️ Basic Ref"
# Create the soft link in the parent folder
stow .

# Example
mkdir -p $HOME/dotfiles/ && cd $_   # Create the dot file
mkdir .config   # Prepare for .config folder for stow
mv $HOME/.config/nvim $HOME/dotfiles/.config    # Move the folder that need for stow
stow .  # Stow trigger for syslink
```

# How it works
GNU Stow walks the file and directory hierarchy of the directory passed as the first parameter to the stow command and creates symbolic links to those files in the equivalent locations in the target directory.

The important thing to be aware of here is that our dotfiles directory must have the same layout as where the files would be placed under the home directory. This means you will need to have the equivalent subdirectory structure in your dotfiles folder so that all symbolic links get created in the right place.

# Ignoring files and directories
By default, GNU Stow does a good job of [ignoring common files and directories](https://www.gnu.org/software/stow/manual/stow.html#Types-And-Syntax-Of-Ignore-Lists
) you might not want to be linked back to your home directory like README and LICENSE files, source control folders.

To skip files like this, we can create a file in our dotfiles folder called .stow-local-ignore. Each line of this file should be a string or regular expression representing any file or directory you don’t want to link to your home folder.

```bash title="💁‍♂️    Ignore files and directories"
# Example
\.git
misc
#LICENSE
^/.*\.org
```

# Cleaning up symbolic links
If for some reason you’d like to get rid of all the symbolic links that GNU Stow created in your home folder, you can do that with one extra parameter to the command we’ve been running so far:

```bash
stow -D .   # Under that folder which did run "stow ." command before.
```

# Check out the GNU Stow manual
For more information about GNU Stow and details on other ways it can be used, check out the
[GNU-Stow Manual](https://www.gnu.org/software/stow/manual/)

:material-google-downasaur: [SYSTEM CRAFTERS](https://systemcrafters.net/managing-your-dotfiles/using-gnu-stow/)
