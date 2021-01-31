---
title: Capsule Networks Explained
date: 2017-11-10
author: Kendrick Tan
categories: ["machine learning"]
---

----

_Assumed Knowledge: [Convolutional Neural Networks](https://ujjwalkarn.me/2016/08/11/intuitive-explanation-convnets/), [Variational Autoencoders](https://kvfrans.com/variational-autoencoders-explained/)_

__Disclaimer: This article does not cover the mathematics behind Capsule Networks, but rather the intuition and motivation behind them.__

----

### What are Capsule Networks and why do they exist?

The [Capsule Network](https://arxiv.org/abs/1710.09829) is a new type of neural network architecture conceptualized by [Geoffrey Hinton](https://www.cs.toronto.edu/~hinton/), the motivation behind Capsule Networks is to address some of the short comings of Convolutional Neural Networks (__ConvNets__), which are listed below:

#### Problem 1: ConvNets are Translation Invariant [^1]

What does that even mean? Imagine that we had a model that predicts cats. You show it an image of a cat, it predicts that it's a cat. You show it the same image, _but shifted to the left_, it still thinks that it's a cat _without predicting any additional information_.

![img](https://i.imgur.com/mEIUqT8.png)
##### Figure 1.0: Translation Invariance

What we want to strive for is __translation equivariance__. That means that when you show it an image of a cat shifted to the right, it predicts that it's a cat shifted to the right. You show it the same cat but shifted towards the left, it predicts that its a cat shifted towards the left.

![img](https://i.imgur.com/u4ydpQ6.png)
##### Figure 1.1: Translation Equivariance

_Why is this a problem?_ ConvNets are unable to identify the position of one object relative to another, they can only identify if the object exists in a certain region, or not. This results in difficulty correctly identifying objects that hold spatial relationships between features.

For example, a bunch of randomly assembled face parts will look like a face to a ConvNet, because all the key features are there:

![img](https://i.imgur.com/0ZyaPt3.png)
##### Figure 1.2: Translation Invariance

If Capsule Networks do work as proposed, it should be able to identify that the face parts aren't in the correct position relative to one another, and label it correctly:

![img](https://i.imgur.com/mLt9suH.png)
##### Figure 1.3: Translation Equivariance

#### Problem 2: ConvNets require a _lot_ of data to generalize [^2].

In order for the ConvNets to be translation invariant, it has to learn different filters for each different viewpoints, and in doing so it requires a __lot__ of data.

#### Problem 3: ConvNets are a bad representation of the human vision system

According to Hinton, when a visual stimulus is triggered, the brain has an inbuilt mechanism to _"route"_ low level visual data to parts of the brain where it belives can handle it best. Because ConvNets uses layers of filters to extract high level information from low level visual data, this routing mechanism is absent in it.

![img](https://i.imgur.com/CVtE4HG.png)
##### Figure 1.4: Humans vs CNN

Moreover, the human vision system imposes coordinate frames on objects in order to represent them. For example:

![img](https://i.imgur.com/W8peps6.png)
##### Figure 1.5: Imposing a coordinate frame

And if we wanted to compare the object in Figure 1.6 to say the letter 'R', most people would perform mental rotation on the object to a point of reference which they're familiar to before making the comparison. This is just not possible in ConvNets due to the nature of their design.

![img](https://thumbs.gfycat.com/PortlyGracefulBichonfrise-size_restricted.gif)
##### Figure 1.6: Mental Rotation then realizing it's not 'R'

We'll explore this idea of imposing a bounding rectangle and performing rotations on objects relative to their coordinates later on.

### How do Capsule Networks solve these issues?

#### Inverse Graphics

> You can think of (computer) vision as "Inverse Graphics" - Geoffrey Hinton [^3]

What is _inverse graphics_? Simply put, it's the inverse of how a computer renders an object onto the screen. To go from a mesh object onto pixels on a screen, it takes the pose of the whole object, and multiplies it by a transformation matrix. This outputs the pose of the object's part in a lower dimension (2D), which is what we see on our screens.

![img](https://i.imgur.com/DCmDyHl.png)
##### Figure 2.0: Computer Graphics Rendering Process (Simplified)

So why can't we do the opposite? Get pixels from a lower dimension, multiply it by the inverse of the transformation matrix to get the pose of the whole object?

![img](https://i.imgur.com/fOqnQ3C.png)
##### Figure 2.1: Inverse Graphics (Proposed)

Yes we can (on an approximation level)! And by doing that, we can represent the relationship between the object as a whole and the pose of the part as a matrix of weights. And these matrices of weights are __viewpoint invariant__, meaning that however much the pose of the part has changed we can get back the pose of the whole using the same matrix of weights.

__This gives us complete independence between the viewpoints of the object in a matrix of weights. The translation invariance is now represented in the matrix of weights, and not in the neural activity__.

#### Where do we get the matrix of weights to represent the relationship?

![img](https://i.imgur.com/2fHUQrQ.png)
##### Figure 2.2 Extract from Dynamic Routing Between Capsules [^4]

In [Hinton's Paper](https://arxiv.org/pdf/1710.09829.pdf) he describes that the Capsule Networks use a reconstruction loss as a regularization method, similiar to how an autoencoder operates. __Why is this significant?__

![img](https://i.imgur.com/eCmc5fR.jpg)
##### Figure 2.3 Autoencoder Architecture

In order to reconstruct the input from a lower dimensional space, the Encoder and Decoder needs to __learn a good matix representation to relate the relationship between the latent space and the input__, _sounds familiar_?

To summarize, by using the reconstruction loss as a regularizer, the Capsule Network is able to learn a global linear manifold between a whole object and the pose of the object as a matrix of weights via unsupervised learning. As such, the _translation invariance_ is encapsulated in the matrix of weights, and not during neural activity, making the neural network _translation equivariance_. Therefore, we are in some sense, performing a 'mental rotation and translation' of the image when it gets multiplied by the global linear manifold!

#### Dynamic Routing

Routing is the act of relaying information to another actor who can more effectively process it. ConvNets currently perform routing via pooling layers, most commonly being _max pooling_.

![img](https://computersciencewiki.org/images/8/8a/MaxpoolSample2.png)
##### Figure 3.0: Max Pooling with a 2x2 Kernel and 2 Stride

Max pooling is a very primitive way to do routing as it only attends to the most active neuron in the pool. Capsule Networks is different as it tries to send the information to the capsule above it that is best at dealing with it.

![img](https://i.imgur.com/Vd9kw7m.png)
##### Figure 3.1: Extract from Dynamic Routing Between Capsules [^4]

### Conclusion

Using a novel architecture that mimics the human vision system, Capsule Networks strives for _translation equivariance_ instead of _translation invariance_, allowing it to generalize to a greater degree from different view points with less training data.

You can check out a [barebone implementation of Capsule Network here](https://gist.github.com/kendricktan/9a776ec6322abaaf03cc9befd35508d4), which is just a cleaned up version of [gram.ai's implementaion](https://github.com/gram-ai/capsule-networks).


[^1]: [CNN: Translation Equivariance and Invariance](https://aboveintelligent.com/ml-cnn-translation-equivariance-and-invariance-da12e8ab7049)
[^2]: [The Impact of Imbalanced Training Data for Convolutional Neural Networks](https://www.kth.se/social/files/588617ebf2765401cfcc478c/PHensmanDMasko_dkand15.pdf)
[^3]: [What is wrong with Convolutional Neural Networks?](https://youtu.be/rTawFwUvnLE?t=1750)
[^4]: [Dynamic Routing Between Capsules](https://arxiv.org/pdf/1710.09829.pdf)