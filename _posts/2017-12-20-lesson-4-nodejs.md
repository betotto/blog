---
layout: post
title: Lesson 4 Configuring database
date: 2017-12-20 12:00:00 -0700
icon: chat
tags: nodejs
---

# Configuring database

There is a lot of things to do to create the chat, but one of the most important things is to store the information that the chat will generate. We gonna include [Mysql](https://www.mysql.com/) as primarly database.

# Prerequisites
 
- You need a Mysql engine where you have the DBA role.
- All the previous lessons
- Create the database script for reproduce the entity relation model.

# Adding Mysql to the project

When we're working with SQL databases one of the firsts steps is to define the data model, so we will working on the chat session model, with two events create and close and for that we will use the next data model:

![Chat ER](https://betotto.github.io/blog/css/assets/chatDB_1.png "Chat ER"){:class="img-content"}

## Installing dependencies

We need to install the next dependencies:

- fastify-plugin
- mysql

with the command:

```bash
$ npm install --save fastify-plugin mysql
```

## Defining the HTTP request handle process

The way that we will handle the requests in this project will be like this:

![Request Process](https://betotto.github.io/blog/css/assets/Request.png "Request Process"){:class="img-content"}

For every request it will be processed by the routes in every model, in the routes file every input data will be validated and formatted as well as the output data. The route can call one or many controller functions, but the login inside routes should be only:

- Routing controllers between modules.
- Format input/output data.
- Validate input/output data.

Each controller function can execute database querys, consume external services or apply any logic required to generate the output, a controller can call other controllers only of the same module. Once we have the output required, the router will send the output to the client.

## Configuring database

One of the greatest things in fastify are the plugins, it allows us to easily share objects or data between endpoints, we will have a module called database with all the querys of the application and the MySQL plugin.

1. Add MySQL parameters

In the config/index.js file, add the database server info:

```javascript
module.exports = {
  port: 3000,
  indexHtml: {
    title: 'nodejscourse'
  },
  mysqlParams: {
    connectionLimit: 2,
    host: 'localhost',
    user: 'nodejs',
    password: 'yourpassword',
    database: 'yourdatabase'
  }
};
```

We will keep two connections in the pool per nodejs instance.

2. Create the querys

Create a file in modules/database called **querys.json** that will contain all the querys of the application, this is useful for the dba and for us.

```json
{
  "session": {
    "init": "insert into SESSION (STATUS) values (?)",
    "close": "UPDATE SESSION set CLOSINGDATE = CURDATE(), STATUS = ? WHERE IDSESSION = ?"
  }
}
```

3. Create the plugin for fastify

Create a file in modules/database called **plugin.js** with the content: 

```javascript
const fp = require('fastify-plugin');
const mysql = require('mysql');
const config = require('../../conf');
const pool  = mysql.createPool(config.mysqlParams);

const mysqlPool = async (fastify) => {
  fastify.decorate('mysql', pool);
};

module.exports = fp(mysqlPool);
```

Now we have the mysql pool available for all the requests.

4. Adding routes

Create a file in modules/session called **routes.js** with the content:

```javascript
const sessionController = require('./controller');

const routes = async (fastify) => {
  fastify.get('/init-session', async (req, reply) => {
    const pool = fastify.mysql;
    sessionController.initSession(pool, req.log).then((response) => {
      reply.send(response);
    }).catch((err) => {
      reply.send(err);
    });
  });

  fastify.post('/close-session', async (req, reply) => {
    const pool = fastify.mysql;
    sessionController.closeSession(pool, req.log, req.body.idSession).then((response) => {
      reply.send(response);
    }).catch((err) => {
      reply.send(err);
    });
  });
};

module.exports = routes;
```
5. Adding controllers

Create a file in modules/session called **routes.js** with the content:

```javascript
const querys = require('../database/querys.json');

const sessionStatus = {
  OPEN: 1,
  CLOSED: 2
};

const initSession = (pool, log) => {
  return new Promise((resolve, reject) => {
    pool.getConnection((err, connection) => {
      if(err) {
        reject(err);
      }
      const session = {
        status: sessionStatus.OPEN
      };
      const query = connection.query(querys.session.init, [session.status], (error, results) => {
        connection.release();
        session.id = results.insertId;
        if (error) {
          reject(error);
        }
        resolve(session);
      });
      log.info(query.sql);
    });
  });
};

const closeSession = (pool, log, idSession) => {
  return new Promise((resolve, reject) => {
    pool.getConnection((err, connection) => {
      if(err) {
        reject(err);
      }
      const query = connection.query(querys.session.close, [sessionStatus.CLOSED, idSession], (error, results) => {
        connection.release();
        if (error) {
          reject(error);
        }
        resolve(results);
      });
      log.info(query.sql);
    });
  });
};

module.exports = {
  initSession,
  closeSession
};
```

6. Registering endpoints

In the index.js file we need to include 1, 2, 3 and 4 lines between the the 5 and 6 lines (that exist) to register the endpoints:

```javascript
...

5 const indexHtml = require('./index.html');
  1 const sessionRoutes = require('./modules/session/routes');
  2 const mysql = require('./modules/database/plugin');

  3 fastify.register(mysql);
  4 fastify.register(sessionRoutes, { prefix: '/session' });

6 fastify.get('/index.html', (request, reply) => {
  fullHtml(reply, indexHtml, config.indexHtml);
});

...
```


## Checking results

Now we have two endpoints:

___


GET localhost:3000/session/init-session

___

POST localhost:3000/session/close-session

idSession

___

# Reminder

If you have any recommendation or English grammar correction please write it here:  **[blog issues](https://github.com/betotto/blog/issues)**.