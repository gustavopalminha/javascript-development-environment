
# Building a JavaScript Development Environment

## Reference links


Course: https://app.pluralsight.com/library/courses/javascript-development-environment/table-of-contents

Git: https://github.com/coryhouse/javascript-development-environment

Interesting links:
* https://www.lvguowei.me/post/building-js-dev-env/
* https://blog.hellojs.org/setting-up-your-react-es6-development-environment-with-webpack-express-and-babel-e2a53994ade

# Editor and Configuration

First of all, editor of choice here is surprise surprise [VS Code](https://code.visualstudio.com/). I’m actually happly surprised that [Erich Gamma](https://github.com/egamma) is behind this.

Use [EditorConfig](https://editorconfig.org/) to manage, well, editor configurations. Tabs VS spaces, etc. Note that VS Code need to install a plugin for it to work.

The example `.editorconfig` file:
```
root = true

[*]
indent_style = space
indent_size = 2
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
charset = utf-8

[*.md]
trim_trailing_whitespace = true
```
# Package Management

## Package Manager

We will use node > 6, install it:
```
sudo apt install nodejs
```
The choice of our package manager is [npm](https://www.npmjs.com/).
If necessary to install it using terminal, run `sudo apt install npm`.
The `package.json` has all the packages that will be used in the course.
This file can be created using `npm init`.
Make sure your `package.json` has the following content (adjust some properties if necessary):
```
{
  "name": "javascript-development-environment",
  "version": "1.0.0",
  "description": "JavaScript development environment Pluralsight course by Cory House",
  "scripts": {},
  "author": "Cory House",
  "license": "MIT",
  "dependencies": {
    "npm": "^6.0.1",
    "whatwg-fetch": "1.0.0"
  },
  "devDependencies": {
    "babel-cli": "6.16.0",
    "babel-core": "6.17.0",
    "babel-loader": "6.2.5",
    "babel-preset-latest": "6.16.0",
    "babel-register": "6.16.3",
    "chai": "3.5.0",
    "chalk": "1.1.3",
    "cheerio": "0.22.0",
    "compression": "1.6.2",
    "cross-env": "3.1.3",
    "css-loader": "0.25.0",
    "eslint": "3.8.1",
    "eslint-plugin-import": "2.0.1",
    "eslint-watch": "2.1.14",
    "express": "4.14.0",
    "extract-text-webpack-plugin": "1.0.1",
    "html-webpack-plugin": "2.22.0",
    "jsdom": "9.8.0",
    "json-schema-faker": "0.3.6",
    "json-server": "0.8.22",
    "localtunnel": "1.8.1",
    "mocha": "3.1.2",
    "nock": "8.1.0",
    "npm-run-all": "3.1.1",
    "nsp": "2.6.2",
    "numeral": "1.5.3",
    "open": "0.0.5",
    "rimraf": "2.5.4",
    "style-loader": "0.13.1",
    "webpack": "1.13.2",
    "webpack-dev-middleware": "1.8.4",
    "webpack-hot-middleware": "2.13.0",
    "webpack-md5-hash": "0.0.5"
  }
}

```
To install the packages, do `npm install`.
## Package Security
***Before continuing this topic: npm now has this node security built-in... This info will stay here for future reference.***

Since everyone can submit to npm, it is better to check for any vulnerabilities before using packages.

[https://nodesecurity.io/](https://nodesecurity.io/)

To install:

`npm install -g nsp`

To run:

`nsp check`

# Development Web Server

We are gonna use [Express](https://expressjs.com/) as our development web server.

Create the `/buildScript/srcServer.js`:

```javascript
var express = require('express');
var path = require('path');
var open = require('open');

var port = 3000;
var app = express();

app.get('/', function(req, res) {
  res.sendFile(path.join(__dirname, '../src/index.html'));
});

app.listen(port, function(err) {
  if (err) {
    console.log(err);
  } else{
    open('http://localhost:' + port);
  }
});

```

Then create the `/src/index.html`:

```html
<html>
  <body>
    <h1>Hello World!</h1>
  </body>
</html>

```

# Sharing Work In Progress

For expose your WIP to the world, let’s use [localtunnel](https://localtunnel.github.io/www/).

`npm install -g localtunnel`

Start your app.

`lt --port 8000`

# Automation

We choose [npm scripts](https://docs.npmjs.com/misc/scripts) here. Let’s try it out.

## npm scripts

First, write the script to start the dev server, in `package.json`:

```javascript
...
"scripts": {
    "start": "node buildScript/srcServer.js"
  },
...  
```

Now we can start the server by just `npm start`

## Pre/Post Hooks

Create `buildScript/startMessage.js`:

```javascript
var chalk = require('chalk');
console.log(chalk.green('Starting app in dev mode...'));
```

Then add `prestart` to the scripts:

```javascript
...
"scripts": {
    "prestart": "node buildScript/startMessage.js", 
    "start": "node buildScript/srcServer.js"
  },
...  
```

Now if we run `npm start`, the prestart script will be ran before the start script.

## Add Security Check and Share

We now create a script to run security check and share:

```javascript
...
"scripts": {
    "prestart": "node buildScript/startMessage.js", 
    "start": "node buildScript/srcServer.js",
    "security-check": "nsp check;exit 0",
    "share": "lt --port 3000"
  }, 
...  
```

Notice that in this way, `nsp` doesn’t need to be installed globally.

One thing that is not mentioned in the video is that if `nsp check` finds some vulnerabilities, it will fail npm. To fix this, we can just force it to always return 0.

## Concurrent Tasks

Run mutiple tasks in parallel:

```javascript
...
"start": "npm-run-all --parallel security-check open:src",
"open:src": "node buildScripts/srcServer.js",
...
```

# Transpiling

We choose [Babel](https://babeljs.io/) to _compile_ newer javascript code to older one.

Let’s first add the `.babelrc` to the root of our project:

```javascript
{
  "presets": [
    "latest"
  ]
}

```

This basically says use the latest JS.

Let’s try to use the module feature from ES6 in `startMessage.js`:

```javascript
import chalk from 'chalk';

```

Now in `package.json` we use `babel-node` instead of ´node´.

```javascript
...
  "prestart": "babel-node buildScripts/startMessage.js",
...

```

# Bundling

There are many bundling modules available, ex: AMD, UMD, CommonJS...

We should consider 2 visions:
* Node = CommonJS: `var jquery = require('jquery'); ...`
* Web = ES6 / ES2015 Modules: `import jQuery from 'jquery'; ...`

We choose ES6 Modules to be the module format and [webpack](https://webpack.js.org/) as the bundler. [^1]:
[^1]: There are 2 courses about them which are interesting at Pluralsight:  'Webpack Fundamentals' and 'Build Applications with React and Redux in ES6';

## Configure Webpack with Express

Now create a `webpack.config.dev.js` in the root:

```javascript
import path from 'path';

export default {
  debug: true,
  devtool: 'inline-source-map',
  noInfo: false,
  entry: [
    path.resolve(__dirname, 'src/index')
  ],
  target: 'web',
  output: {
    path: path.resolve(__dirname, 'src'),
    publicPath: '/',
    filename: 'bundle.js'
  },
  plugins: [],
  module: {
    loaders: [
      {test: /\.js$/, exclude: /node_modules/, loaders: ['babel']},
      {test: /\.css$/, loaders: ['style','css']}
    ]
  }
}
```

Then we need to configure Express to use it.

In `/buildScripts/srcServer.js`:

```javascript
...
import webpack from 'webpack'
import config from '../webpack.config.dev'

const port = 3000
const app = express()
const compiler = webpack(config)

app.use(require('webpack-dev-middleware')(compiler, {
  noInfo: true,
  publicPath: config.output.publicPath
}))
...
```

## Create an App Entry Point

First create `/src/index.js`:

```javascript
import numeral from 'numeral'

const courseValue = numeral(1000).format('$0,0.00')
console.log(`I would pay ${courseValue} for this course!`)
```
Edit `/src/index.html` by adding the following script tag:

```html
...
  <body>
    ...
    <script src="bundle.js"></script>
  </body>
...
```
Now start the app, you should see that `bundle.js` from the browser, it is not really a physical file but in memory.


## Handling CSS with Webpack

First let’s create `/src/index.css`:

```css
body {
  font-family: sans-serif;
}

table th {
  padding: 5px
}
```

Now we can _import_ this css into our `index.html` through `index.js`.

In `index.js` add one line:

```javascript
import './index.js'
```

# Sourcemap

Sourcemap allows us to see the original code in the browser console. We have already configured it in our webpack config

`devtool: 'inline-source-map'`.

Now if you try to set a `debugger` break point in the `index.js`, you can see the original code in the browser console.

The cool thing is that this sourcemap will only be downloaded when the console is open, neat.

# Linting

We choose [ESLint](https://eslint.org/).

Create `/.eslintrc.json`:

```javascript
{
  "root": true,
  "extends": [
    "eslint:recommended",
    "plugin:import/errors",
    "plugin:import/warnings"
  ],
  "parserOptions": {
    "ecmaVersion": 7,
    "sourceType": "module"
  },
  "env": {
    "browser": true,
    "node": true,
    "mocha": true
  },
  "rules": {
    "no-console": 1 // 0 Off, 1 Warning e 2 Error
  }
}

```

Next, create a _lint_ script in the `package.json`:

```javascript
...
"scripts": {
    "lint": "esw webpack.config.* src buildScripts --color; exit 0"
  },
...
```

## Watch files

To watch for file changes and run eslint, we create another script, and add it to the `start` script.

```javascript
...
"scripts": {
     ...
     "start": "npm-run-all --parallel security-check open:src lint:watch",
     "lint:watch": "npm run lint -- --watch",
     ...
  },
...

```

# Testing and Continuous Integration

Mini introduction on this topic includes this types of tests.
 Style         | Focus
-------------- | -----------------------------
Unit           | Single function or module
Integration    | Interactions between modules
UI             | Automate interactions with UI

For this course, let’s frame the scope here, we are talking about unit testing, not UI testing.

OK, now we have to make 6 decisions:

1.  Testing Framework: [Mocha](https://mochajs.org/)
2.  Assertion Library: [Chai](http://www.chaijs.com/)
3.  Helper Library: [JSDOM](https://github.com/jsdom/jsdom), [Cheerio](https://cheerio.js.org/)
4.  Where to Run Tests: Here we choose JSDOM again.
5.  Where do Test Files Belong? Alongside the file being tested.
6.  When should tests run? Unit Tests Should Run When You Hit Save.

Now we have made the decisions, let’s it all setup.

First let’s add a script to configure the tests `/buildScripts/testSetup.js`:

```javascript
// This file is't transpiled, so must use CommonJS and ES5

// Register babel to transpile before our tests run.
require('babel-register')();

// Disable webpack features that Mocha doesn't understand
require.extensions['.css'] = function() {};

```

Now create a task to run tests:

```javascript
"scripts": {
     ...
     "test": "mocha --reporter progress buildScripts/testSetup.js \"src/**/*.test.js\"",
     ...
  },
...

```

Now let’s write some simple tests to try it out.

Create `/src/index.test.js`:

```javascript
import {expect} from 'chai';
import jsdom from 'jsdom';
import fs from 'fs';

describe('Our first test', () => {
  it('should pass', () => {
    expect(true).to.equal(true);
  });
});

describe('index.html', () => {
  it('should say hello', (done) => {
    const index = fs.readFileSync("./src/index.html", "utf-8");
    jsdom.env(index, function(err, window) {
      const h1 = window.document.getElementsByTagName('h1')[0];
      expect(h1.innerHTML).to.equal("Hello World!");
      done();
      window.close();
    });
  })
})

```

Now if you run `npm test`, you will see 2 tests pass.

OK, let’s next configure the tests to be ran on every save.

Add the following task in `package.json` and add it in `start`:

```javascript
"scripts": {
     ...
     "start": "npm-run-all --parallel security-check open:src lint:watch test:watch",
     "test:watch": "npm run test -- --watch"
     ...
  },

```

## Setup Travis CI

Go to [Travis](https://travis-ci.org) and login with github account, go to profile and enable the js-dev-env project.

Now create `/.travis.yml`:

```javascript
language: node_js
node_js:
  - "6"
```

That’s all you need to setup Travis, now just wait and see the result.

# HTTP Calls

Let’s first create a dummy api endpoint `localhost:3000/users` which just returns a list of user objects.

In `/buildScripts/srcServer.js`, add the following code:

```javascript
app.get('/users', function(req, res) {
  res.json([
    {"id": 1, "firstName": "Bob", "lastName": "Smith", "email": "bob@gmail.com"},
    {"id": 2, "firstName": "Tammy", "lastName": "Norton", "email": "tammy@gmail.com"},
    {"id": 3, "firstName": "Tinna", "lastName": "Lee", "email": "tina@yahoo.com"}
  ]);
});
```

Next, let’s create `/src/api/userApi.js`:

```javascript
import 'whatwg-fetch';

export function getUsers() {
  return get('users');
}

function get(url) {
  return fetch(url).then(onSuccess, onError);
}

function onSuccess(response) {
  return response.json();
}

function onError(error) {
  console.log(error); // eslint-disable-line no-console
}
```

Then, put a table in the `/src/index.html`:

```html
<h1>Users</h1>
<table>
  <thead>
    <th> </th>
    <th>Id</th>
    <th>First Name</th>
    <th>Last Name</th>
    <th>Email</th>
  </thead>
  <tbody id="users"></tbody>
</table>
```

And in `/src/index.js` add:

```javascript
import {getUsers} from './api/userApi';

// Populate table of users via API call
getUsers().then(result => {
  let usersBody = "";

  result.forEach(user => {
    usersBody += `<tr>
    <td><a href="#" data-id="${user.id}" class="deleteUser">Delete</a></td>
    <td>${user.id}</td>
    <td>${user.firstName}</td>
    <td>${user.lastName}</td>
    <td>${user.email}</td>
    </tr>`
  });
  window.document.getElementById('users').innerHTML = usersBody;
});
```
After this change, our 2nd test will fail, so we will need to update the  `/src/index.test.js` file with the following:

```javascript
...
describe('index.html', () => {
  it('should have h1 that contains Users', (done) => {			//TEST DESCRIPTION UPDATED...
    const index = fs.readFileSync("./src/index.html", "utf-8");
    jsdom.env(index, function(err, window) {
      const h1 = window.document.getElementsByTagName('h1')[0];
      expect(h1.innerHTML).to.equal("Users"); 				//Change string to Users instead of Hello world.
      done();
      window.close();
    });
  })
});
```

Now we can see a list of users displayed in the table, great.

## Our Plan for Mocking HTTP

1.  Declare our schema:
    
    -   [JSON Schema Faker](http://json-schema-faker.js.org/)  
        
2.  Generate Random Data:
    
    -   [faker.js](https://github.com/marak/Faker.js/)
    -   [chance.js](https://chancejs.com/)
    -   [randexp.js](http://fent.github.io/randexp.js/)
3.  Serve Data via API
    
    -   [JSON Server](https://github.com/typicode/json-server)

Now let’s glue them together to generate some fake data!

First create `/buildScripts/mockDataSchema.js`:

```javascript
export const schema = {
  "type": "object",
  "properties": {
    "users": {
      "type": "array",
      "minItems": 3,
      "maxItems": 5,
      "items": {
        "type": "object",
        "properties": {
          "id": {
            "type": "number",
            "unique": true,
            "minimum": 1
          },
          "firstName": {
            "type": "string",
            "faker": "name.firstName"
          },
          "lastName": {
            "type": "string",
            "faker": "name.lastName"
          },
          "email": {
            "type": "string",
            "faker": "internet.email"
          }
        },
        "required": ["id", "firstName", "lastName", "email"]
      }
    }
  },
  "required": ["users"]
};
```

Next, write a script `/buildScripts/generateMockData.js` to generate fake data:

```javascript
/* This script generate mock data for local development. */

/* eslint-disable no-console */

import jsf from 'json-schema-faker';
import {schema} from './mockDataSchema';
import fs from 'fs';
import chalk from 'chalk';
import faker from "faker"

// This is a fix from github which is not in the video
jsf.extend("faker", function() {
  return faker
})

const json = JSON.stringify(jsf(schema));

fs.writeFile("./src/api/db.json", json, function (err) {
  if (err) {
    return console.log(chalk.red(err));
  } else {
    console.log(chalk.green("Mock data generated."));
  }
});
```

Finally, configure `package.json` to include it and also use _json server_ serve the data.

```javascript
"scripts": {
     ...
     "start": "npm-run-all --parallel security-check open:src lint:watch test:watch start-mockapi",
     "generate-mock-data": "babel-node buildScripts/generateMockData",
     "prestart-mockapi": "npm run generate-mock-data",
     "start-mockapi": "json-server --watch src/api/db.json --port 3001"
     ...
  },
```

We would like to use the fake api for development. How to do that?

Well first let’s create `/src/api/baseUrl.js`:

```javascript
export default function getBaseUrl() {
  const inDevelopment = window.location.hostname === 'localhost';
  return inDevelopment ? 'http://localhost:3001' : '/';
}
```

Now in `/api/userApi.js` make the following changes:

```javascript
import getBaseUrl from './baseUrl';

function get(url) {
  return fetch(baseUrl + url).then(onSuccess, onError);
}

```
Now we have the get users api, let’s also add delete users api. First add the delete user api in `/src/api/userApi.js`:

```javascript
export function deleteUser(id){
  return del(`users/${id}`)
}

function del(url) {
  const request = new Request(baseUrl + url, {
    method: 'DELETE'
  });
  return fetch(request).then(onSuccess, onError);
}
```

Then in `/src/index.js` add the following code:

```javascript
// Populate table of users via API call
getUsers().then(result => {
  ...
  
  const deleteLinks = window.document.getElementsByClassName('deleteUser');

  // Must use array.from to create a real array from a DOM collection
  // getElementsByClassName only returns an "array like" object
  Array.from(deleteLinks, link => {
    link.onclick = function(event) {
      const element = event.target;
      event.preventDefault();
      deleteUser(element.attributes["data-id"].value);
      const row = element.parentNode.parentNode;
      row.parentNode.removeChild(row);
    };
  });
});
```

Now if you click the delete button, that user will be removed from the screen, and also from the fake json data db!

# Project Structure

Some useful tips on structuring JS projects:

1.  JS Belongs in a `.js` File
2.  Organize by Feature instead of File Type
3.  Extract logic into *POJO*s.

# Production Build

## Minification and Sourcemaps

First create a `webpack.config.prod.js` for production:

```javascript
import path from 'path';
import webpack from 'webpack';

export default {
  debug: true,
  devtool: 'source-map',
  noInfo: false,
  entry: [
    path.resolve(__dirname, 'src/index')
  ],
  target: 'web',
  output: {
    path: path.resolve(__dirname, 'dist'),
    publicPath: '/',
    filename: 'bundle.js'
  },
  plugins: [
    // Eliminate duplicate packages when generating bundle
    new webpack.optimize.DedupePlugin(),
    // Minify JS
    new webpack.optimize.UglifyJsPlugin()
  ],
  module: {
    loaders: [
      {test: /\.js$/, exclude: /node_modules/, loaders: ['babel']},
      {test: /\.css$/, loaders: ['style','css']}
    ]
  }
}

```

Note that we changed the output to `dist` and also added some plugins for minifications.

Next, let’s create a `/buildScripts/distServer.js` to serve the files in `dist`:

```javascript
import express from 'express';
import path from 'path';
import open from 'open';
import compression from 'compression';

/* eslint-disable no-console */

const port = 3000;
const app = express();

app.use(compression());

app.use(express.static('dist'));

app.get('/', function(req, res) {
  res.sendFile(path.join(__dirname, '../dist/index.html'));
});

app.get('/users', function(req, res) {
  res.json([
    {"id": 1, "firstName": "Bob", "lastName": "Smith", "email": "bob@gmail.com"},
    {"id": 2, "firstName": "Tammy", "lastName": "Norton", "email": "tammy@gmail.com"},
    {"id": 3, "firstName": "Tinna", "lastName": "Lee", "email": "tina@yahoo.com"}
  ]);
});

app.listen(port, function(err) {
  if (err) {
    console.log(err);
  } else{
    open('http://localhost:' + port);
  }
});

```

## Toggle Mock Api

Then, let’s put in a better way to toggle between real data and mock data, modify `/src/api/baseUrl.js`:

```javascript
export default function getBaseUrl() {
  return getQueryStringParameterByName('useMockApi') ? 'http://localhost:3001/' : '/';
}

function getQueryStringParameterByName(name, url) {
  if (!url) url = window.location.href;
  name = name.replace(/[\[\]]/g, "\\$&");
  var regex = new RegExp("[?&]" + name + "(=([^&#]*)|&|#|$)"),
      results = regex.exec(url);
  if (!results) return null;
  if (!results[2]) return '';
  return decodeURIComponent(results[2].replace(/\+/g, " "));
}

```

Now to use mock data, we just append `?useMockApi=true` to the url in the browser.

Now it’s time to write some npm script to build all of these.

Add these new tasks:

```javascript
"clean-dist": "rimraf ./dist && mkdir dist",
"prebuild": "npm-run-all clean-dist test lint",
"build": "babel-node buildScripts/build.js",
"postbuild": "babel-node buildScripts/distServer.js"
    

```

Now if you run `npm run build -s`, it will show that no index.html found, which is what we are going to tackle next.

## Dynamic HTML Generation

Our solution to this is to use webpack to dynamically generate html files.

Add the following code to `/webpack.config.prod.js`:

```javascript
import HtmlWebpackPlugin from 'html-webpack-plugin';

plugins: [
    // Create HTML file that includes reference to bundled JS.
    new HtmlWebpackPlugin({
      template: 'src/index.html',
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeRedundantAttributes: true,
        useShortDoctype: true,
        removeEmptyAttributes: true,
        removeStyleLinkTypeAttributes: true,
        keepClosingSlash: true,
        minifyJS: true,
        minifyCSS: true,
        minifyURLs: true
      },
      inject: true
    }),
  ]
```

Now we can remove the javascript tag from `index.html` since it will be injected by webpack. Also copy the same to `/webpack.config.dev.js`.

## Bundle Splitting

We are going to put all the vendor JS libraries in a separate bundle so that the client won’t have to donwload those libraries everytime when our application code changes.

First create a `/src/vendor.js`:

```javascript
/* This file contains references to the vendor libraries
 we're using in this project. This is used by webpack
 in the production build only*. A separate bundle for vendor
 code is useful since it's unlikely to change as often
 as the application's code. So all the libraries we reference
 here will be written to vendor.js so they can be
 cached until one of them change. So basically, this avoids
 customers having to download a huge JS file anytime a line
 of code changes. They only have to download vendor.js when
 a vendor library changes which should be less frequent.
 Any files that aren't referenced here will be bundled into
 main.js for the production build.
 */

/* eslint-disable no-unused-vars */

import fetch from 'whatwg-fetch';
```

Then modify the `webpack.config.prod.js` as follows:

```javascript
entry: {
    vendor: path.resolve(__dirname, 'src/vendor'),
    main: path.resolve(__dirname, 'src/index')
  },
  output: {
    ...
    filename: '[name].js'
  },
  plugins: [
    // Use CommonsChunkPlugin to create a separate bundle
    // of vendor libraries so that they're cached separatetly.
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor'
    }),
    ...
  ],

```

## Cache Busting

If we attach a hash number at the end of every files, then the client can cache the files forever until we make a new release.

Let’s do it.

In `webpack.config.prod.js`:

```javascript
import WebpackMd5Hash from 'webpack-md5-hash';

output: {
    ...
    filename: '[name].[chunkhash].js'
  },

plugins: [
    // Hash the files using MD5 so that their names change when the content changes.
    new WebpackMd5Hash(),
    ...
]
```

## Extract and Minify CSS

The goal is to extract out the css file and put a hash number in its name.

So in `webpack.config.prod.js`:

```javascript
import ExtractTextPlugin from 'extract-text-webpack-plugin';

plugins: [
    // Generate an external css file with a hash in the filename
    new ExtractTextPlugin('[name].[contenthash].css'),
    ...
]

module: {
    loaders: [
      ...
      {test: /\.css$/, loader: ExtractTextPlugin.extract('css?sourceMap')}
    ]

```

## Error Logging

The author uses TrackJS, since it is not free, I will omit this part all together.

Interesting! The index.html contains a code which include a logic using htmlWebpackPlugin.
It parses logic to only add html code when using webpack prod config.

```html
...
<!--
    The htmlWebpackPlugin will parse the logic below so that trackJS is only added to the production version of index.html (since only webpack.config.prod.js provides the trackJS token)
    This is an example of how to add features to index.html for only one environment.
-->
<% if (htmlWebpackPlugin.options.trackJSToken) { %>
    <!-- BEGIN TRACKJS -->
    ...OMITTED
    <!-- END TRACKJS -->
<% } %>
...
```

# Production Deploy

We will deploy the api backend to [Heroku](https://www.heroku.com/) and the frontend to [Surge](https://surge.sh/).

## Deploy API to Heroku

First, take a look at the [guide](https://devcenter.heroku.com/articles/getting-started-with-nodejs#introduction) on how to deploy nodejs app on Heroku.

OK, now let’s try to split up our project.

First separate out the api part. The author created a [repo](https://github.com/lvguowei/js-dev-env-demo-api) here already so we can just use that. Remember to run `npm install` after clone it.

Then follow the steps:

1.  Login to Heroku.

`heroku login`

1.  Deploy the app.

`heroku create`

`git push heroku master`

`heroku ps:scale web=1`

`heroku open`

OK, now the api site is deployed!

Now we go back to our project, open `/src/api/baseUrl.js` and update the following code:

```javascript
export default function getBaseUrl() {
  return getQueryStringParameterByName('useMockApi') ? 'http://localhost:3001/' : 'https://[YOUR-HEROKU-ID].herokuapp.com/'; //CHANGE THIS ADDRESS TO MATCH YOUR HEROKU URL
}
```

## Deploy UI to Surge

Add one line to `scripts` in `package.json`:

`"deploy": "surge ./dist"`

Then run `npm run build` to build once.

Now we can deploy to surge `npm run deploy`.

And this finished our course on JS dev environment... :)

***THE END!***

## Extending / using this toolkit

Edit the `/src/*.*` files:
* index.html for templating html
* index.js for main app code...
* *.test.js for tests bound to the npm script test
* index.css for css data obviously :)
* `/src/api/*.*` contains a rest mock endpoints to simulate custom / random data for UI developemnt

Ideas for extending this toolkit:
* Add a new npm script to copy static files for a self host site
* Add scss support
