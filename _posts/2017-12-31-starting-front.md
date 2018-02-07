---
layout: post
title: Starting with the UI 
date: 2017-12-31 12:00:00 -0700
published: true
icon: chat
tags: nodejs
---

Now we move to the UI, one of my favorite things is [Preact](https://preactjs.com) and we will use it for all our components in front, basically is React Api smaller and faster, like many people say "The same but Different".

# Prerequisites
 
- All the previous lessons.

# Adding new dependencies

The next command will install all the dependencies that we need:
```bash
$ npm install --save-dev babel-core babel-loader babel-preset-es2015 memory-fs preact ramda redux webpack
```
We will not use Redux and Ramda yet but is ok to install them now. These are dev dependencies, only needed to build and develop the dependencies.

## Moving files

I moved a lot of files because now we need to focus on the front and not only on the back, also as our application is growing it needs a better structure.

I moved utils file to modules folder, it includes now the webpack on the fly compilation and I commented some lines for security that are not needed by now.

this is the file now modules/utils.js
```javascript
const path = require('path');
const fs = require('fs');
const zlib = require('zlib');
const { Duplex } = require('stream');
const less = require('less');
const webpack = require('webpack');
const MemoryFS = require('memory-fs');

const fullHtml = (reply, element, props, children) => {
  const stream = new Duplex();

  reply.header('content-type', 'text/html');
  reply.header('Content-Encoding', 'gzip');
  // reply.header('x-frame-Options', 'SAMEORIGIN');
  // reply.header('Content-Security-Policy', 'script-src \'self\'');
  // reply.header('x-xss-protection', '1; mode=block');
  // reply.header('x-content-type-options', 'nosniff');

  const gzip = zlib.createGzip();
  const html = Buffer.from(`<!DOCTYPE html>${element(props, children).outerHTML}`);
  stream.push(html);
  stream.push(null);
  reply.send(stream.pipe(gzip));
  stream.end();
};

const lessOnFly = (reply) => {
  reply.header('Content-Encoding', 'gzip');
  reply.header('x-xss-protection', '1; mode=block');
  reply.header('x-content-type-options', 'nosniff');
  reply.header('content-type', 'text/css');

  less.render(fs.readFileSync(path.join(__dirname, '../less/styles.less'), 'utf8'), {
    paths: [path.join(__dirname, '../less')],
    filename: 'styles.less'
  }, (e, output) => {
    if(e) {
      reply.code(500).send(e);
    }
    const stream = new Duplex();
    const gzip = zlib.createGzip();

    stream.push(output.css);
    stream.push(null);
    reply.code(200).send(stream.pipe(gzip));
    stream.end();
  });
};

const webpackCompiler = (reply, entry) => {
  const fs = new MemoryFS();

  const compiler =  webpack({
    context: __dirname.replace('/modules', ''),
    entry: `./widgets/${entry}`,
    output: {
      path: '/out',
      filename: entry
    },
    module: {
      rules: [
        {
          test: /\.js$/,
          exclude: [/node_modules/],
          use: [{
            loader: 'babel-loader',
            options: {
              presets: ['es2015']
            }
          }],
        }
      ]
    }
  });

  compiler.outputFileSystem = fs;

  compiler.run((err, stats) => {
    if(stats.compilation.errors > 0) {
      pino.error(stats.compilation.errors[1]);
    }
    const stream = new Duplex();
    const gzip = zlib.createGzip();
    reply.header('Content-Encoding', 'gzip');
    reply.header('x-xss-protection', '1; mode=block');
    reply.header('x-content-type-options', 'nosniff');
    reply.header('content-type', 'text/javascript');
    if(err) {
      stream.push(err);
      stream.push(null);
      reply.code(500);
    } else if(stats.compilation.erros && stats.compilation.errors.length > 0) {
      stream.push(stats.compilation.errors);
      stream.push(null);
      reply.code(500);
    } else {
      stream.push(fs.readFileSync(`/out/${entry}`).toString('utf8'));
      stream.push(null);
      reply.code(200);
    }
    reply.send(stream.pipe(gzip));
    stream.end();
  });
};

module.exports = {
  fullHtml,
  lessOnFly,
  webpackCompiler
};
```

I removed the routes configuration from index.js

```javascript
const pino = require('pino')({ name: 'myapp', level: 'info', prettyPrint: true });
const fastify = require('fastify')({ logger: pino });

const config = require('./conf');

const App = require('./modules');

App(fastify);

global.pino = pino;

fastify.listen(config.port, (err) => {
  if (err) {
    pino.fatal(err);
    throw err;
  }
  global.pino.info(`server listening on ${fastify.server.address().port}`);
});
```

I moved less folder from widget to the root, utils is updated with that

I created a file modules/index.js, and now it contains all the routes configurations.

```javascript
const sessionRoutes = require('./session/routes');
const messagesRoutes = require('./messages/routes');
const mysql = require('./plugins/database');
const { lessOnFly, fullHtml, webpackCompiler } = require('./utils');
const config = require('../conf');
const Views = require('../views');
const tokenHeaderCheck = require('./middlewares/tokenHeaderCheck');

const configApp = (fastify) => {

  fastify.use(['/session', '/messages'], tokenHeaderCheck);

  fastify.register(mysql);

  fastify.register(sessionRoutes, { prefix: '/session' });
  fastify.register(messagesRoutes, { prefix: '/messages' });

  fastify.get('/index.html', (request, reply) => {
    fullHtml(reply, Views.Base, config.indexHtml, [Views.HomePage]);
  });

  fastify.get('/chat.html', (request, reply) => {
    fullHtml(reply, Views.Base, config.indexHtml, [Views.ChatPage]);
  });

  fastify.get('/session.html', (request, reply) => {
    fullHtml(reply, Views.Base, config.indexHtml, [Views.SessionPage]);
  });

  fastify.get('/css/styles.css', (req, reply) => {
    lessOnFly(reply);
  });

  fastify.get('/js/app.js', (req, reply) => {
    webpackCompiler(reply, 'app.js', 'app.js');
  });
};

module.exports = configApp;
```

And finally, I moved the modules/database/plugins.js to modules/plugins/database.js

## Creating the view containers

We we're using hyperscript to display the html content of the page, so for easy handling of the content, we need to create a new folder **views** it will contain all the html that is been served from the server, yes I know it sounds obvious.

Here I have a base html page called **base.html.js**

```javascript
const h = require('hyperscript');
const Menu = require('./menu.html');

const BasePage = (props, children) => h('html', {}, [
  h('head', {}, [
    h('title', {}, props.title),
    h('meta', { charset: 'UTF-8'}),
    h('meta', {
      name: 'viewport',
      content: 'width=device-width, initial-scale=1'
    }),
    h('link', {
      href: '/css/styles.css',
      rel: 'stylesheet'
    }),
  ]),
  h('body', {}, [
    h('div', {
      class: 'pure-g'
    }, [
      h('div', { class: 'pure-u-1-4' }, Menu),
      h('div', { class: 'pure-u-3-4' }, children)
    ])
  ])
]);

module.exports = BasePage;
```

It will show the Menu component and the children that It has, with that we don't have to include the Menu and the basic structure on each Page. So we can create **homPage.html.js**

```javascript
const h = require('hyperscript');

const HomePage = () => h('section', {
  class: 'homePage'
}, [
  h('div', {id: 'main'}, 'Home'),
  h('script', {src: '/js/app.js'}, )
]);

module.exports = HomePage;
```

Wi will pass this as a Child of Base.html and we will have a Menu integrated.

We need to add other 2 pages **chatPage.html.js** 

```javascript
const h = require('hyperscript');

const ChatPage = () => h('div', {
}, 'hello');

module.exports = ChatPage;
```
and **sessionPage.html.js**

```javascript
const h = require('hyperscript');

const SessionPage = () => h('div', {

}, [
  h('div', {}, 'hello'),
  h('script', { src: '/js/app.js' })
]);

module.exports = SessionPage;
```
Now the **menu.html.js** can have links to all this pages:

```javascript
const h = require('hyperscript');

const menuItems = [
  {
    link: './chat.html',
    text: 'Chat'
  },
  {
    link: './session.html',
    text: 'Chat Sessions'
  }
];

const Menu = () => h('aside', {
  class: 'pure-menu'
}, [
  h('div', {class: 'pure-menu-heading'}, 'Small Chat'),
  h('ul', {class: 'pure-menu-list'}, menuItems.map((item) =>
    h('li', {class: 'pure-menu-item'}, [
      h('a', {class: 'pure-menu-link', href: item.link}, item.text)
    ])
  ))
]);

module.exports = Menu;
```
And we can expose all the pages through a simple object **index.js**

```javascript
const Base = require('./base.html');
const HomePage = require('./homePage.html');
const ChatPage = require('./chatPage.html');
const SessionPage = require('./sessionPage.html');

module.exports = {
  Base,
  HomePage,
  ChatPage,
  SessionPage
};
```
Check that we're not exposing the Menu, because is a Component, not a Page. The magic for this to work is the function fullHtml from modules/utils.js working together with the routes in modules/index.js

## Adding SPA capabilities

We have the skeletons of the pages, but this is a chat application, so reloading all the content will not be a good idea, so we need an SPA (Simple Page Application), to handle all the more dynamic parts of the app.

If you check the modules/utils.js file you'll see a webpack configuration, and its working on the fly, I will not explain that, because it's for another time, you just have to know that that is the responbibility of compiling the SPA content.

I added a route in modules/index.js that contains the route to **app.js** from the browser and calls the webpack compiler.

We need to create a folder **widget** and a file inside **app.js** with the next content:

```javascript
const Preact = require('preact');
const { h, render, Component } = Preact;

class Form extends Component {
  constructor(props) {
    super(props);
    this.changeText = this.changeText.bind(this);
    this.state = {
      text: ''
    };
  }

  changeText(e) {
    this.setState({
      text: e.target.value
    });
  }

  render(_, state) {
    return h('div', {
    }, [
      h('input', {
        type: 'text',
        value: state.text,
        onKeyUp: this.changeText
      }),
      h('div', {}, state.text)
    ]);
  }
}

render(h(Form), document.getElementById('main'));
```
Now run the dev-server  and you should get something like this:

![Chat ER](https://betotto.github.io/blog/css/assets/result_chat_1.png){: .img-content}

## Git

If you want to compare the content I created a branch for this post in https://github.com/betotto/nodejscourse/tree/StartingUI

Try to do the changes before you compare with the code.

# Reminder

If you have any recommendation or English grammar correction please write it here:  **[blog issues](https://github.com/betotto/blog/issues)**.