---
title: How NixOS Made Me a Better Programmer
date: 2024-05-26
permalink: /blog/2024/05/24/how-nixos-made-me-a-better-programmer
tags: programming nixos learning
---

TL;DR: I can read and understand code way faster and better in general. I also more often just open the source code if documentation is lacking or seems incomplete. 

In the last year, I gradually switched from a casual, curious frontend programmer to a seemingly full-fledged nerd. It all started with [My Journey to a 36-Key Keyboard] and then gradually spiraled into a general productivity increase.

## Discovering NixOS

I would be lying if I said I remember exactly how I found NixOS, but I am fairly sure it was through some YouTuber who was showcasing Neovim and using NixOS. It was just around the time when I was trying to create my first `dotfiles` repo, mainly for storing Alacritty and Neovim config. 

If you don't know NixOS, go and look it up. It is the most different Linux "distro" you will find out there. Its main programming language is the functional `nix` language. You use this language to describe the end state of your system in a bunch of files, including `systemd` devices, user config files, installed applications, and various boot information. Then you just build the system. 

What I saw in it was reproducibility, config hell management, and dependency management. So I booted up another system in a dual boot manner and started experimenting with it.

### A Few Months Later

Currently, I am doing all my programming on a NixOS system, including some LLM projects in Python, full-stack Ruby/Go/JS/TS development, and lately, Rust. I am also maintaining a config for my girlfriend's PC in a repo so we can have shared modules. This is great in case I need to do something on the terminal (having all my bindings and tools there too). 

## Who Needs Documentation, Right?

One of the biggest weaknesses of NixOS is the lack of documentation. Where you are used to having searchable and documented packages, now you find vague names of the programs you need. Where you used to be able to debug issues just by googling them, now you find half-decade-old threads that are way more complex for your beginner understanding. 

The people in the community are nice, but for the first time in a long while, I found myself using Matrix chat to get a result for a simple problem I could not solve for days (how to add a runtime dependency to an app). The repository of "packages" (referring here to nixpkgs) is just a config. You need to know the name of the package and all the properties of the package. This is often listed in various places but very rarely includes meaningful comments about what each option does. 

This led me to actually clone the nixpkgs repository and start using my `fzf` fuzzy finder to find packages and figure out from the code which option is which setting in the original application or what options are available for them in general. I got pretty comfortable with it, in fact.

### Documentation on Other Tools is Overrated

Since then, I caught myself doing more digging in code when I am lacking documentation. One of the latest examples is [my plugin for Obsidian](https://github.com/brumik/obsidian-ollama-chat). I had to use LlamaIndex.ts for creating an indexer and wanted to do it in-app. Since Obsidian plugins on mobile run as in-browser, I could not use the node functions baked into LlamaIndex. Well, I just fired up the source code, searched for the classes, and copied out the relevant function calls to my code, skipping `fs.readFile` functions or similar. 

In the process, I also discovered how the `storage context` is working (surprisingly simple), which by documentation is implied as "magic" (you set it up and it works). 

## Understanding Your OS

I know that most people do not need to understand their OS. However, when using NixOS, sooner or later you need to gain some understanding of it. Do you know how your Ubuntu boots up? How to modify the boot partition contents? How that partition knows where to boot from? Well, I did not know. I had formatted a whole disk before just because I messed up dual boot. With NixOS, since it is a config, I had a chance to try changing values and rebuilding the system. This allowed me to try and fail without major time loss. 

Another cool piece of knowledge is the pain and gain with Wayland and XOrg. Oh, and of course my doomed Nvidia 3090 card that needs like 20 different settings to run properly. Obviously, these things are baked into mainstream Linux distros. However, making it work and somewhat understanding how they work makes me feel good and hopefully helps me debug weird issues too.

## Summary

All in all, it is a hassle at first digging up code instead of documentation. However, it is a surprisingly good feeling being able to read foreign code. I am sure that it also gave me ideas when I am writing code, and it helps me in general with code reviews and new complicated code pieces too.

