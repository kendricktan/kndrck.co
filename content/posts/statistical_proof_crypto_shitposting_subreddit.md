---
title: Statistical Proof of Shitposting in Crypto Subreddits
date: 2018-03-01
author: Kendrick Tan
categories: ["machine learning", "crypto"]
---


## Prelude

A couple of days ago, an idea came to mind:

_Crypto-currencies that have a high marketcap must have a consensus on quality discussions within their community that distinguishes themselves from the rest of the pack._

Sounds like a reasonable hypothesis, no? Now if I could somehow find cryptocurrencies with a similar pattern of quality discussions on Reddit which haven't 'mooned' yet, I could potentially make bank ( ͡° ͜ʖ ͡°).

Oh boy, was I so __wrong__.

## Data Aggregration and Processing

After checking out a couple of platforms I decided to settle on aggregating data from Reddit because:

1. Most, if not all crypto-projects have a subreddit
2. Relatively active community
3. Lots of documentation on how to aggregate data from Reddit


I started off by getting the top 350 coins and their official subreddit. The marketcap of the lowest and highest from the list ranged from 20 mil all the way up to 100 bil, so I thought I had a pretty diverse range. I obtained the data via [coinmarketcap's api](https://api.coinmarketcap.com/v1/ticker/?limit=350).

Next up was associating each project with their relavent subreddit. There wasn't an easy and reliable way to automate it, and so much of it was [done by hand](https://github.com/kendricktan/cryptoshitposting/blob/master/data/subreddits.json) (ง'̀-'́)ง.

With the list of subreddits, I can now easily grab the top titles and their respective updoots via the JSON endpoint. E.g.: `reddit.com/r/ethereum/top/.json`

Once that was complete I chucked the aggregated data into [TfidfVectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html) which returns me an `N` dimensional matrix that describes the document. The output is then passed the output through [PCA](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html) which compresses the `N` dimensional matix into 2D so we can visualize it.

## Analyzing the Data

Recall the hypothesis:

```
quality content = quality community = quality crypto
shit    content = shit    community = shit    crypto
```

In an ideal world, we should be able to see crypto communities with a similar marketcap cluster together, like the graph below. (Pay no attention to the metrics / axis as they're just the top 2 principal components of the data. Just focus on where the data-point is positioned relative to their marketcap.)

![img](https://i.imgur.com/s6VwjYV.png)

But instead we got:

![img](https://i.imgur.com/IjWOGCs.png)

Notice how most the points _regardless_ of marketcap congregate in one area. That basically means that crypto-subreddits regardless of marketcap, posts similar content. Now go to any crypto subreddit, filter by top of &lt;period&gt;, you should see that it's a shit post. So according to the original hypothesis, all posts are now shitposts. BAM! There you go! __You now have statistical proof that all top posts on crypto-subreddits comprises of shitposts__.

Some people with a sharp eye might argue, hey, what about that mid-right section? You know that section comprises mostly of 500Mil++ marketcap. What does that contain?

![img](https://i.imgur.com/Mvtxkd6.png)

I will admit, that got me excited for a second, until I analyzed what those points represented:

![img](https://i.imgur.com/1B344iz.png)

Turns out, that section represented reddit titles which had the word `bitcoin` in it. Naturally bitcoin and bitcoin-cash frequently had that word in their titles, which represented the majority of the points in that section. When it was used in other high marketcap crypto subreddit it was used to amplify its (the respective crypto) superiority over bitcoin. Anyhow, no worries, it's still a shitpost amirite?

Analyzing top posts by month yields the same result:

![img](https://i.imgur.com/vKaFfMU.png)

## Conclusion and Tl;DR

All crypto subreddits rewards shitposting behavior.

## Misc

If you want to play around with your own graphs you can grab the [cryptoshitposting repository here](https://github.com/kendricktan/cryptoshitposting).

You should also [follow my favorite cat on instagram: mr.miso.oz](https://www.instagram.com/mr.miso.oz/).
