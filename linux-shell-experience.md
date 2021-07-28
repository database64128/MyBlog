# Improving Shell Experience on Arch Linux

## 0. Terminal

Install the [GNOME Terminal with transparency patch](https://aur.archlinux.org/packages/gnome-terminal-transparency/). Install [some nerd fonts](https://aur.archlinux.org/packages/nerd-fonts-fira-code/). Install [some color schemes](https://mayccoll.github.io/Gogh/), because the built-in ones are all trash.

```console
❯ paru -Syu --needed gnome-terminal-transparency nerd-fonts-fira-code gogh-git
```

Run `gogh` afterwards to install theme profiles.

I'm using `SweetTerminal`, because *it's purple*.

## 1. [Fish: The Friendly Interactive Shell](https://wiki.archlinux.org/title/Fish)

```console
❯ sudo pacman -Syu --needed fish
```

Add to `.config/fish/conf.d/gpg-tty.fish` to fix GPG signing with `git`:

```fish
set -gx GPG_TTY (tty)
```

## 2. [Starship: The Cross-Shell Prompt for Astronauts](https://starship.rs/)

```console
❯ sudo pacman -Syu --needed starship
```

Add to `.config/fish/conf.d/starship.fish` to enable Starship:

```fish
starship init fish | source
```

## 3. GNU Coreutils Replacements

```console
❯ sudo pacman -Syu --needed bat bottom exa fd tealdeer tokei
```

- [sharkdp/bat: A cat(1) clone with wings.](https://github.com/sharkdp/bat)
- [ClementTsang/bottom: Yet another cross-platform graphical process/system monitor.](https://github.com/ClementTsang/bottom)
- [ogham/exa: A modern replacement for ‘ls’.](https://github.com/ogham/exa)
- [sharkdp/fd: A simple, fast and user-friendly alternative to 'find'](https://github.com/sharkdp/fd)
- [dbrgn/tealdeer: A very fast implementation of tldr in Rust.](https://github.com/dbrgn/tealdeer)
- [XAMPPRocky/tokei: Count your code, quickly.](https://github.com/XAMPPRocky/tokei)
