---
layout: post
title: Lesson 1 Nodejs course
date: 2017-10-30 12:00:00 -0700
icon: code
tags: nodejs
---

# Project structure

The first thing that we have to think about in any new project, it's the objective that we want to achieve. So discussing with friends, I'm starting a course (I hope that I can release one lesson per week), to practice and enjoy Nodejs.

We decided to create a little project trying to build a Chat application, following the best practices in javascript and Nodejs.

## Prerequisites

- Node js >= 8.0.0
- Text editor or IDE

For the first lesson we only need [Nodejs](https://nodejs.org) we want the last version, so all the [ES6](http://es6-features.org) that we need is supported, and the text editor (I'm not sure if its an IDE) is [VS Code](https://code.visualstudio.com) because it has a great support for git and Nodejs without installing any dependency.

### Nodejs dependencies

The dependencies in Nodejs are all the projects that other people have created and we can use through NPM.

The main difference between dependencies and dev-dependencies is: If we publish our project to npm registry when other people install it, npm will download only the dependencies, not dev-depencies.

There are more types of dependencies, but they don't make sense for now.

#### Dependencies

- [Fastify](https://www.fastify.io/)
- [Pino](http://getpino.io/)
- [Ramda](http://ramdajs.com)
- [Hyperscript](https://github.com/hyperhype/hyperscript)

#### Dev dependencies

- [Nodemon](https://nodemon.io)

## Links to more content

Instead of writing everything (like in a book), I will add this section with a link.

- [package.json](https://docs.npmjs.com/files/package.json)
- [npm init command](https://docs.npmjs.com/cli/init)
- [project structure Node Hero tutorial](https://blog.risingstack.com/node-hero-node-js-project-structure-tutorial/)

## Defining the project structure

**Init project**

Our first step is to init the Nodejs project, we can do this easily using:

```bash
$ npm init
```

This command will ask some questions related to the project metadata, follow the link to package json to know what means every question.

**Install dependencies**

```bash
$ npm install --save pino ramda fastify hyperscript
```
**Install dev-dependencies**

```bash
$ npm install --save-dev nodemon
```
**Creating index.html**

In this course we gonna write almost everything in javascript including html, that's why we included hyperscript: So we will create a file with name **index.html.js** and paste the next content:

```javascript
const h = require('hyperscript');

const IndexPage = (props) => h('html', {}, [
  h('head', {}, [
    h('title', {}, props.title),
    h('meta', { charset: 'UTF-8'}),
    h('meta', { 
      name: 'viewport',
      content: 'width=device-width, initial-scale=1'
    }),
    h('link', {
      href: '/css/style.css',
      rel: 'stylesheet'
    }),
  ]),
  h('body', {}, [
    h('div', {}, 'Hello World from nodejs'),
    h('script', { src: '/js/app.js' })
  ])
]);

module.exports = IndexPage;
```

*require* function and *module.exports* are explained [here](https://nodejs.org/dist/latest-v8.x/docs/api/modules.html). Basically is how commonjs works. We are importing hyperscript and exposing IndexPage function to other modules, this creates a closure so every variable that we create here will be only accessible for the functions in this module (we well use module/file indistinctly).

hyperscript function is really simple *(tag, attributes, children)*, it works perfectly fine as html template generator, maybe looks weird at the beginning but now we have all the power or javascript to modify the content easily (we are rendering from the server side, there is no document object available).

**Creating utils.js**

Lets create a *utils.js* file, which will have somo utility functions, at these moment just a function to gzip and add some headers to out html file.

```javascript
const path = require('path');
const fs = require('fs');
const zlib = require('zlib');
const Duplex = require('stream').Duplex;

const fullHtml = (reply, element, props) => {
  let stream = new Duplex();

  reply.header('content-type', 'text/html');
  reply.header('Content-Encoding', 'gzip');
  reply.header('x-frame-Options', 'SAMEORIGIN');
  reply.header('Content-Security-Policy', 'script-src \'self\'');
  reply.header('x-xss-protection', '1; mode=block');
  reply.header('x-content-type-options', 'nosniff');
  
  const gzip = zlib.createGzip();
  const buffer = Buffer.from(`<!DOCTYPE html>${element(props).outerHTML}`);
  stream.push(buffer);
  stream.push(null);
  reply.send(stream.pipe(gzip));
};

module.exports = {
  fullHtml
};
```

**Create configuration file**

Next task is to add a configuration file, it will have all configurations required for the project like ports, urls, constant names, etc.

It will be under *conf* folder with the name *index.js* this name have a meaning in nodejs modules, if you require a folder instead of a file, it will get the index file of that folder otherwise nodejs will tell you that it can't find the module.

The content is really simple by the moment:

```javascript
module.exports = {
  port: 3000,
  indexHtml: {
    title: 'nodejscourse'
  }
};
```

**Create entry point**

Finally we can create the main file of the application *index.js* this is the entre point of the application, and every module should be called from here.

```javascript
const pino = require('pino')({ name: 'myapp', level: 'debug', prettyPrint: true });
const config = require('./conf');
const fastify = require('fastify')({ logger: pino });
const {fullHtml } = require('./utils');
const indexHtml = require('./index.html');

fastify.get('/index.html', function (request, reply) {
  fullHtml(reply, indexHtml, config.indexHtml);
});

fastify.listen(config.port, (err) => {
  if (err) {
    pino.fatal(err);
    throw err
  };
  pino.info(`server listening on ${fastify.server.address().port}`);
});
```

## Final project structure

All our changes we should save them in a version control system like git, so we need to intiallize the repo and include *.gitignore* file, also we gonna include a folder named *messages* with an empty *index.js* file inside, just to prepare our code for next lesson. With this changes this should be the final structure of the project in lesson 1:

```bash
|  conf
|  |--  index.js
|  messages
|  |--  index.js
|  node_modules
|  .gitignore
|  LICENSE
|  index.html.js
|  index.js
|  utils.js

```

## Running the application

To run the application you just have to execute the node command:

```bash
$ node index.js
```
and you should see something in the output. Then you can verify that everything is working by visiting:

`http://localhost:3000/index.html`

## Github

The project will be in [Github](https://github.com/betotto/nodejscourse).

# Help me

If you have any recomendation or english grammar correction please write it here:  **[blog issues](https://github.com/betotto/blog/issues)**.