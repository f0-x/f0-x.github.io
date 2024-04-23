+++
title = "ES6 Maps are underrated!"
publishDate = 2024-04-21T00:00:00+05:45
lastmod = 2024-04-22T11:24:47+05:45
tags = ["reactjs", "javascript","algorithms"]
categories = ["algorithms", "frontend"]
draft = false
featuredImage = "images/maps-vs-array-feature-image.jpg"
featuredImagePreview = "./posts/js-map-vs-array/images/maps-vs-array-feature-image-preview.jpg"
+++

Javascript `Map` is a rarely used data structure, hence it is not as widely understood as it would first appear, also is not as little as it first appears. We will start off with some fundamentals in this post and gradually head towards explaining the `Map`'s function as well as certain situations in which it may be helpful.

## The Big :o: Notation

Understanding Big O Notation is crucial for analyzing the efficiency and scalability of algorithms and data structures. It allows developers to assess how algorithms will perform as the size of the input data increases. The notation provides a standardized way to express the worst-case scenario of an algorithm’s runtime in relation to the size of the input. As developers, it’s essential to choose algorithms and data structures with favorable time complexities to ensure efficient performance, especially when dealing with large datasets.

When considering all this data, it becomes apparent that it’s best for the methods we use not to be dependent on the size of **’n’** data. While these operations may be fast enough to go unnoticed individually, combining multiple operations simultaneously or nested loops can significantly degrade the user experience.

{{< admonition type=info title="Big O Analogy" open=true >}}
Imagine you're a chef. **Big O Notation** is like a recipe that tells you how much more time you'll need if you decide to cook for a banquet instead of a family dinner. It's a way to measure the **_"heaviness"_** of a recipe. Just like you'd want a dish that can be scaled up easily for a large crowd, in programming, you want algorithms that can handle growing amounts of data gracefully. So, Big O is like a heads-up:

> _"Hey, this recipe might take a bit longer if you add more guests,"_

{{< /admonition >}}

## `Array.prototype.find()` :mag:

