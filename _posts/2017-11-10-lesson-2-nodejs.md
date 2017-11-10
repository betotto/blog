---
layout: post
title: Lesson 2 Adding CSS and restart development
date: 2017-10-30 12:00:00 -0700
icon: code
tags: nodejs
---

# Adding CSS and restart development

In this lesson we gonna add [Less](http://lesscss.org/) for preprocessing our CSS also we'll include [Nodemon](https://nodemon.io/) to the development process.

## Prerequisites

- Lesson 1 completed
- Text editor or IDE

### Adding CSS

Less is a CSS-preprocessor that help us to organize and write better CSS in a simpler way, the disadvantage is that we need to compile Less in order to produce CSS, but is not difficult, for development purposes we will compile on the fly, as the CSS is required; so any change that we made in a less file, will be ready on the browser when we reload the page. 

#### Installing less

To install the less compiler we just need it as node dependecy with the next command:

```bash
$ npm install --save-dev less
```

#### Compiling

In the *utils.js* file, we need to add the function that will compile less and send the compiled file to the browser.

```javascript
const lessOnFly = (reply) => {
  reply.header('Content-Encoding', 'gzip');
  reply.header('x-xss-protection', '1; mode=block');
  reply.header('x-content-type-options', 'nosniff');
  reply.header('content-type', 'text/css');

  less.render(fs.readFileSync(path.join(__dirname, '/widget/less/styles.less'), 'utf8'), {
    paths: ['./widget/less'],
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
```

don't forget to include the Less dependency in the dependencies section (top of the file) and export the new function in the export section (bottom of the file);

```javascript
//at the top
const less = require('less');

//at the bottom

module.exports = {
  fullHtml,
  lessOnFly
};
```
#### Adding CSS endpoint

In the *index.js* file we need to include the new endpoint for the CSS file:

```javascript
fastify.get('/css/styles.css', (req, reply) => {
  lessOnFly(reply);
});
```

that code should be added before calling the listen function of fastyfy. Again include the dependency:

```javascript
const { lessOnFly, fullHtml } = require('./utils');
```
#### Adding less file

Now we can add the file with our CSS content. We need to create a new folder called *widget*, it will have all the web content, including CSS, images, fonts and the javascript. Create a folder *less* inside widget and create a fila with the name *styles.less*. This is the entrypoint of our CSS code, so you can write any style that you want from here like this:

```css
body {
  color: blue;
}
```

We're done, now everytime that we reload our page the css will be compiled and sent to the browser. I know this is not good for production, but it works for the lesson 2.

### Adding Nodemon

Nodemon is a tool that restarts the server any time that we update our javascript code. It helps us to prevent starting the server manually when we are adding functionality to our application.

#### Installing Nodemon

The same as any other dependency

```bash
$ npm install --save-dev nodemon
```

#### Registering dev-server

We need to add the command to run nodemon, so in *package.json* file add an entry **scripts** then add a key to scripts **dev-server** with **nodemon index.js** verify that your package.json has something similar to:

```json
{

  //... more content
  "main": "index.js",
  "engine": ">=8.0.0",
  "scripts": {
    "dev-server": "nodemon index.js"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/betotto/nodejscourse.git"
  },
  //... more content
}
```

#### Running nodemon

Execute the next command
```bash
$ npm install --save-dev less
```

And try to modify any js file in our project and save it.

# Reminder

If you have any recommendation or English grammar correction please write it here:  **[blog issues](https://github.com/betotto/blog/issues)**.