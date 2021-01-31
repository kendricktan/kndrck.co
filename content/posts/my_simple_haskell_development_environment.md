---
title: My Simple Haskell Development Environment
date: 2018-12-16
author: Kendrick Tan
categories: ["haskell", "tooling"]
---

## Background
When I started developing in Haskell, there were many opinions on what build-tools, what IDE/text-editor you should use. Each coming with their own pros and cons.

After playing around with the number of options, I eventually found a setup which _just works_, and would like to share that tool stack with anyone / newcomers who are still undecided on what toolchain they should use.

## Project Building and Management
For the project building and management side of things, `Nix` and `Cabal` is my main go-to solution.

### Nix and Cabal
Nix is a package manager for Linux and Unix systems. (Think `virtualenv` but on steroids).

Cabal is a package manager for Haskell, combine that with `Nix` and you'll have next level sandboxing and environment reproducability.

## Editors
I tried many different editors/ide-combos but eventually settled on `NeoVim` with `ghci`, due to it's sheer simplicity.

### NeoVim
![img](https://i.imgur.com/R5i7BPJ.png)

NeoVim is great. It's lightweight, its simple, and it boots up almost instantly. It also does the one thing you want a text editor to do really well - editing text.

[You can grab the neovim config file here](https://gist.githubusercontent.com/kendricktan/e3936d0b93b677d6f4ca843fe74c77d0/raw/b032153693f9b77935a30f946c5d2414cd504300/init.vim)

### GHCI

<strong>G</strong>lasgow's <strong>H</strong>askell <strong>C</strong>ompiler <strong>I</strong>nteractive is a REPL for Haskell. Just because it's simple doesn't mean its not powerful though.

With the use of `typeholes` (`_`), `ghci` will able infer the types and assist you in solving the problem.

![img](https://i.imgur.com/HiGV517.png)

## Conclusion

Developing on Haskell can be as complicated, or as simple as you choose it to be.