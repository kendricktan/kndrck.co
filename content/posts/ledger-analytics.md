---
title: Introducing ledger-analytics - Analytics for ledger-cli
date: 2018-08-26
author: Kendrick Tan
categories: ["tooling", "finance"]
---

### Prelude

Recently, I've been playing around with a really neat accounting tool: [ledger-cli](https://www.ledger-cli.org/). 

I love the simplicity of it, because all it is is a text-file that respects the ledger format. And because it's just a text-file, it works pretty much _everywhere_: Linux, Windows, MacOS, you name it. Not only that, I don't have to download legacy GTK2/GTK3/QT libraries to display the GUI. And, because it's a text-file, all modern developer tools such as `git` are applicable to it.

Sounds great, no?

### Personal Gripes with ledger-cli

ledger-cli is a great tool, but because it's a CLI tool, generating interactive graphs is out of the question. There's a couple of tools out there such as [ledger-web](https://github.com/peterkeen/ledger-web), and [hledger-web](https://hledger.org/hledger-web.html). Problem is they either require too much effort to setup (requiring me to install postgres), don't provide any/enough insight to the data, or both.

### ledger-analytics

[ledger-analytics](https://github.com/kendricktan/ledger-analytics) is my attempt at the problem. Currently the core features it provides are:

1. A general (interactive) overview of your transactions
2. Monthly comparison between multiple accounts
3. Wealthgrowth

The usage preview can be seen below:

![img](https://thumbs.gfycat.com/PaleHeartfeltLice-size_restricted.gif)
##### Account Overview

![img](https://thumbs.gfycat.com/InbornRaggedCarpenterant-size_restricted.gif)
#####  Account Comparison and Wealthgrowth


The project has just started and is currently in alpha, so if you [find any bugs please report them](https://github.com/kendricktan/ledger-analytics/issues). Again, you can [check out the ledger-analytics project here](https://github.com/kendricktan/ledger-analytics).

### Further reading
If you're interesting in plain text accounting, [plaintextaccounting](https://plaintextaccounting.org/) has an awesome guide on getting started.