The [`find()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find) method in JavaScript performs a linear search on an array. It returns the first element that satisfies the given condition, and it reads through all the data until it finds that element. Therefore, considering the worst-case scenario, where the item we are searching for is the last element of the array, we say it has a time complexity of `O(n)`. This situation is exemplified in *Aditya Bhargava*'s book [Grokking Algorithms](https://www.manning.com/books/grokking-algorithms) as follows:

{{< figure src="images/187d6bfb52acd189fdcf1dd8c5478470c8b2f5a8.png" caption="*Linear Search*">}}

{{< admonition type=info title="The Library of Hashes" open=true >}}
You’re in a huge library filled with books, and you’re tasked with finding specific information. Without a system, you’d have to look through each book one by one, which is time-consuming and inefficient, especially if the library keeps getting more books.

Now, imagine you have a magical index that instantly points you to the exact location of the information you need, no matter how many books are in the library. This magical index is like a hash map in programming. It doesn’t matter if you have 10 books or 10,000; the magical index (hash map) helps you find your information quickly every time, which is why we say it has a time complexity of O(1)—it’s always fast, regardless of the size of the library (or data). 📚✨
{{< /admonition >}}

## Hash `Map` :world_map:

A hash map is essentially a storage that can map keys to their corresponding values. When you provide it with a key, it returns the value associated with that key, and you can perform insertion and deletion operations using the same key value in these hash tables. So, it’s a very fast storage system and requires `O(1)` time complexity for all these operations.

{{< figure src="images/hash-map.png" caption="*Hash Map*">}}

Nevertheless, there are certain considerations to keep in mind. For instance, if keys are distributed using an inefficient hash function, collisions may occur, complicating search operations. Additionally, maintaining a hash table with a fill rate exceeding approximately **70%** can result in performance degradation.

## `Map` > `Array` ? :thinking:

In recent projects, the `find()` method is commonly used within user interactive components. For instance, when we need to retrieve data based on parameters from dynamic routes, manipulate specific data, or fetch information selected from dropdowns or input fields, methods like `find` (and occasionally `filter`) come into play. These operations follow a linear approach and have a time complexity of `O(n)`. Now, let's explore how a `Map` can simplify accomplishing these tasks.

{{<highlight javascript >}}
const data = [
  ...
  {
    "age": 50,
    "email": "atuny@@sohu.com",
    "firstName": "Terry",
    "lastName": "Medhurst",
    "gender": "male",
    "id": 1,
    "username": "atunyo",
  },
  {
    "age": 28,
    "email": "hbingleyl@plala.or.jp",
    "firstName": "Sheldon",
    "lastName": "Quigley",
    "gender": "male",
    "id": 2,
    "username": "hbingley1",
  }
  ...
]
{{</highlight>}}

## What is a `Map`? :flashlight:

The `Map` in JavaScript provides a data structure for mapping keys to values. Unlike arrays, which are indexed by a range of numbers, maps can use **any value** as a key. This flexibility makes maps useful in various scenarios, especially when dealing with complex data structures or when **quick access to data** is required.

{{<highlight javascript "style=dracula">}}
let hashData = null;
const getUsers = async () => {
  try {
    const response = await fetch("https://dummyjson.com/users");
    const data = await response.json();
    hashData = new Map(data.users.map((user) => [user.username, user]));
  } catch (error) {
    console.error("Oops:", error);
  }
};
{{</highlight>}}
To convert our data into a `Map` using this method and to ensure that our keys are unique, we need to iterate over our data once. The only point we need to pay attention to is that our keys must be **unique**. Then we have the following result;

{{<highlight javascript>}}
Map(30) { atuny0 → {…}, hbingley1 → {…}, rshawe2 → {…}, yraigatt3 → {…}, kmeus4 → {…}, jtreleven5 → {…}, dpettegre6 → {…}, ggude7 → {…}, nloiterton8 → {…}, umcgourty9 → {…}, … }
size: 30
<entries>
0: atuny0 → Object { id: 1, firstName: "Terry", lastName: "Medhurst", … }
1: hbingley1 → Object { id: 2, firstName: "Sheldon", lastName: "Quigley", … }
2: rshawe2 → Object { id: 3, firstName: "Terrill", lastName: "Hills", … }
3: yraigatt3 → Object { id: 4, firstName: "Miles", lastName: "Cummerata", … }
4: kmeus4 → Object { id: 5, firstName: "Mavis", lastName: "Schultz", … }
5: jtreleven5 → Object { id: 6, firstName: "Alison", lastName: "Reichert", … }
6: dpettegre6 → Object { id: 7, firstName: "Oleta", lastName: "Abbott", … }
<prototype>: Map.prototype { … }
{{</highlight>}}

This way, instead of using `find`, we can access the value associated with a key in our `Map` data structure using `get` with a time complexity of `O(1)`, independent of the size of the data. Moreover, using the `has` method, we can check for the existence of that data with a time complexity of `O(1)` instead of using `includes` with a time complexity of `O(n)`. Keys are case-sensitive, which provides us with convenience.

{{< style "display:flex; justify-content:center;" >}}
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">JavaScript Maps are super underrated.<br><br>Keys of a map can be anything - including an object!<br><br>Here I&#39;m using the new groupBy proposed API to group array items into a Map or an Object → <a href="https://t.co/RirGlB5akF">pic.twitter.com/RirGlB5akF</a></p>&mdash; Wes Bos (@wesbos) <a href="https://twitter.com/wesbos/status/1697246046009864504?ref_src=twsrc%5Etfw">August 31, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
{{< /style >}}

### Methods of the Map data-structure :pushpin:

{{<highlight javascript>}}
hashData.clear() // delete all items
hashData.forEach((e) => e.email) // loop for all items
hashData.get("ggude7") // search a key, returns the value
hashData.has("ggude7") // search a key if its exist as a Boolean
hashData.set("testUser", "user123") // add a new item with unique key
hashData.delete("ggude7") // delete an item with a key
hashData.keys() // iterate keys
hashData.values() // iterate values
hashData.size // returns how many elements it has
{{</highlight>}}

## Final Thoughts :thought_balloon:

In conclusion, both JavaScript `Maps` and `Arrays` have their own strengths and use cases.

{{< mermaid >}}
gantt
    dateFormat  YYYY-MM-DD
    title When to use Maps OR Arrays ?
    section Use Arrays When
    An ordered collection            :done,    des1, 2014-01-06,2014-01-08
    filtering, mapping, and reduction :active,  des2, 2014-01-09, 3d
    Access by index               :         des3, after des2, 5d
    section Use Maps When
    Key-Value pairs:crit, done, 2014-01-06,24h
    Keys based fast-lookup         :crit, done, after des1, 2d
    Complex Data Structures            :crit, active, 3d
    Non-Numeric keys       :crit, 5d
{{< /mermaid >}}


It's important to note that creating a `Map` in JavaScript can be resource-intensive, so it's only worthwhile if you anticipate using `find()` frequently.

{{< admonition type=tip title="JSON as an alternative" open=true >}}
Alternatively, you can store keys as properties within a JavaScript `Object` and access values that way. This approach is nearly identical to using a `Map`, but it has the added benefit of being serializable into `JSON`.
{{< /admonition >}}


Lastly, when working with JavaScript for front-end development, it's worth considering whether excessive focus on performance is a wise decision or not. In my POV, script execution speed rarely becomes the bottleneck for web page or app load times. Rather than vigorously replacing every `find()` with a `Map`, consider optimizing your functions & APIs or refining your UI to enhance the overall user experience. Prioritizing milliseconds of speed improvement in JavaScript code may not always yield noticeable benefits.

## References
* [Grokking Algorithms](https://www.manning.com/books/grokking-algorithms)
* [Linear Search in JS](https://www.doabledanny.com/linear-search-in-javascript)
* [ES6 `Map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
* [Array.prototype.find](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find)
* [Array.prototype.filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)
* [Array.prototype.includes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes)
