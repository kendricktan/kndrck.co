---
title: Indexing Faces on Selfie Central
date: 2017-06-18
author: Kendrick Tan
categories: ["machine learning"]
---

I just wanted to build something cool using machine learning on a bunch of public images. But after showing it to a couple of my *"friends"* they thought it was too creepy and [Selfie Central](https://www.instagram.com) might sue me for breaking their [platform policy](https://www.instagram.com/about/legal/terms/api/) and I should stop doing it.

So, I did what most sane people would do - write a blog post detailing how I did it, and [open source](https://github.com/kendricktan/iffse) it.

__Whats the worst that could happen?__ ( ͡° ͜ʖ ͡°) > [IFFSE - Selfie Central Facial Feature Search Engine](https://iffse.kndrck.co) <

---

## What did you do?
Err.. that's a good question. I basically told a computer to download a bunch of images containing faces from , and sort them according to similarity.

Essentially building a database of indexed faces ( ͡°( ͡° ͜ʖ( ͡° ͜ʖ ͡°)ʖ ͡°) ͡°).

## So how did you do it?

Before we start off doing anything machine learning-y we need a bunch of data, a bunch of #selfies to train our model on. Hmmm, sounds like we need to drop by and pay [some photo sharing website](https://www.instagram.com) a visit ( ͡°╭͜ʖ╮͡° ).

#### Loaning Images from Selfie Central

I've decided to use the word 'loan' instead of scrap, as scrapping has such negative connotations with it. Anyway, I wanted a way to be able to loan images Selfie Central without providing them with any of my keys. After playing around with the web app for a couple of minutes, I discover the [tags endpoint](https://www.instagram.com/explore/tags/selfie/), which gives like infinite images without authentication.

#### Reverse Engineering the tags endpoint

I noticed that everytime I scroll to the bottom of the page, this particular request pops up:

![img](https://i.imgur.com/Lc1Qpim.png)
##### graphql endpoint: https://www.instagram.com/graphql/query/?query_id=17882293912014529&tag_name=selfie&first=6&after=J0HWUYXkgAAAF0HWUYXigAAAFiYA

Looks like an API call, I wonder what could that hold?

```javascript
{
    data: {
        hashtag: {
            name: "selfie",
            edge_hashtag_to_media: {
                count: 304858155,
                page_info: {
                    has_next_page: true,
                    end_cursor: "J0HWUYaEwAAAF0HWUYaAwAAAFlIA"
                },
                edges: [{
                        node: {
                            comments_disabled: false,
                            id: "1539698684278926474",
                            edge_media_to_caption: {
                                edges: [{
                                    node: {
                                        text: "Just now ➡Follow Me Fast , , , #likeforlike #likeforfollow #followtrain #hair #readystock #selfie #good #lookatme #poses @pappya_gaikwad_official @gopal.pathak.143 @cherryscupcake @arianagrande @instagram @justinbieber @banty_dixit_official @officialdhillonpreet @rohit_rony_mr.famous_7050 @priyankachopra @mileycyrus @katyperry"
                                    }
                                }]
                            },
                            shortcode: "BVeG8uxlOyK",
                            edge_media_to_comment: {
                                count: 0
                            },
                            taken_at_timestamp: 1497766407,
                            dimensions: {
                                height: 1350,
                                width: 1080
                            },
                            display_url: "https://instagram.fbne3-1.fna.fbcdn.net/t51.2885-15/e35/19228424_484834361849914_5079388217292095488_n.jpg?se=7",
                            edge_liked_by: {
                                count: 1
                            },
                            owner: {
                                id: "4767834667"
                            },
                            thumbnail_src: "https://instagram.fbne3-1.fna.fbcdn.net/t51.2885-15/s640x640/sh0.08/e35/c0.135.1080.1080/19228424_484834361849914_5079388217292095488_n.jpg",
                            is_video: false
                        }
                    }
               ]
           },
           edge_hashtag_to_content_advisory: {
               count: 0,
               edges: []
           }
       }
   },
   status: "ok"
}
```
##### JSON dump of the graphql endpoint

Woah nifty! The URL to the images are easily accesible and it looks like the only variables we need to change are the `query_id` and `after`. `after` looks suspiciously similar to `end_cursor` obtained from the graphql query endpoint, I wonder if it would make a difference if I changed the `after` variable in our query to be `J0HWUYaEwAAAF0HWUYaAwAAAFlIA`, making our GET request url now: `https://www.instagram.com/graphql/query/?query_id=17882293912014529&tag_name=selfie&first=6&after=J0HWUYaEwAAAF0HWUYaAwAAAFlIA`. Querying it gives us:

![img](https://i.imgur.com/ic6sY24.png)
##### Bingo, we're now able to continously query for new data

What about `query_id`? At first I thought it was some constant as I couldn't find it anywhere and kinda just hardcoded it into my queries and it worked for a while until I started getting timeout errors ♪~ ᕕ(ᐛ)ᕗ . In the end, I found out that the `query_id` is generated from the file `en_US_Commons.js`, which is included during your inital request to Selfie Central.

So, the journey of automating our loaning of images from Selfie Central currently looks like:

1. GET `instagram.com/explore/tags/<TAG>`
2. Find `display_url`(s), `query_id` and `end_cursor` from step 1.
3. Query graphql endpoint using the variables obtained from last step.
4. Store `display_url`(s) and get new `end_cursor` from step 3.
5. Repeat step 3 - 5 until you run out of memory or happy.

So far so good, I wrote a [multithreaded scrapper](https://github.com/kendricktan/iffse/blob/master/scrapper.py), chucked it onto a C4 instance on AWS, and left it overnight.

![img](https://i.imgur.com/vZE6CKH.png)
##### Working all 36 of my cores (ᵔᴥᵔ)

![img](https://i.imgur.com/jge7rKh.png)
##### Indexes around 6-8 images a second

The next morning I woke up and decided to check its progress, and got around 200k images. Sounds good right? ༼ʘ̚ل͜ʘ̚༽ Not really, Selfie Central gets ~35 million images per day, thats only __0.4%__ of the images they get __per day__. But whatevs, 200k is enough for me, I think. (ღ˘⌣˘ღ)


## Preprocessing our images

We now have our images, problem is they aren't normalized.

![img](https://i.imgur.com/qCaGYGe.jpg)
##### random #selfie

See how the above face is tilted to the right? Thats baaaad. We can't have feed it into the black machine learning box that way ب_ب. So I uh, did a bit of googling and found out about this cool library called [dlib](https://pypi.python.org/pypi/dlib) that does what I want it to do. It is able to find the 68 landmarks of the face with some magic, and using that I can transform the original image to an aligned face.

![img](//cdn-images-1.medium.com/max/800/1*AbEg31EgkbXSQehuNJBlWg.png)
##### 68 facial landmaks

Dlib's super easy to use, just from looking at the [example code](https://dlib.net/face_landmark_detection.py.html), I was able to plot my own 68 facial landmarks too!

![img](https://i.imgur.com/tSbx3yn.png)
##### 68 facial landmaks on random #selfie

With a little bit more [magic](https://github.com/kendricktan/iffse/blob/master/iffse/utils/cv/faces.py#L80), I was able to align the face, voila! ◔̯◔

![img](https://i.imgur.com/kRF34rv.png)
##### aligned #selfie

## 'Building' our Machine Learning model

Remember how I said we needed alot of data to train __our__ model? I lied. I tried to get the [MS Celeb 1M](https://www.microsoft.com/en-us/research/project/ms-celeb-1m-challenge-recognizing-one-million-celebrities-real-world/) dataset. But do you really think Microsoft is really going to hand over the dataset to some kid who says he's going to use it to index faces on Selfie Central and be non-creepy? (´・ω・)っ由

In the end, I decided to just use a pretrained model from [OpenFace](https://github.com/cmusatyalab/openface/blob/master/models/get-models.sh). Problem is, these cool kids are using `Torch`, and I'm using `PyTorch`. Whats the difference? There's a `Py` infront of `PyTorch`, having a longer name must mean its newer and better. And communicating between `Lua` (what `Torch` is built on) and `Python` (what `PyTorch` is built on) induces a large overhead and is a huge hassle.

Using my 1337 googling skills, I found a [OpenFacePytorch implementation online from some korean guy who works at intel](https://github.com/thnkim/OpenFacePytorch). I cloned the repository and low and behold! It works! Thank you Pete Tae-hoon Kim!


![img](https://i.imgur.com/1EsTMrW.png)
##### OpenFace in PyTorch!

The architecture used by OpenFace is based on [FaceNet](https://arxiv.org/abs/1503.03832):

![img](https://i.imgur.com/7IiugMo.png)
##### FaceNet Architecture

An analogy I like to use to explain the intuition behind FaceNet is to imagine it as a coin sorting machine. The coin sorting machine can measure the coins' size and weight, and will sort coins of similar size and weight into one pile. FaceNet is doing exactly the same thing, except it's measuring the high level abstract features of the images (position of mouth, eyes, nose), and not low level concrete features (skin color, hair color, etc)

![img](//machinelearningmastery.com/wp-content/uploads/2014/04/plot_mean_shift_1.png)
##### Clustering in 2D, FaceNet does it in 128D

Because of the Triplet Loss, it can be assumed that faces with similar features will have a similar embedding. Now I'm not going to explain CNN's here, but if you would like the intuition, here's a [good post](https://ujjwalkarn.me/2016/08/11/intuitive-explanation-convnets/). 

After our preprocessed image passes the Deep Architecture and the L2 regularization (in pic), the abstract features (e.g. position of nose, eyes) are fed into a Feed Forward Network (embeddings), and it is the activations of the Feed Forward Network (embeddings) that we are after: a 128 dim vector.

Once we index all our loaned preprocessed images with their respective 128 dimensional vector, we can start using [K-nearest neighbours](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm) to search for similar faces. For this I used [Spotify's Annoy](https://github.com/spotify/annoy).

## What's Next?

TBH, not much. I wrote a couple endpoints and spun up a [sanic server](https://github.com/channelcat/sanic), hosted my files, and decided to call it a day.

Just hope the FBI doesn't come knocking on my door. (>人<)

## TL;DR:

Steps to index faces:

1. Load images from Selfie Central
2. Preprocess and normalize loaded images
3. Pass image through a blackbox and index image with blackbox outputs
4. Use an algorithm like KNN to find images with similar indexes


![img](https://media.giphy.com/media/upg0i1m4DLe5q/giphy.gif)


---

UPDATE 1: So... the first server got hugged to death at 4am in the morning. Thank you...? I've spin up a general instance (instead of a micro) to handle the load.

UPDATE 2: Day 2, server ran out of storage space, also added new sharable features :)

UPDATE 3: 2017/06/26 08:30, I decided to shut down the server. If you would like to play around with it more, host your own!

UPDATE 4: 2017/306/30 Got an email from Selfie Central, need to rebrand, losing all my disqus comments. Sad days :(

---