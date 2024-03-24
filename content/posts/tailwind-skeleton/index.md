+++
title = "✨Shiny Skeleton Animation using TailwindCSS💨"
publishDate = 2022-07-03T00:00:00+05:45
lastmod = 2022-07-20T11:24:47+05:45
tags = ["tailwind", "css"]
categories = ["UI", "UX"]
draft = false
featuredImage = "assets/featured-image.gif"
featuredImagePreview = "./posts/tailwind-skeleton/assets/featured-image-preview.gif"

+++

Content loaders, skeleton screens, ghost elements, and content placeholders. These are the names given to the effect we'll be exploring today. Many companies, such as LinkedIn, Facebook, YouTube and Slack, use this effect in their apps and websites, as you may have noticed. As much as we developers want our websites to load as quickly as possible, there are times when a lot of data needs to be rendered on the page, so Skeleton screens are a great option.

{{< admonition type=info title="CSS technology used" open=true >}}
Although some of the above mentioned sites use Vanilla CSS and other CSS libs/frameworks to achieve the effect, I have used the latest version of Tailwind CSS (v3.1.5) as of date to create it. 
{{< /admonition >}}

Developers earlier used to add a loading spinner animation to indicate that the page resources are still being fetched in the background which doesn't fit the UI/UX standards of the current timeline.

{{< figure src="assets/loading.gif" title="Classic Loading Spinner Animation">}}

