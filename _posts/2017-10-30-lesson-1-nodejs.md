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

## Dependencies

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

## Github

The project will be in [Github](https://github.com/betotto/nodejscourse).

# Help me

If you have any recomendation or english grammar correction please write it here:  **[blog issues](https://github.com/betotto/blog/issues)**.