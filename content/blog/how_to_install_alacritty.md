---
title: "How To Install Alacritty"
date: 2024-04-09T11:33:03+05:30
draft: false
author: Jewel Mahanta
tags: [terminal, linux, alacritty]
image: /alacritty_on_mint.webp
description: A step by step guide on how to install Alacritty
toc: 
---

I recently switched to Alacritty on my laptop and wanted to share how to get it up and running. Alacritty, much like kitty and wezterm, is a GPU accelerated terminal emulator. It is also the lightest and the fastest among the three and it plays well with TMUX which is a must for me.

## Install Steps
Alacritty is not in APT so we can’t simply do apt install alacritty. Alacritty also does not provide binaries for linux. So we will have to manually build the binary from the source code.

## Step 1 - Installing Rust
Alacritty is written in rust so it will need a working rust compiler. We will use rustup.rs to install and manage rust. Open up your terminal (Ctrl + Alt + T) and start by doing:
```bash
sudo apt update
```
then,
```bash
# Install rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
**Choose default while installing.** Once you are done, do:
```bash
rustup override set stable
rustup update stable
```

## Step 2 - Dependencies
Alacritty also needs a few other libraries before we can build. Let’s get those:
```bash
sudo apt install cmake pkg-config libfreetype6-dev libfontconfig1-dev libxcb-xfixes0-dev libxkbcommon-dev python3
```

## Step 3 - Download the source code
Now we have everything we need to build our own Alacritty binary. We just need the source code. Let’s get that:
```bash
cd ~
git clone https://github.com/alacritty/alacritty.git
cd alacritty
```
This will place alacritty in your HOME_FOLDER/alacritty

## Step 4 - Let’s start building
For the following steps please stay in the alacritty directory.
```bash
cargo build --release
```
This step usually takes a while. Once this is done, we are ready to move onto the final steps:

Adding Alacritty to terminfo
```bash
infocmp alacritty
```
If this returns without any errors, we can move on to the next step. Otherwise run the following:
```bash
sudo tic -xe alacritty,alacritty-direct extra/alacritty.info
```

### Desktop entry
To add alacritty to your desktop menu, do:
```bash
sudo cp target/release/alacritty /usr/local/bin # or anywhere else in $PATH
sudo cp extra/logo/alacritty-term.svg /usr/share/pixmaps/Alacritty.svg
sudo desktop-file-install extra/linux/Alacritty.desktop
sudo update-desktop-database
```

### Manual Page
This lets you do man alacritty
```bash
sudo mkdir -p /usr/local/share/man/man1
sudo mkdir -p /usr/local/share/man/man5
scdoc < extra/man/alacritty.1.scd | gzip -c | sudo tee /usr/local/share/man/man1/alacritty.1.gz > /dev/null
scdoc < extra/man/alacritty-msg.1.scd | gzip -c | sudo tee /usr/local/share/man/man1/alacritty-msg.1.gz > /dev/null
scdoc < extra/man/alacritty.5.scd | gzip -c | sudo tee /usr/local/share/man/man5/alacritty.5.gz > /dev/null
scdoc < extra/man/alacritty-bindings.5.scd | gzip -c | sudo tee /usr/local/share/man/man5/alacritty-bindings.5.gz > /dev/null
```

### Shell Completions
Bash
```bash
mkdir -p ~/.bash_completion
cp extra/completions/alacritty.bash ~/.bash_completion/alacritty
echo "source ~/.bash_completion/alacritty" >> ~/.bashrc
```
**Congrats! you are finally done. You can open Alacritty by going to your menu and finding the entry. Have fun tinkering.**

To customize Alacritty we need a .alacritty.toml file (which can be placed in the home directory). If you want to see my file you can check out my dotfiles.