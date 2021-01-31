---
title: How to Install GHC v8+ on Ubuntu
date: 2018-04-22
author: Kendrick Tan
categories: ["haskell", "tooling"]
---

This post is merely to guide future me on how to install GHC v8.x.x on Ubuntu 16.04 without using [stack](https://github.com/commercialhaskell/stack.git).


e.g. For ghc 8.2.2 
```bash
sudo add-apt-repository ppa:hvr/ghc
sudo apt-get update
sudo apt-get install ghc-8.2.2 cabal-install-2.2
sudo ln -s /opt/ghc/bin/ghc-8.2.2 /usr/bin/ghc
sudo ln -s /opt/ghc/bin/ghci /usr/bin/ghci
sudo ln -s /opt/cabal/bin/cabal-2.2 /usr/bin/cabal
```