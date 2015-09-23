---
layout: post
title: Simplifying CoreSpotlight
description: Donec at efficitur diam. Vivamus scelerisque nulla et quam feugiat tempor. Etiam pellentesque elit et orci interdum a posuere risus euismod. Aenean convallis dui vitae metus.
---

iOS 9 introduced three different ways to make app content searchable with Spotlight, `CoreSpotlight`, `NSUserActivity` and `WebMarkup`. 
In this article I am going to focus on the `CoreSpotlight` API and how to simplify it using protocols. There might be some confusion on when it is appropriate to use which of the above methods and how they can be used in conjunction, which will be covered in future blog posts.

Let's assume we have a struct called `Post`, that we want to index for Spotlight search.

{% gist e4c9a6a04ed4e2d281a1 %}

To add a Post to Spotlight, we have to create a `CSSearchableItem` with a `CSSearchableAttributeSet` that describes its content and add it to a `CSSearchableIndex`, we will use the default searchable index provided by `defaultSearchableIndex()`, as a own index only has to be used when doing resumable batch updates. To start, lets have a look on how to create and add a `CSSearchableItem` to the index.

{% gist 113a99b2f2210672df62 %}

There are several other properties on `CSSearchableAttributeSet` which might be interesting, like Keywords etc. The API also lets us add pictures to the attribute Set.

I thought about how this could be simplified when indexing different Items in different locations in an application. One could utilize a Singleton or, better, dependency injection to make one object responsible for indexing, but wouldn't it be easier to just call a method on any object that our app wants to appear in Spotlight and be done? This can be accomplished using protocols and protocol extensions in Swift 2.0. Let's start by defining a simple `Searchable` protocol:

{% gist afe94752512312377742 %}

The protocol `Searchable` defines common fields used for indexing, as we will later see not all of them have to be implemented in conforming Types. Classes or Structs that should be searchable through Spotlight can now be modified to conform to this protocol.
Using protocol extensions we can provide default implementations for some of the fields in the protocol. As not all seachable Types may want to provide a image to appear in the Spotlight results, the UIImage variable was defined as an Optional in the protocol. In order to not needing to still implement it and returning `nil` in these cases, we extend the protocol to return `nil` if not other specified by a conforming Type. We are also able to provide a default implementation for the `searchableItem` variable, in which we simply construct a `CSSearchableItem` using the fields declared in the protocol. Furthermore we can extend the protocol with functions to add and remove it to the `CSSearchableIndex`, as well as remove all entries with the domain of the conforming type.

{% gist c4d2814812839624244d %}

All types conforming to `Searchable` now have an accessor for their `CSSearchableItem` which can easily added to a `CSSearchableIndex`. To simplify indexing a little bit more we can add another protocol extension on `Searchable` and implement functions to index or remove items from the index.

{% gist 5d1560de43fa455fddc0 %}

Now letting our `Post` struct conform to the `Searchable` protocol and implement all neccessary fields is all that is needed in order to make it easily indexable with CoreSpotlight.

{% gist ea77ea20ec450fb78048 %}

Each `Post` instance can now be added and removed Spotlight search by calling `.addToSpotlightIndex()` and `.removeFromSpotlightIndex()` on it. The index parameter defaults to the default `CSSearchableIndex` and the optional completion closure to `nil`, to make sure the indexing succeeded you should implement the closure and check for errors.
