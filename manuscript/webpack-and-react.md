# Webpack and React

Facebook's [React](https://facebook.github.io/react/) is one of those projects that has changed the way we think about frontend development. Thanks to [React Native](https://facebook.github.io/react-native/) the approach isn't limited just to web. Although simple to learn, React provides plenty of power.

Webpack is an ideal tool to complement it. By now we understand how to set up a simple project on top of Webpack. Let's turn it into a React project next and implement a little todo app. It won't be very complex but will help you to understand some of the basics.

## Installing React

To get started install React to your project. Just hit `npm i react --save` and you should be set. As a next step we could port our **app/component.js** to React. Provided we use ES6 module and class syntax and JSX, we can go with a solution like this:

**app/TodoItem.js**

```javascript
import React from 'react';

export default class TodoItem extends React.Component {
    render() {
        return <div>Learn Webpack</div>;
    }
}
```

In addition we'll need to adjust our `main.js` to render the component correctly. Here's one solution:

**app/main.js**

```javascript
import React from 'react';
import Hello from './TodoItem';

React.render(<TodoItem />, document.getElementById('app'));
```

> Avoid rendering directly to `document.body`. This can cause strange problems with plugins that rely on body due to the way React works. It is much better idea to give it a container of its own.

## Setting Up Webpack

