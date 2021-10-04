# JAMStack Personal Blog Project

This project will create a blog that uses static files. I'm making it using the [tutorial by Kevin Powell](https://github.com/mattfussell/new-site.git). It uses [11ty](https://www.11ty.dev/), the documentation for which can be found [here](https://www.11ty.dev/docs/).


As one does with lab/learning exercises, I started it, messed stuff up, will be fixing it with this project.

As I use multiple OSs across my devices through the day, a big part of getting this project off the ground was figuring out how to sandbox my development environment. This particular project uses NodeJS in a Ubuntu VM, so the notes are going to apply to how I achieve things in that space.

## Setup Instructions

### Establish the Environment

This project uses NodeJS and the tutorial uses NPM - make sure those are installed in your environment. The tutorial also assumes you'll be using VSCode, so make sure that's installed. The following modules are recommended for quality of life improvements:

* Live Server - Ritwick Dey
* Live Sass Compiler - Glenn Marks
* Bracket Pair Colorizer - CoenraadS
* Nunjucks Template - eseom

You may also want to change your tabs from 4 spaces to 2.

To enable emmet autocomplete of nunjucks, hit `F1` and search for 'preferences', and select the option that includes 'Open Settings (JSON)'. Aside from your other preferences, make sure that the section for emmet detailed below is included:

```json
{
    "editor.tabSize": 2,
    "workbench.colorTheme": "Monokai",
    "explorer.confirmDelete": false,
    "emmet.includeLanguages": {
        "nunjucks": "html",
        "njk": "html"
      }
}
```

**Terminal Control**

* `CTRL+SHIFT+\``: opens up a new terminal

**Init NPM**

Type the following to init NPM:

```bash
npm init -y
```

The `-y` flag tells it that we are going to accept the defaults. This will give us a `package.json` file in the root directory. Next, install **11ty**:

```bash
npm install @11ty/eleventy --save-dev
```

`--save-dev` is a dependecy for the development process, but it's not something that will be public-facing. This will be installed in the newly created `node_modules` folder, which is blocked in .gitignore. When cloning this to a new development machine, you will have to re-run the installer.

Next, `package.json` needs to have have some code updated as follows:

```json
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1"
},
```

**changed to**

```json
"scripts": {
  "start": "eleventy --serve",
  "build": "eleventy"
},
```

Next, eleventy needs a configuration file. Create the following new file in the root of the directory:

`.eleventy.js`

Add the following code to the `.eleventy.js` file:

```javascript
module.exports = function(eleventyConfig) {
  return {
    dir: {
      input: "src",
      output: "public"
    }
  };
}
```

To test your current progress, enter `npm start` into the terminal. Use `CTRL+C` in the terminal to stop the process once you've confirmed the site works. This should have generated a `public` folder in the root directory, but some additional configuration changes will be necessary to get all the parts working as expected.

Inside the `.eleventy.js` file, add the following above the `return` statement insde the `module.exports` function expression:

```javascript
eleventyConfig.addPassthroughCopy('./src/style.css');
eleventyConfig.addPassthroughCopy('./src/assets');
```

Run `npm start` to test.

### Convert Static Files to Templates

If you haven't already done so, stop the server, then delete the `/public` folder to avoid conflicts.

Inside the `/src` folder, create a new folder called `_includes`. Inside, create a file called `base.njk`. This is a [Nunjucks](https://mozilla.github.io/nunjucks/) file - the documentation for Ninjucks can be found [here](https://mozilla.github.io/nunjucks/getting-started.html).

Building out templates is similar to other CMS implementations - nothing new here if you've worked with them before (CodeIgniter/WordPress/Django/etc.) 

Type `!` then hit the `Tab` key to autofill the basic HTML template in the `base.njk` file.

Update the `<title>Document</title>` to `<title>{{ title }}</title>`.

Inside the `<body>` tags, add `{{ content | safe }}`.

Next, rename the `index.html` file in the `/src` folder to be `index.njk`. Delete the `<!DOCTYPE>` tag, the opening and closing `<html>` and `<body>` tags, and delete the opening and closing `<head>` tags, as well as everything inside the `<head>` tags.

At the top of the `index.njk` file, insert the following:

```nunjucks
---
title: example
layout: 'base.njk'
---
```

This will allow you to use **front matter** to add meta information to the top of templates. Run `npm start` to verify that the template are loading - note that they're probably going to be ugly at the moment.

It's safe to leave the node live server running while making changes. Go back to `base.njk` and add a link to the style sheet - use `/style.css` for the `href` - and note that when you save the `base.njk` file, it automatically updates the server. When you check the web browser, things should look a lot better.

Next, go back to the `/_includes` folder and create a new file called `header.njk`. Cut the `<header>` tags and their content from the `index.njk` file and paste them into the new `header.njk` file and save. Go back to the `base.njk` file, and above `{{ content | safe }}`
type `{% include 'header.njk'%}` and save. Repeat this process for creation of `footer.njk`, and include it in the `base.njk` file after the content section.

Rename `all-articles.html` to be `blog.njk`. Follow the same steps for removing tags and adding front matter as was done for the `index.njk` file. Use the following for the front matter for this file:

```nunjucks
title: Blog
layout: 'base.njk'
```

Add `<main>` tags around the `{{ content | safe }}` in the base files, and remove the `<main>` tags from `index.njk` and `blog.njk`.

### Build Blog Functionality

Open up `blog.njk`. Inside, only keep the first `<li>` - delete the rest of the entries.

Inside the `/blog` folder, create a file called `blog.json` that contains the following:

```json
{
  "tags": "post"
}
```

Comment out the `li` so that only the `<div>`, `<ul>`, and the `<h1>` remain in the `blog.njk` file using `{# #}`.

We're going to use a for loop to leverage the power of tags; again, this works very similarly to other CMSs. Insert the following inside the `<ul>`:

```nunjucks
  {%- for post in collections.post | reverse -%} {# 'reverse' puts the newest post first #}
  <li>
    <article class="snippet">
      <img src="{{ post.data.image }}" alt="{{ post.data.imageAlt }}" class="snippet__img">
      <h3 class="snippet__title"><a href="{{ post-url}}">{{ post.data.title }}</a></h3>
      <p class="snippet__meta">by <span>{{ post.data.author }}</span> on <span>{{ post.data.date }}</span></p>
      <p class="snippet__body">{{ post.data.description }}</p>
      <a href="{{ post.url }}" class="btn btn--primary">Continue Reading</a>
    </article>
  </li>

  {%- endfor -%}
```

Stop the server, and open `.eleventy.js`. Above the `module.exports` function expression, create a variable using `const` called `{ DateTime }` with a value of `require("luxon");`. Inside the `module.exports` function expression, add the following:

```javascript
eleventyConfig.addFilter("postDate", (dateObj) => {
  return DateTime.fromJSDate(dateObj).toLocaleString(DateTime.DATE_MED);
})
```

Return to the `blog.njk` file, and on the author line, modify `{{ post.data.date }}` to be `{{ post.data.date | postDate }}`.
