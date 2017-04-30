# Webpack Sass / Scss compiling to separate file

Webpack is an amazing tool for transpiling and bundling JavaScript, but it can also take care of compiling Sass or Scss to static files.

[**get the code on github**](https://github.com/JonathanMH/webpack-scss-sass-file)

I came across this issue while developing a prototype and not a single page app, that I needed to have a `.scss` file include some other files and output a `.css` file. I didn't need inlined and scoped CSS like one would probably do with a single page app.

This took me surprisingly long to figure this out, mostly because there is a lot of varying information out there and not a lot of good examples that work with the up to date versions of webpack 2 RC 4 and the ExtractTextPlugin.

First of all, I recommend you globally install a recent webpack version:

```
npm -g install webpack@2.2.0-rc.4
```

Inside your project you need to install the following dependencies, remember to append the `--save` flag if you want them to be written to your `package.json`:

```
npm install css-loader node-sass sass-loader webpack@2.2.0-rc.4
```

Now let's say we have a project structure for a simple one-page that looks like the following (identical to the example project):

```
├── app.js
├── package.json
├── scss
│   ├── about
│   │   └── about.scss
│   └── main.scss
├── webpack.config.js
```

* `app.js` contains all our cool JavaScript code
* `package.json` defines our dependencies
* `scss/main.scss` is our scss / sass entry point
* `webpack.config.js` is our config that tells webpack what to do


If you have cloned [the example repository](https://github.com/JonathanMH/webpack-scss-sass-file), after running the build with `webpack` it should look like the following:

```
├── app.js
├── dist
│   ├── bundle.js
│   └── main.bundle.css
├── package.json
├── scss
│   ├── about
│   │   └── about.scss
│   └── main.scss
├── webpack.config.js
```

Note: Frequently developers use `npm run dev` or `npm run build` to execute the `webpack` command.

Notice that the directory `dist` including the files `bundle.js` and `main.bundle.css` have been added. The latter contains the compiled SCSS from `main.scss` and `about.scss`.

## Webpack SCSS / SASS config

Typically [webpack][] processes everything in an input file and what ever is *required* by that file or other files that are required from there. It builds a tree of requirements, scoops up all the JavaScript and other resources and compiles them for you. If you want to just have it look for a file, we need to use the [ExtractTextPlugin][] to process it.

Have a look at the config for this project below:

```javascript
var ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
  entry: ['./app.js', './scss/main.scss'],
  output: {
    filename: 'dist/bundle.js'
  },
  module: {

    rules: [
      /*
      your other rules for JavaScript transpiling go in here
      */
      { // regular css files
        test: /\.css$/,
        loader: ExtractTextPlugin.extract({
          loader: 'css-loader?importLoaders=1',
        }),
      },
      { // sass / scss loader for webpack
        test: /\.(sass|scss)$/,
        loader: ExtractTextPlugin.extract(['css-loader', 'sass-loader'])
      }
    ]
  },
  plugins: [
    new ExtractTextPlugin({ // define where to save the file
      filename: 'dist/[name].bundle.css',
      allChunks: true,
    }),
  ],
};
```

The config that enables us to compile SCSS/SASS from a file instead of requiring it in our JavaScript source is the `loaders` that refer to `ExtractTextPlugin` and the `plugin` that specifies where to write the file.

Also note that in our entry points, we specify that `./scss/main.scss` should be read and then run through the rules (in this case `.scss` would match).

## Transpiled SCSS file

Congratulations, now our `.scss` files that used to look like this:

**main.scss**
```css
// include another .scss file from a sub-directory
@import './about/about.scss';

body {
    a {
        color: magenta;
    }
}
```

**about/about.scss**
```css
body {
    p {
        line-height: 1.5;
    }
}
```

Should be transpiled into a plain css file:

```css
body p {
  line-height: 1.5; }

body a {
  color: magenta; }
```

Thank you so much for reading this post, I hope it was useful to you! Consider following me on [twitter](https://twitter.com/JonathanMH_com), [feedly](http://cloud.feedly.com/#subscription%2Ffeed%2Fhttp%3A%2F%2Fjonathanmh.com%2Ffeed) or [instagram](https://www.instagram.com/gegenwind.dk/)!

If anything doesn't work, please leave a comment and I'll have a look!

[webpack]: https://webpack.js.org/
[ExtractTextPlugin]: https://github.com/webpack/extract-text-webpack-plugin