In order to make everything work again, we'll need to tweak our configuration a little. In order to deal with ES6 and JSX, we'll use [babel-loader](https://www.npmjs.com/package/babel-loader). Install it using `npm i babel-loader --save-dev`. In addition add the following loader declaration to the *loaders* section of your configuration:

```javascript
{
    test: /\.js$/,
    loader: 'babel',
    include: path.join(ROOT_PATH, 'app'),
}
```

We will specifically include our `app` source to our loader. This way Webpack doesn't have to traverse whole source. Particularly going through `node_modules` can take a while. You can try taking `include` statement out to see how that affects the performance.

> An alternative would be to `exclude` but `include` feels like a better solution here.

If you hit `npm run build` now, you should get some output after a while. Here's a sample:

```bash
> webpack_demo@1.0.0 build /Users/something/projects/webpack_demo
> webpack --config config/build

Hash: 00f0b93cf085fc55d4ec
Version: webpack 1.7.3
Time: 1149ms
    Asset    Size  Chunks             Chunk Names
bundle.js  633 kB       0  [emitted]  main
   [0] multi main 28 bytes {0} [built]
    + 162 hidden modules
```

As you can see, the output is quite chunky in this case! Don't worry. This is just an unoptimized build. We can do a lot about the size at a later stage when we apply optimizations, minification and split things up.

## Resolving JSX Files

The configuration above works but what if we want to resolve files with `.jsx` extension? After all it can be a good idea to separate vanilla JavaScript files from those containing JSX.

To achieve this we need to extend out Regex pattern like this:

```javascript
{
    test: /\.jsx?$/,
    loader: 'babel',
    include: path.join(ROOT_PATH, 'app'),
}
```

If you try renaming **app/component.js** as **app/component.jsx** and hit `npm run build`, you should get an error like this:

```bash
ERROR in ./app/main.js
Module not found: Error: Cannot resolve 'file' or 'directory' ./component in /Users/something/projects/webpack_demo/app
 @ ./app/main.js 11:13-35
```

This means Webpack cannot resolve our `import Hello from './component';` to a file.

The problem has to do with Webpack's default resolution settings. Those settings describe where Webpack looks for modules and how. We'll need to tweak these settings a little.

Add the following bit to your configuration:

```javascript
var common = {
    ...
    resolve: {
        extensions: ['', '.js', '.jsx', '.css'],
    }
};
```

Now Webpack will be able to resolve files ending with `.jsx` and everything should be fine. If you try running `npm run build` again, the build should succeed.

## Activating Hot Loading for Development

If you hit `npm run dev` and try to modify our component (make it output `hello world again` or something), you'll see it actually works. After a flash. We can get something fancier with Webpack, namely hot loading. This is enabled by [react-hot-loader](https://gaearon.github.io/react-hot-loader/).

To make this work, you should `npm i react-hot-loader --save-dev` and tweak the configuration as follows:

**config/index.js**

```javascript
var path = require('path');
var webpack = require('webpack');

...

exports.build = _.merge({
    module: {
        loaders: [
            {
                test: /\.jsx?$/,
                loader: 'babel',
                include: path.join(ROOT_PATH, 'app'),
            },
        ]
    },
}, common, joinArrays);

exports.develop = _.merge({
    entry: ['webpack/hot/dev-server'],
    module: {
        loaders: [
            {
                test: /\.jsx?$/,
                loaders: ['react-hot', 'babel'],
                include: path.join(ROOT_PATH, 'app'),
            },
        ]
    },
    plugins: [
        // hot module replacement plugin itself. if you pass `--hot` to
        // webpack-dev-server, do not activate this!
        new webpack.HotModuleReplacementPlugin(),
        // do not reload if there is a syntax error in your code
        new webpack.NoErrorsPlugin()
    ],
}, common, joinArrays);
```

Note what happens if you `npm run dev` now and try to modify the component. There should be no flash (no refresh) while the component should get updated provided there was no syntax error.

The advantage of this approach is that the user interface retains its state. This can be quite convenient! There will be times when you may need to force a refresh but this tooling decreases the need for that significantly.

## Setting Up ESLint

A little discipline goes a long way in programming. Linting is one of those techniques that will simplify your life a lot at a minimal cost. It is possible to integrate this process into your editor/IDE.

That way you can fix potential problems before they become actual issues. It won't replace testing but it will simplify your life and make it more boring. Boring is good.

[ESLint](http://eslint.org/) is a recent linting solution for JavaScript. It builds on top of ideas presented by JSLint and JSHint. Most importantly it allows you to develop custom rules. As a result a nice set of rules have been developed for React in form of [eslint-plugin-react](https://www.npmjs.com/package/eslint-plugin-react).

Setting it up is quite simple. We'll just need to a couple of little tweaks to our project. To get eslint installed, invoke `npm i babel-eslint eslint eslint-plugin-react --save-dev`. That will add ESLint and the plugin we want to use as our project development dependency.

Next we'll need to do some configuration. Add `"lint": "eslint . --ext .js --ext .jsx"` to the *scripts* section of **package.json**. Besides this we'll need to set up some ESlint specific configuration.

First we'll ignore `node_modules/` since we don't want to lint that. You should ignore possible distribution build and so on in this file.

**.eslintignore**

```
node_modules/
build/
```

Next we'll activate [babel-eslint](https://www.npmjs.com/package/babel-eslint) so that ESLint works with our Babel code. In addition we activate React specific rules and set up a couple of our own. You can adjust these to your liking. You'll find more information about the rules at [the official rule documentation](http://eslint.org/docs/rules/).

**.eslintrc**

```
{
  "parser": "babel-eslint",
  "env": {
    "browser": true,
    "node": true
  },
  "plugins": [
    "react"
  ],
  "ecmaFeatures": {
    "jsx": true,
    "globalReturn": false
  },
  "rules": {
    "no-underscore-dangle": false,
    "no-use-before-define": false,
    "quotes": [4, "single"],
    "comma-dangle": "always",
    "react/display-name": true,
    "react/jsx-quotes": true,
    "react/jsx-no-undef": true,
    "react/jsx-uses-react": true,
    "react/jsx-uses-vars": true,
    "react/no-did-mount-set-state": true,
    "react/no-did-update-set-state": true,
    "react/no-multi-comp": true,
    "react/prop-types": true,
    "react/react-in-jsx-scope": true,
    "react/self-closing-comp": true,
    "react/wrap-multilines": true
  }
}
```

If you hit `npm run lint` now, you should get some errors and warnings to fix depending on the rules you have set up.

There is a good ESLint integration available for many IDEs and editors already. You should consider setting it up for yours as that will allow you to spot possible issues as you develop.

In case you want Webpack to emit ESLint messages, you can set up [eslint-loader](https://www.npmjs.com/package/eslint-loader) for this purpose. You can even make your build fail on purpose if it doesn't pass the lint.

## Implementing Basic Todo List

Given we have a nice development setup now, we can actually get some work done. Hit `npm run dev`. It's time to start developing.

To make it easier, let's set up `TodoApp.jsx` that coordinates application itself. This will get rendered by `main.js` and will deal with the application logic. You should end up with files like this:

**app/main.js**

```javascript
import './main.css'

import React from 'react';
import TodoApp from './TodoApp';

React.render(<TodoApp />, document.getElementById('app'));
```

**app/TodoApp.jsx**

```javascript
'use strict';
import React from 'react';
import TodoItem from './TodoItem';

export default class TodoApp extends React.Component {
    render() {
        return <TodoItem />;
    }
}
```

**app/TodoItem.jsx**

```javascript
'use strict';
import React from 'react';

export default class TodoItem extends React.Component {
    render() {
        return <div>Learn Webpack</div>;
    }
}
```

Note that as you perform the needed modifications, your browser should get updated. You might see some error every now and then but that is to be expected given we are doing breaking changes here.

A good next step would be to extend `TodoItem` interface. We would probably want to render a list of these. Ideally we should be able to perform basic editing operations over the list and create new items as needed. We'll probably also want to mark items as done.

This means `TodoApp` will have to coordinate the state. Let's start by rendering a list and then expand from there. Here's sample code for an enhanced `render` method:

```javascript
render() {
    var todoItems = [{
        task: 'Learn Webpack'
    }, {
        task: 'Learn React'
    }, {
        task: 'Do laundry'
    }];

    return (
        <div>
            <ul>{todoItems.map((todoItem, i) =>
                <li key={'todoitem' + i}>
                    <TodoItem task={todoItem.task} />
                </li>
            )}</ul>
        </div>
    );
}
```

To make it easy to grow the code, we treat possible todo items as a list of objects. We then map through them and generate `TodoItem` for each. During the process we set `key` for each list item. This is property React requires in order to render each item to correct place during each render pass. React will warn you if you forget to set it. In addition we pass the task in question to our `TodoItem` as a property.

If everything went correctly, you should see a list with three `Learn Webpack` items on it. That's almost nice. To make `TodoItem` render its property correctly, we'll need to tweak its implementation a little bit like this:

```javascript
render() {
    return <div>{this.props.task}</div>;
}
```

As you can see the property we passed to our component gets mapped to `this.props`. After that it is just a matter of showing it.

We haven't achieved much yet but we're getting there. Next we should add some logic to the list so this application can do something useful.

## Adding New Items to Todo List

It would be useful if we could add new items to our Todo list. Let's just do a button with plus that when triggered adds a new dummy item to our list.

To get a button show up, add `<button onClick={this.addItem.bind(this)}>+</button>` somewhere within `TodoApp` JSX. Besides this we'll need to define that `onClick` handler. Define a method like this:

```javascript
addItem() {
    console.log('add item');
}
```

Now when you click the button, you should see something at your browser console.

Next we will need to connect this with our data model somehow. It is problematic that now it is stored within our `render` method. React provides a concept known as state for this purpose. We can move our data there like this:

```javascript
constructor(props) {
    super(props);

    this.state = {
        todoItems: [{
            task: 'Learn Webpack'
        }, {
            task: 'Learn React'
        }, {
            task: 'Do laundry'
        }]
    };
}
render() {
    var todoItems = this.state.todoItems;

...
```

Now our `render` method points at `state`. As a result we can implement `addItem` that actually does something useful:

```javascript
addItem() {
    this.setState({
        todoItems: this.state.todoItems.concat([{
            task: 'New task'
        }])
    });
}
```

If you hit the button now, you should see new items. It might not be pretty yet but it works.

## Editing Todo List Items

TODO

## Optimizing Rebundling

XXX: this is difficult to set up with react-hot-loader. see https://github.com/gaearon/react-hot-loader/blob/master/docs/README.md#usage-with-external-react . maybe skip this part?

As waiting for the reload to happen can be annoying we can take our approach a little further. Instead of making Webpack go through React and all its dependencies, you can override the behavior in development. We'll set up an alias for it and point to its minified version.

**webpack.config.js**

```javascript
var path = require('path');
var node_modules = path.resolve(__dirname, 'node_modules');
var pathToReact = path.resolve(node_modules, 'react/dist/react.min.js');

var config = {
  entry: path.resolve(__dirname, 'app/main.js'),
  resolve: {
    alias: {
      'react': pathToReact
    }
  },
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'bundle.js'
  },
  module: {
    noParse: [pathToReact]
  }
};

module.exports = config;
```
We do two things in this configuration:

1. Whenever "react" is required in the code it will fetch the minified React.js file instead of going to *node_modules*

2. Whenever Webpack tries to parse the minified file, we stop it, as it is not necessary

Take a look at `Optimizing development` for more information on this.

> TBD: point at the correct chapter here

## Type Checking with Flow

If you come to JavaScript from other programming languages you are familiar with types. You have types in JavaScript too, but you do not have to specify these types when declaring variables, receiving arguments etc. This is one of the things that makes JavaScript great, but at the same time not so great.

Specifically when working on very large projects with many developers type checking gives stability to your project, much like a good test does. So using **Flow** is definitely not a requirement. It is for developers who depends on type checking as more of a routine and for the before mentioned large projects with many developers. Webpack makes it easy to include **Flow** in your workflow.

### Installing flow

- Have to try this out :-)
- What about "flowcheck-loader", tried it? https://www.npmjs.com/package/flowcheck-loader (probably works, haven't tried this one yet)
- https://tryflow.org/

> TBD: expand this section

## PureRenderMixin

This gives you a very short and nice syntax for defining components. A drawback with using classes though is the lack of mixins. That said, you are not totally lost. Lets us see how we could still use the important **PureRenderMixin**.

```javascript
import React from 'react/addons';

class Component extends React.Component {
  shouldComponentUpdate() {
    return React.addons.PureRenderMixin.shouldComponentUpdate.apply(this, arguments);
  }
}

class MyComponent extends Component {
  constructor() {
    this.state = {message: 'Hello world'};
  }
  render() {
    return (
      <h1>{this.state.message}</h1>
    );
  }
}
```