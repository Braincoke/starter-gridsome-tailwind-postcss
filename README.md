# Gridsome, Tailwind, and other PostCSS

In this post I'll describe how to setup PostCSS plugins to work with Tailwind and Gridsome.

We start by creating a new project with  `gridsome create gridsome-tailwind-postcss`.

We will start by setting up Tailwind, move on to add PurgeCSS and finally configure other interesting PostCSS plugins to work with everything.

## Setup Tailwind

The Gridsome documentation already does a good job at explaining [how to add Tailwind](https://gridsome.org/docs/assets-css/#add-tailwindcss-manually). This part of the post is based on this documentation and provided here so you don't have to go back and forth between multiple sources.

The steps to setup Tailwind are :

1. Install Tailwind with npm or yarn

2. Create a `main.css` file

3. Import the file in the project

4. Create a `tailwind.config.js` file

5. Update `gridsome.config.js`

### 1. Install Tailwind

```bash
npm install tailwindcss
```

### 2. main.css

Create a file `main.css` in the `/src/css` directory with the following content:

```css
@tailwind base;

@tailwind components;

@tailwind utilities;
```

### 3. Import main.css

In `main.js` import the newly created css file:

```js
import '~/css/main.css'

import DefaultLayout from '~/layouts/Default.vue'

export default function (Vue, { router, head, isClient }) {
  // Set default layout as a global component
  Vue.component('Layout', DefaultLayout)
}
```

### 4. Tailwind configuration file

Create the TailwindCSS configuration file in `/src`. You can generate it with:

```bash
npx tailwind init
```

This generates a minimal `tailwind.config.js` at the root of your project:

```js
module.exports = {
    theme: {
        extend: {}
    },
    variants: {},
    plugins: []
}
```

### 5. Update gridsome.config.js

We need to inform Gridsome that we are using TailwindCSS. To do so we add an import to TailwindCSS and update the configuration `css.loaderOptions.postcss`.

```js
const tailwind = require("tailwindcss");

const postcssPlugins = [tailwind()];

module.exports = {
  siteName: "Gridsome",
  plugins: [],
  css: {
    loaderOptions: {
      postcss: {
        plugins: postcssPlugins
      }
    }
  }
};
```

### 6. Testing our config

In the existing `src/pages/Index.vue` we add a button styled with TailwindCSS to verify that the setup is working properly :

```html
<button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
 Button
</button>
```

Then we start (or restart) the server with `gridsome develop`.

## Setup PurgeCSS

PurgeCSS allows us to remove unused CSS. It is particularly powerful when using Tailwind. To setup PurgeCSS the steps are:

1. Install PurgeCSS

2. Create a `purgecss.config.js` file

3. Update `gridsome.config.js`

### 1. Install PurgeCSS

```bash
npm install -D @fullhuman/postcss-purgecss
```

### 2. PurgeCSS config file

Create a file named `purgecss.config.js` in the root of the project with the configuration below:

```js
module.exports = {
    content: [
        './src/**/*.vue',
        './src/**/*.js',
        './src/**/*.jsx',
        './src/**/*.html',
        './src/**/*.pug',
        './src/**/*.md',
    ],
    whitelist: [
        'body',
        'html',
        'img',
        'a',
        'g-image',
        'g-image--lazy',
        'g-image--loaded',
    ],
    extractors: [
        {
            extractor: content => content.match(/[A-z0-9-:\\/]+/g),
            extensions: ['vue', 'js', 'jsx', 'md', 'html', 'pug'],
        },
    ],
}
```

### 3. Update the Gridsome configuration

Update `gridsome.config.js` to import `purgecss` and add it to the postcss plugins.

Start by adding purgecss:

```js
const purgecss = require('@fullhuman/postcss-purgecss')
```

You only really need purgeCSS in production, to add the plugin conditionnaly you can use the value of `process.env.NODE_ENV` and check if it is equal to `'production'`.

The conditionnal import looks like this:

```js
if (process.env.NODE_ENV === 'production') postcssPlugins.push(purgecss(require('./purgecss.config.js')))
```

Your file should now look like this:

```bash
const tailwind = require('tailwindcss')
const purgecss = require('@fullhuman/postcss-purgecss')

const postcssPlugins = [
	tailwind(),
]

if (process.env.NODE_ENV === 'production') postcssPlugins.push(purgecss(require('./purgecss.config.js')))

module.exports = {
    siteName: 'Gridsome',
    plugins: [],
    css: {
        loaderOptions: {
            postcss: {
                plugins: postcssPlugins,
            },
        },
    },
}
```

Great! PurgeCSS is now  completely set up !

### CSSNext

One interesting PostCSS plugin is [CSSNext](https://cssnext.github.io/) which gives us the ability to use future CSS specification today. Think of it of the Babel of CSS.

### Install CSSNext

```console
 npm install postcss-cssnext
```

### Update the Gridsome config

We add the PostCSS plugin to the Gridsome config as usual:

```js
const tailwind = require("tailwindcss");
const cssnext = require("postcss-cssnext")
const purgecss = require("@fullhuman/postcss-purgecss");

const postcssPlugins = [cssnext(), tailwind()];

if (process.env.NODE_ENV === "production")
  postcssPlugins.push(purgecss(require("./purgecss.config.js")));

module.exports = {
  siteName: "Gridsome",
  plugins: [],
  css: {
    loaderOptions: {
      postcss: {
        plugins: postcssPlugins
      }
    }
  }
};

```

The order in which we load the plugins is important. Tailwind is not capable of reading next generation CSS so we need to transform it for Tailwind first.

### Testing our setup

If you are using VSCodium (or VSCode), you can use the plugin [PostCSS Language Support](https://marketplace.visualstudio.com/items?itemName=csstools.postcss) to get syntax highlighting in your CSS files.

Let's test some features of CSSNext in our basic project. In `main.css` add some css between `@tailwind components` and `@tailwind utilities`:

```css
@tailwind base;
@tailwind components;

:root {
    --mainColor: red;
  }

button {
  & a {
      color: var(--mainColor)
  }

  @nest &.custom {
      background-color: pink
  }
}
@tailwind utilities;

```

We want `@tailwind utilities` last to avoid overriding tailwind utilities.

Here we use a variable `--mainColor` set to `red`, and some nesting.

Let's update our `Index.vue` to make use of our new CSS:

```html
<button class="custom font-bold py-2 px-4 rounded"><a>Button</a></button>
```

Now if we start our server we should see a pink button with a red text. We managed to use CSSNext with tailwind!

### Importing files

Now let's say we have created multiple themes and split them across different files to better structure our project, our css folder could look like this:

```bash
main.css
_base.css
_theme-pink.css
_theme-blue.css
```

If we want to import the css from `_theme-pink.css` into our project we can simply use an `@import` statement in our `main.css` file:

```css
@tailwind base;
@tailwind components;

@import "./_theme-pink.css";

@tailwind utilities;
```
