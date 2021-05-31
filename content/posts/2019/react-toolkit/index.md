---
title: React Toolkit
date: '2019-01-02'
spoiler: Setting up build toolchain for React
---

This tutorial assumes that the reader has a basic understanding of the command line and node. This will go through creating a toolchain yourself rather than relying on `create-react-app`. Some portions aren't explained as they are beyond this simple tutorial.

### Necessary Tools

- [Node](https://nodejs.org/en)
- [Yarn](https://yarnpkg.com/en/docs/install)
- [React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)

### Tutorial

Create a new folder called `helloworld`. This can be done with Explorer on Windows or the shell on OSX / Linux. For example, I would create it in my `src` directory where I store all source code.

#### Folders

**OSX / Linux**:

```shell
$ mkdir -pv ~/src/react-tutorials/helloworld
$ cd ~/src/react-tutorials/helloworld
```

**Windows**: Create the folders in explorer and then open your `Node Command Prompt` and navigate to your new source directory.

Inside the folder `helloworld`, create the following folders:

```text
src/
public/
```

#### Versioning

It would be a fantastic idea to setup git before starting!

```shell
$ git init
```

Create a `.gitignore` file that excludes `node_modules`.

**`.gitignore`**

```text
node_modules
```

#### The Toolchain

We will now create a basic `package.json` file. This allows `yarn` to handle the package dependencies. The scripts section contains programs that we do not have installed yet. The duplicated `--mode` and `--env.NODE_ENV` lines are intentional, this ensures that the developer has access to a `process.NODE_ENV` variable inside the app. This is not used in this tutorial, but it's nice to have for the future.

**`package.json`**

```json
{
  "name": "HelloWorld",
  "private": true,
  "scripts": {
    "build": "webpack --mode=production --env.NODE_ENV=production --config ./webpack.config.js",
    "start": "webpack-dev-server --hot --mode=development --env.NODE_ENV=development --config ./webpack.config.js"
  },
  "version": "1.0.0"
}
```

We now need to add `babel`, this is used to [transpile](https://en.wikipedia.org/wiki/Source-to-source_compiler) the ES6 / JSX we write into a form compatible with most browsers.

```shell
$ yarn add --dev @babel/preset-react @babel/core @babel/cli @babel/preset-env
```

Now create a file called `babel.config.js`, this file is used by `babel` to determine how to [transpile](https://en.wikipedia.org/wiki/Source-to-source_compiler) your js files.

**`babel.config.js`**

```jsx
const plugins = [
  [
    'react-hot-loader/babel'
  ]
];

const presets = [
  [
    '@babel/env',
    {
      targets: {
        edge: '17',
        firefox: '60',
        chrome: '67',
        safari: '11.1',
      },
      useBuiltIns: 'usage',
    },
  ],
  '@babel/preset-react'
];

module.exports = {
  plugins,
  presets
};
```

We now have our code [transpiling](https://en.wikipedia.org/wiki/Source-to-source_compiler), but we have no way to use it in the browser yet! Let's install `webpack` to fix that.

```shell
$ yarn add --dev webpack webpack-cli webpack-dev-server babel-loader css-loader style-loader
```

Notice the `webpack.config.js` mentioned in the `package.json` file, we have to create that file.

**`webpack.config.js`**

```jsx
const path = require('path');
const webpack = require('webpack');

module.exports = (env, argv) => {
  return {
    devServer: {
      contentBase: path.join(__dirname, 'public/'),
      hotOnly: true,
      port: 3000,
      publicPath: 'http://localhost:3000/dist/'
    },
    devtool: 'none',
    entry: './src/index.js',
    module: {
      rules: [
        {
          test: /\.jsx?$/,
          exclude: /node_modules/,
          loader: 'babel-loader'
        },
        {
          test: /\.css$/,
          use: [ 'style-loader', 'css-loader' ]
        }
      ]
    },
    output: {
      filename: 'bundle.min.js',
      path: path.join(__dirname, 'dist/'),
      publicPath: '/dist/'
    },
    plugins: [
      new webpack.DefinePlugin({
        'process.env': {
          'NODE_ENV': `'${env.NODE_ENV}'`
        }
      })
    ],
    resolve: {
      extensions: [ '.webpack.js', '.web.js', '.js', '.jsx' ]
    }
  }
};
```

We will finally install `react` proper! We are also including `react-hot-loader` so our development server will automatically reload on any source changes we make.

```shell
$ yarn add react react-dom react-hot-loader
```

#### The App

We are nearly ready to launch a basic html page, we should now create the `index.html` file that will be loaded.

**`public/index.html`**

```html
<!doctype html>
<html lang='en-us'>
  <head>
    <title>Hello World!</title>
    <meta charset='utf-8' />
    <meta name='viewport' content='width=device-width, initial-scale=1.0, shrink-to-fit=no' />
    <script defer src='../dist/bundle.min.js'></script>
  </head>
  <body>
    <div id='root'></div>
  </body>
</html>
```

We are now ready to create our first react files. We will start with the entry file at `src/index.js`.

**`src/index.js`**

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

import App from './App.js';

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

Now we create the `App.js` file that will be loaded by `index.js`.

**`src/App.js`**

```jsx
import React, { Component } from 'react';
import { hot } from 'react-hot-loader/root';

class App extends Component {
  render() {
    return (
      <div>
        Hello World!
      </div>
    );
  }
}

export default hot(App);
```

We can now start our development server with:

```shell
$ yarn start
```

Then open Google Chrome and point to `localhost:3000`.

You should see `Hello World!` displayed. If you change this to say `Jello World!` you will see the text on the screen reflect this.

End with adding this all to git and giving a descriptive commit message.

```shell
$ git add *
$ git add .gitignore
$ git commit -am 'My cat is named Gracie!'
```

