# JAMStack Personal Blog Project

This project will create a blog that uses static files. I'm making it using the [tutorial by Kevin Powell](https://github.com/mattfussell/new-site.git). It uses [11ty](https://www.11ty.dev/), the documentation for which can be found [here](https://www.11ty.dev/docs/).


As one does with lab/learning exercises, I started it, messed stuff up, will be fixing it with this project.

As I use multiple OSs across my devices through the day, a big part of getting this project off the ground was figuring out how to sandbox my development environment. This particular project uses NodeJS in a Ubuntu VM, so the notes are going to apply to how I achieve things in that space.

## Setup Instructions

This project uses NodeJS and the tutorial uses NPM - make sure those are installed in your environment. The tutorial also assumes you'll be using VSCode, so make sure that's installed. The following modules are recommended for quality of life improvements:

* Live Server - Ritwick Dey
* Live Sass Compiler - Glenn Marks
* Bracket Pair Colorizer - CoenraadS

You may also want to change your tabs from 4 spaces to 2 ... 

### Super Hand Shortcuts

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