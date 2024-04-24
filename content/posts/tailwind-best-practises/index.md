+++
title = "How to TailWind CSS like a Pro"
publishDate = 2024-04-23T00:00:00+05:45
lastmod = 2024-04-23T11:24:47+05:45
tags = ["tailwind", "reactjs"]
categories = ["frontend", "UI"]
draft = false
featuredImage = "assets/tailwind-wizard.jpg"
featuredImagePreview = "./posts/tailwind-best-practises/assets/tailwind-surfer.jpg"
+++


In this article, I’ll share some best practices for working with Tailwind CSS, which I usually prefer to follow and have learnt over a period of time.
By following these tips, you can keep your application **clean**, **organized**, and **maintainable**, even as your project grows.

1\. Utilize Design Tokens 🪙
------------------------------------------------------
Design tokens are like special containers that hold all the important things that make your designs look good, like colors, sizes, fonts, and where things break on a screen. They help you make sure everything looks the same across all your designs and makes it easy to change things later on. In the `tailwind.config.js` file, you can create special codes called **design tokens**. It's a smart way to keep all your design codes in one place so that all your special designs can be used by all your different class names in Tailwind CSS.

{{<figure src="assets/tailwind-color-palette.webp" title="Tailwind Built-in Color Palette">}}
Ensure that you assign meaningful semantic names to all the colors used in your project, [replacing or extending the default color palette](https://tailwindcss.com/docs/customizing-colors#using-custom-colors).

For multi-theme support, opt to [define your colors as CSS variables](https://tailwindcss.com/docs/customizing-colors#using-css-variables) instead of within your Tailwind configuration. This approach enables you to associate multiple colors with the same semantic name, depending on the selected theme.

{{<highlight javascript "lineNos=inline">}}
//<!-- tailwind.config.js -->
import defaultTheme from "tailwindcss/defaultTheme";

const config = {
  content: [
    "./src/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {
      fontFamily: {
        manrope: "var(--font-manrope)",
      },
      colors: {
        ...defaultTheme.colors,
        theme: "#7743DB",
        light: {
          primary: "#FFFFFF",
          secondary: "#f1f1ef",
        },
        dark: {
          primary: "#0F0F0F",
          secondary: "#202020",
        },
        "background": "#F5F5F5",
      },
      screens: {
        ...defaultTheme.screens,
        xs: "340px",
        sm: "420px",
      },
      spacing: {
        spacing: {
          ...defaultTheme.spacing,
          1: '5px',
          2: '10px',
          3: '15px',
          4: '20px',
          5: '25px'
        }
      }
    },
  },
};

export default config;
{{< /highlight >}}


2\. Make styles reusable 🎨
-----------------------------------------------
Tailwind works best with [React](https://react.dev/), [Vue](https://vuejs.org/), or [Svelte](https://svelte.dev/) because it helps make components easier to use and reuse. It can still be used with regular `HTML`, but it might lead to a lot of repeated code. The Tailwind Docs suggest using a [special editing technique](https://tailwindcss.com/docs/reusing-styles#multi-cursor-editing) to avoid this, but it can be tricky and you have to be careful to select all the right parts.

With a component-based framework like React, you can create a part of your website once and use it many times in different places.

{{<highlight javascript>}}
// Define a reusable Button component using Tailwind utility classes
const Button = ({ children, onClick }) => {
  return (
    <button
      onClick={onClick}
      className="bg-primary hover:bg-primary-alt text-primary-contrast py-2 px-4 rounded"
    >
      {children}
    </button>
  );
};

// Use (and re-use) the Button component without having to duplicate Tailwind classes
const App = () => {
  return (
    <div>
      <Button onClick={() => console.log('Clicked!')}>Click Me</Button>
      <Button onClick={() => console.log('Clicked too!')}>Click Me Too</Button>
    </div>
  );
};
{{</highlight>}}



3\. Use arbitrary values rarely 🙅‍♀️
--------------------------------

Consider a web application that adheres to a specific color scheme. For instance, we've chosen `#7743DB` as our theme color and `#0D0D0D` as our background color.

To maintain consistency and improve readability, we can add these colors to our **Tailwind CSS** configuration. Instead of using arbitrary values directly in our HTML or CSS, we'll create semantic class names. For example:
- `bg-background` for the background color
- `text-theme` for the theme color

{{<highlight javascript >}}
//<!-- tailwind.config.js -->
import defaultTheme from "tailwindcss/defaultTheme";

const config = {
  content: [
    "./src/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {
      colors: {
        ...defaultTheme.colors,
        theme: "#7743DB",
        background: "#0D0D0D"
      },
    },
  },
};
export default config;
{{</highlight>}}

Now, if we decide to change our application's color scheme, we only need to update the `tailwind.config.js` file. This approach avoids the hassle of renaming arbitrary class names scattered throughout our codebase.

4\. Handle dynamic classes with care 🧐
----------------------------------------------------

You all might have encountered this issue while working with dynamic classes that whenever we apply some dynamic class name based on state or some condition, the class name appears in elements panel on the browser, but its corresponding CSS does not.

This is because Tailwind scans your source code for classes using [regular expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions) to extract every string that could possibly be a class name. He nce, any broken class name string such as `border-[${borderColor}]` would not be recognised by tailwind at build time, and it would not be included in the output CSS file of tailwind.

Suppose, we have to change the border colour of our element based on the colour code passed in props. There are two ways to it:

1.  Defining a separate class name for each state value. This is only applicable if you know all the expected values of the `borderColor` at build time.

> Note: [**clsx**](https://github.com/lukeed/clsx#readme) is utility package for constructing `className` strings conditionally.

{{<highlight javascript >}}
//<!-- BorderBox.jsx -->
const BorderBox = ({ borderColor }) => {
  return (
    <div
      className={clsx("border border-solid", {
        "border-black": borderColor === "black",
        "border-red-500": borderColor === "red",
        "border-green-500": borderColor === "green",
      })}
    >
      Some Random Text
    </div>
  );
};
{{</highlight>}}

2\. If we do not know all the expected value of `borderColor` at build time, it is better to pass the border colour in the style attribute of the element to support unknown values.

{{<highlight javascript >}}
//<!-- BorderBox.jsx -->
const BorderBox = ({ borderColor }) => {
  return (
    <div className="border border-solid" style={{ borderColor }}>
      Some Random Text
    </div>
  );
};
{{</highlight>}}

5\. Accessing CSS design tokens in JavaScript 🧲
-----------------------------------------------------------------

In Web applications, there are multiple scenarios where you need to access a CSS design token value in JavaScript. When such situations arise, developers often hardcode the CSS design token directly into their code while working with Tailwind CSS. However, this practice is not ideal. If you later modify the design token value in your Tailwind configuration, your code might still reference the old value, potentially leading to unintended behavior.

To address this, consider creating a custom utility function that reads the Tailwind configuration dynamically within your codebase.

{{<highlight javascript >}}
//<!-- tailwind.config.js -->
import resolveConfig from "tailwindcss/resolveConfig";
import tailwindConfig from "../tailwind.config";

const getTailwindConfiguration = () => {
  return resolveConfig(tailwindConfig).theme;
};

const config = getTailwindConfiguration();

console.log(config.colors.red[500]); // would print #ef4444
{{</highlight>}}

6\. Tailwind Plugins 🧩
----------------------------------------------------------------

Tailwind provides us with [plugins](https://tailwindcss.com/docs/plugins#overview) to register new styles to inject into the user’s stylesheet using JavaScript instead of writing custom CSS styling in stylesheets.

I find this approach better, as writing custom CSS classes means you're essentially rewriting CSS and waving goodbye to Tailwind’s organized workflow and simple maintenance.

Suppose we want to create a `.header` class which has several styles attached to it. This is how we can achieve it:

{{<highlight javascript >}}
//<!-- tailwind.config.js -->
import plugin from "tailwindcss/plugin";

const config = {
  content: [
    "./src/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  plugins: [
    plugin(function ({ addComponents }) {
      addComponents({
        ".header": {
            fontWeight: "600",
            height: "3rem",
            borderRadius: ".25rem",
            backgroundColor: "#f5f5f5",
            boxShadow: "0 2px 4px 0 rgba(0,0,0,.1)",
            display: "flex",
            justifyContent: "space-between",
            alignItems: "center",
            "&:hover": {
              backgroundColor: "#f5f5f5",
            },
        },
      });
    }),
  ],
};

export default config;
{{</highlight>}}

> This is how the `.header` class would look like when you hover over it:

{{<figure src="assets/tailwind-plugin.png" title="Custom Tailwind Plugin for Complex styles">}}

## Final Words 🎤
While Tailwind CSS provides a strong and versatile utility-first approach to styling web apps, it's important to avoid common errors that can jeopardize the maintainability and scalability of your code. By carefully integrating Tailwind with a **uniform design system**, embracing **component-based architecture**, and establishing a library of reusable components, you can create clean, consistent, and highly maintainable user interfaces.