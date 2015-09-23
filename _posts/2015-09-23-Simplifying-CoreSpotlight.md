---
layout: post
title: Simplifying CoreSpotlight with default implementations in Swift 2.0 protocol extensions
description: This article describes a simple application of Swift 2.0's protocol extensions. We will simplify how types can be easily made searchable with Spotlight by conforming to our protocol while moving the indexing logic in a separat file and reducing code duplication.

---

Default implementations in protocol extensions allow us to inject functionalities in to a type by simply letting it conform to the extended protocol. In this article I want to take a look at how we can remove code duplication with this patttern using the example of making instances of structs or classes available to Spotlight searching.
With `CoreSpotlight`, `NSUserActivity` and `WebMarkup` there are three different ways in iOS 9 to make app content searchable with Spotlight. In this article I am going to focus on `CoreSpotlight` as it is intended to add app-specific content to the on-device index. To start, let's assume we are building a note taking application and have a `Note` struct that we want to index for Spotlight searching. 

{% gist e4c9a6a04ed4e2d281a1 %}

To make a note available to Spotlight, we have to create a `CSSearchableItem` with a `CSSearchableAttributeSet` that describes its content and then pass it as a parameter to `indexSearchableItems(items:completionHandler:)` called on a `CSSearchableIndex`. We will use the default searchable index provided by `defaultSearchableIndex()`, as a own index only has to be used when doing resumable batch updates. Lets have a look on how to create and add a `CSSearchableItem` to the index.

{% gist 113a99b2f2210672df62 %}

The Apple documentation states that the properties `keywords`, `title`, `thumbnailData`, `rating`, and `contentDescription` should be typically set on a `CSSearchableAttributeSet`. Images that should appear as thumbnail in the Spotlight search can be set using the mentioned `thumbnailData`, or alternatively by setting a `thumbnailURL` pointing to a local file. There are however lots of other properties that can be used to describe specific items like places, music or images in high detail, which we will not cover for now and can be looked up in the documentation.

We can image that doing this in every place we want to add or remove something from Spotlight we might end up with a lot of cluttering boilerplate code. For all of types that should be seachable we will have to create a `CSSearchableItem` and `CSSearchableAttributeSet` which we provide with the information needed to index it. By using protocols and protocol extensions, we can extract logic out in a separat file and only leave the types responsible for the mapping to the aformentioned fields, while still beeing able to share code between them. Following is the definition of our `Searchable` protocol:

{% gist afe94752512312377742 %}

The protocol `Searchable` defines common fields used for indexing, as we will later see not all of them have to be implemented in conforming types. Classes and structs that should be searchable through Spotlight can now be modified to conform to this protocol.
Using protocol extensions we can provide default implementations for some of the fields in the protocol. As not all seachable types may want to provide a image to appear in the Spotlight results, the UIImage variable was defined as an Optional in the protocol. In order to not having to implement it and returning `nil` in these cases, we extend the protocol to return `nil` if not other specified by a conforming Type. We are also able to provide a default implementation for the `searchableItem` accessor, in which we simply construct a `CSSearchableItem` using the fields declared in the protocol.

{% gist c4d2814812839624244d %}

Furthermore we can extend the protocol with functions to add and remove it to the `CSSearchableIndex`, as well as remove all entries with the domain of the conforming type.

{% gist 5d1560de43fa455fddc0 %}

Now letting our `Note` struct conform to the `Searchable` protocol (and by that define a mapping) is all that is needed in order to make it easily indexable with CoreSpotlight.

{% gist ea77ea20ec450fb78048 %}

Each `Note` instance can now be added and removed Spotlight search by calling `.addToSpotlightIndex()` and `.removeFromSpotlightIndex()` on it. The index parameter defaults to the default `CSSearchableIndex` and the optional completion closure to `nil`, to make sure the indexing succeeded you should implement the closure and check for errors.

If we now want to add another type like maybe the users friends to Spotlight, we only have to make it conform to our protocol and define the mapping through the properties defined in the protocol. If the newly added type requires to set properties like `altitude` or `city` on the `CSSearchableAttributeSet`, we can extend our protocol like we did with the optional `spotlightImage` and `keywords` properties.
This results in clearer code in places where searchable items are used, while moving the indexing logic in a separat file and is a basic example on how protocols and protocol extensions with default implementations can be used to extend other types.