This boring spinner animation would definitely result a lower score in [User Experience Web Index](https://userresearch.google.com/).

{{< typeit tag=h2 >}}
 Introducing the skeleton loading screen.
{{< /typeit >}} 

{{< figure src="assets/tailwind6.gif" title="Animated skeleton example I grabbed from some random site">}}



## Building a  Skeleton Block 🦴{#building-a-base-skeleton}

First of all, lets use Tailwind's range of utility classes to create a skeleton that roughly resembles the content you're about to load. I have created a simple block to denote a member of our skeleton animation which may(not) scale throughout the viewport. 

{{< admonition type=note title="Size Limit" open=true >}}
For demonstration purposes I have limited the wrapper width to `200px`. If you were to increase the dimension, you would have to update the CSS props accordingly. 
{{< /admonition >}}

{{< figure src="assets/tailwind1.png" >}}

```html
<div class="flex min-h-screen items-center justify-center bg-gray-900">
  <div class="w-[200px]">
    <div class="space-y-5 rounded-2xl bg-white/5 p-4 shadow-xl shadow-black/5">
      <div class="h-24 rounded-lg bg-rose-100/10"></div>
      <div class="space-y-3">
        <div class="h-3 w-3/5 rounded-lg bg-rose-100/10"></div>
        <div class="h-3 w-4/5 rounded-lg bg-rose-100/20"></div>
        <div class="h-3 w-2/5 rounded-lg bg-rose-100/20"></div>
      </div>
    </div>
  </div>
</div>
```

## Linear Gradient Overlay 🌊

I have to admit that I don't cross paths often with Linear Gradients on my day-to-day work. However with enough effort I was able to use Tailwind's gradient color stops to create a gradient that fades from transparent to white and back to transparent.

{{< figure src="assets/tailwind2.png" >}}

```html
<div class="flex min-h-screen items-center justify-center bg-gray-900">
  <div class="w-[200px]">
    <div class="h-48 bg-gradient-to-r from-transparent via-rose-100/20 to-transparent"></div>
  </div>
</div>
```

## Keyframe Animation 🎭

Lets define a CSS keyframe animation that translates elements 100% to the right in the extend keyframes object of `tailwind.config.js`. I have used [Tailwind's arbitrary values](https://tailwindcss.com/docs/adding-custom-styles#using-arbitrary-values) to apply the keyframe animation to the overlay element. I have added a custom keyframe shift by the name of *shimmer* as well as curated my own color named **stone** from the Tailwind's default color palette by importing `tailwindcss/colors`. 

{{< admonition type=example title="tailwind.config.js" open=true >}}

```javascript
const colors = require('tailwindcss/colors')
module.exports = {
  theme: {
    extend: {
      colors: {
        gray: colors.stone,
      },
    },
    keyframes: {
      shimmer: {
        '100%': {
          transform: 'translateX(100%)',
        },
      },
    },
  },
  plugins: [],
}
```

{{< /admonition >}}

{{< figure src="assets/tailwind3.gif" >}}

```html
<div class="flex min-h-screen items-center justify-center bg-gray-900">
  <div class="w-[200px]">
    <div class="h-48 -translate-x-full animate-[shimmer_2s_infinite] bg-gradient-to-r from-transparent via-rose-100/20 to-transparent"></div>
  </div>
</div>
```

## Mixing the Elements 🥢

Now we shall combine our skeleton block with the overlay animation by adding the overlay to a pseudo-element of the skeleton wrapper using Tailwind's `before:` modifier.

{{< figure src="assets/tailwind4.gif" >}}

```html
<div class="flex min-h-screen items-center justify-center bg-gray-900">
  <div class="w-[200px]">
    <div class="relative space-y-5 rounded-2xl bg-white/5 p-4 shadow-xl shadow-black/5 before:absolute before:inset-0 before:-translate-x-full before:animate-[shimmer_2s_infinite] before:bg-gradient-to-r before:from-transparent before:via-rose-100/10 before:to-transparent">
      <div class="h-24 rounded-lg bg-rose-100/10"></div>
      <div class="space-y-3">
        <div class="h-3 w-3/5 rounded-lg bg-rose-100/10"></div>
        <div class="h-3 w-4/5 rounded-lg bg-rose-100/20"></div>
        <div class="h-3 w-2/5 rounded-lg bg-rose-100/20"></div>
      </div>
    </div>
  </div>
</div>
```

## Finishing Touches 🎨

To make our overlay animation sharper I tried hiding the overlay while it's positioned outside the skeleton. This limits the overlay overflow within the skeleton's block and gives our effect a sharper look. 

{{< admonition type=success title="Optional" open=true >}}
Add a white border to the top of the overlay to simulate light reflecting on the top of the skeleton. 
{{< /admonition >}}

{{< figure src="assets/tailwind5.gif" >}}

```html
<div class="flex min-h-screen items-center justify-center bg-gray-900">
  <div class="w-[200px]">
    <div class="relative space-y-5 overflow-hidden rounded-2xl bg-white/5 p-4 shadow-xl shadow-black/5 before:absolute before:inset-0 before:-translate-x-full before:animate-[shimmer_2s_infinite] before:border-t before:border-rose-100/10 before:bg-gradient-to-r before:from-transparent before:via-rose-100/10 before:to-transparent">
      <div class="h-24 rounded-lg bg-rose-100/10"></div>
      <div class="space-y-3">
        <div class="h-3 w-3/5 rounded-lg bg-rose-100/10"></div>
        <div class="h-3 w-4/5 rounded-lg bg-rose-100/20"></div>
        <div class="h-3 w-2/5 rounded-lg bg-rose-100/20"></div>
      </div>
    </div>
  </div>
</div>
```

## Final Words 🎙

Therefore, using powerful Tailwind CSS's utility classes, we can build a complex customized layout to place our animated skeleton such as grid, flexbox, etc. Below is a simple grid layout with our fancy skeleton boxes. Here is the link to the [**Live Tailwind Playground**](https://play.tailwindcss.com/cDmb22zdo6) to play around with our final build.

{{< figure src="assets/tailwind7.gif" >}}

```html
<div class="flex min-h-screen items-center justify-center bg-gray-900">
  <div class="grid w-[600px] grid-cols-3 gap-6">
    <!-- First Card Begins Here -->
    <div class="relative space-y-5 overflow-hidden rounded-2xl bg-white/5 p-4 shadow-xl shadow-black/5 before:absolute before:inset-0 before:-translate-x-full before:animate-[shimmer_2s_infinite] before:border-t before:border-rose-100/10 before:bg-gradient-to-r before:from-transparent before:via-rose-100/10 before:to-transparent">
      <div class="h-24 rounded-lg bg-rose-100/10"></div>
      <div class="space-y-3">
        <div class="h-3 w-3/5 rounded-lg bg-rose-100/10"></div>
        <div class="h-3 w-4/5 rounded-lg bg-rose-100/20"></div>
        <div class="h-3 w-2/5 rounded-lg bg-rose-100/20"></div>
      </div>
    </div>
    <!-- First Card Ends Here -->
    <!--Second Card Begins Here  -->
    <div class="relative space-y-5 overflow-hidden rounded-2xl bg-white/5 p-4 shadow-xl shadow-black/5 before:absolute before:inset-0 before:-translate-x-full before:animate-[shimmer_2s_infinite] before:border-t before:border-rose-100/10 before:bg-gradient-to-r before:from-transparent before:via-rose-100/10 before:to-transparent">
      <div class="h-24 rounded-lg bg-rose-100/10"></div>
      <div class="space-y-3">
        <div class="h-3 w-3/5 rounded-lg bg-rose-100/10"></div>
        <div class="h-3 w-4/5 rounded-lg bg-rose-100/20"></div>
        <div class="h-3 w-2/5 rounded-lg bg-rose-100/20"></div>
      </div>
    </div>
    <!-- Second Card Ends Here -->
    <!-- Third Card Begins Here  -->
    <div class="relative space-y-5 overflow-hidden rounded-2xl bg-white/5 p-4 shadow-xl shadow-black/5 before:absolute before:inset-0 before:-translate-x-full before:animate-[shimmer_2s_infinite] before:border-t before:border-rose-100/10 before:bg-gradient-to-r before:from-transparent before:via-rose-100/10 before:to-transparent">
      <div class="h-24 rounded-lg bg-rose-100/10"></div>
      <div class="space-y-3">
        <div class="h-3 w-3/5 rounded-lg bg-rose-100/10"></div>
        <div class="h-3 w-4/5 rounded-lg bg-rose-100/20"></div>
        <div class="h-3 w-2/5 rounded-lg bg-rose-100/20"></div>
      </div>
    </div>
    <!-- Third Card Ends Here -->
  </div>
</div>

```

I hope you found this article useful and that it serves as a good starting point  (*not only for developers but also for Designers*) for creating various skeleton loading screens. If you found this article insightful, do share it with your friends and network. 

{{< admonition type=warning title="Before you leave 📌" open=true >}}
Do check out some amazing skeleton loading packages I have found for [React](https://blog.openreplay.com/3-ways-to-implement-skeleton-components-in-react#heading-what-is-a-skeleton-component), [Angular](https://openbase.com/categories/js/best-angular-loading-skeleton-libraries) and [Vue](https://openbase.com/categories/js/best-vue-loading-skeleton-libraries). Thanks for reading, and keep creating amazing UI with Tailwind💨! 
{{< /admonition >}}