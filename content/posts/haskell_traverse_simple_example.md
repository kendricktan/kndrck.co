---
title: Haskell's Traverse - Simple Examples
date: 2020-07-22
author: Kendrick Tan
categories: ["haskell"]
---

Recently, a [a good friend of mine](http://blog.ielliott.io/) has been explaining to me how powerful the `traverse` function is in Haskell. The main point being that it's a generalization of both `map` and `fold`.

For those who are unfamiliar, `traverse` has the following function definition:

```haskell
traverse
  :: (Applicative f, Traversable t) => (a -> f b) -> t a -> f (t b)
```

At first glance, it was _very_ unclear to me how one could obtain the features of `fold` within traverse. Things only started to click after I digested the following line:

> To get fold, choose an applicative that "accumulates" the result you want

In other words, we can get `fold` for free simply by choosing our desired applicative to wrap `b` in.

### Example 1: Maybe

Lets say we have a list of numbers, and we would like to ensure that all the numbers in the list are all `Just` even numbers, or `Nothing`.

Without using `traverse` could do it like so:

```haskell
myData = [1,2,3,4,5]

isEven x = x `rem` 2 == 0

myResult = do
    let filtered = filter isEven myData
    if length filtered == length myData
        then Just filtered
        else Nothing
```

It _works_, however we can make it more succinct simply by using `traverse`.

```haskell
myData = [2,4,6,8]

isEvenF x
    | x `rem` 2 == 0 = Just x
    | otherwise      = Nothing

myResult = traverse isEvenF myData
```

### Example 2: State Monad

While the previous example was nice and short, it fails to showcase where (in my opinion) `traverse` excels at - **seperating data handling logic from the data traversing logic**. This is particularly useful in highly nested data structures.

What do I mean by that? Lets say we have a list of numbers. And we would like to sum all the even numbers within the list. Traditionally, we can do it like so:

```haskell
myData = [1,2,3,4,5]

isEven x = x `rem` 2 == 0

myResult = sum $ filter isEven myData
```

But what if we had a nested list?

```haskell
myNestedData = take 6 (repeat myData)

myNestedResult = do
    let temp1 = filter isEven <$> myNestedData
    let temp2 = sum <$> temp1
    sum temp2
```

I'm sure we can refactor this to make it look neater, but things do get a bit messy as our data structures gets more nested / complicated.

Compare that with the `traverse` approach:

```haskell
import Control.Monad.State.Lazy

myData = [1,2,3,4,5]

addIfEven x
  | x `rem` 2 == 0 = (+x)
  | otherwise      = id

sumIfEven x = do
  modify (addIfEven x)
  return x

myResult = execState (traverse sumIfEven myData) 0
```

While the initial setup is messier, we can easily compose `traverse` so we can easily perform the same operation regardless of how nested the underlying data structure is:

```haskell
myNestedData = take 6 (repeat myData)

myResult = execState ((traverse . traverse) sumIfEven myData) 0
```

### Conclusion

`traverse`, like many many haskell concepts are extremely unintuitive on first glance, due to how abstract they are. But as you slowly build up a foundation and intuition on these generalized abstractions, they become very powerful tools in your arsenal.