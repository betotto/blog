---
layout: post
title: Managing Events
date: 2017-12-20 12:00:00 -0700
icon: chat
tags: nodejs
---

# Managing events

The first approach of the chat, will be using long polling requests because in many companies they have very complex network systems, so they don't allow open sockets between the browser and the servers. Long polling is widely used and it doesn't mean that is bad as everything it has advantages and disadvantages, [here](http://dsheiko.com/weblog/websockets-vs-sse-vs-long-polling) is a good explanation for it with some examples.

# Prerequisites
 
- All the previous lessons.
- Read about [Events](https://nodejs.org/docs/latest-v8.x/api/events.html) in Nodejs.
- Add 1 user in the database with id 1.
- Use the service /session/init-session to have at least one session in DB or insert the register in the session table.

# Creating files

## Creating new Message class

As the documentation says, we need to create an Event Emitter class or use that class directly (but maybe we will use more things so it's better to create a new one). The file will be **/modules/messages/NewMessage.js** with the next content:

```javascript
const EventEmitter = require('events');

class NewMessage extends EventEmitter {}

const messageEmitter = new NewMessage();

module.exports = messageEmitter;
```

## Creating Message controller

Now we need to include the message's controller to store the information in database, add a new file **/modules/messages/controller.js** and use the next info

```javascript
const querys = require('../database/querys.json');

const saveMessage = (pool, idSession, idUser, content) => {
  return new Promise((resolve, reject) => {
    pool.getConnection((err, connection) => {
      if(err) {
        reject(err);
      }
      const message = {
        idSession,
        idUser,
        content
      };
      const query = connection.query(querys.messages.save,
        [message.idSession, message.idUser, message.content],
        (error, results) => {
          connection.release();
          if (error) {
            reject(error);
          } else {
            message.id = results.insertId;
            resolve(results);
          }
        });
      global.pino.info(query.sql);
    });
  });
};

module.exports = {
  saveMessage,
};
```

## Creating Routes file

Here is where the magic happens, basically we will create an endpoint that keeps listening until another event is fired, the first approach is very rudimentary and only shows the concept; once we have more advanced our project, we will handle timeouts, multiple messages, disconnections, errors and other stuff, the routes files should be placed in **/modules/messages/routes.js** with the next content:

```javascript
const messageController = require('./controller');
const NewMessage = require('./NewMessage');

const routes = async (fastify) => {

  fastify.post('/send', async (req, reply) => {
    messageController.saveMessage(fastify.mysql, 1, 1, req.body.message)
      .then((messageResponse) => {
        NewMessage.emit('newMessage', req.body.message);
        reply.send(messageResponse);
      }).catch((error) => {
        reply.send(error);
      });
  });

  fastify.get('/events', async (req, reply) => {
    //const pool = fastify.mysql;
    const newMessageHandler = (message) => {
      req.log.info('newMessage');
      reply.send(message);
      NewMessage.removeListener('newMessage', newMessageHandler);
    };
    NewMessage.on('newMessage', newMessageHandler);
  });
};

module.exports = routes;
```

In the ***/events*** endpoint we send the response until we receive a 'newMessage' event, and we remove the listener, because this request is already processed, the ***/send*** endpoint creates a new message in the database and then it fires the 'newMessage' event with the **emit** function, currently the sessionId and the userId are fixed to 1, we will change that once we have the UI.

## Register the routes in the index file

Remember that any new module will be included in the initial configuration file, same as Session module:

```javascript
...

 const indexHtml = require('./index.html');
  3 const sessionRoutes = require('./modules/session/routes');
  1 const messagesRoutes = require('./modules/messages/routes');
    const mysql = require('./modules/database/plugin');

    fastify.register(mysql);
    fastify.register(sessionRoutes, { prefix: '/session' });
  2 fastify.register(messagesRoutes, { prefix: '/messages' });

4 fastify.get('/index.html', (request, reply) => {
  fullHtml(reply, indexHtml, config.indexHtml);
});

...
```

# Checking results

Now we have two more endpoints:

___


GET localhost:3000/messages/event

___

POST localhost:3000/messages/send

message

___

To verify how it works first call the **localhost:3000/messages/event** and keep that request running, in other request use the other endpoint **POST localhost:3000/messages/send** sending a message, you should be able to see that the message was received and the event should respond with the message that you sent. Also check that you have a new value in the database.

# Reminder

If you have any recommendation or English grammar correction please write it here:  **[blog issues](https://github.com/betotto/blog/issues)**.