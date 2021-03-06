---
title: Create an action function
weight: 25
---
Create a file `action.js` for the action part of the rule.

Complete these steps:

1. Create the `action.js` file and include the object, relation, and dotenv modules.
```javascript
    require('dotenv').config({path: __dirname + '/.env'});
    var request = require('request');
    var KnowledgeObject = require('./sdk/object');
```
The action part of the rule finds the front door that is open and sends an alert to the owner of the house that the door belongs to.  An alert is sent to the home owner through a [chat UI](http://wpa-chat-bot.mybluemix.net).  
2. Add a main function to `action.js` to find the name of the door that has opened and to send an alert to the chat UI to notify the owner of the potential security breach.
```javascript
    // Main function
    function main(event) {
      console.log('in action main');
      var doorId = event[0].id;
      console.log('got door id as ' + doorId)
      return KnowledgeObject.retrieve(doorId).then((doorObj) => {
        console.log('Door name', JSON.stringify(doorObj));
        name = doorObj.attributes.name;

        var post_data = {
          'key': process.env.API_KEY,
          'alert': `Alert! Your ${name} is open!`
        };
        var headers = {
          'Content-type': 'application/json',
        };
        var options = {
          url: 'http://wpa-chat-bot.mybluemix.net/notification',
          method: 'POST',
          headers: headers,
          body: JSON.stringify(post_data)
        };

        return new Promise(function (res, rej) {
          request(options, function (err, response, body) {
            if (err || response.statusCode !== 200) {
              console.log('Error (status code ' + response.statusCode + ': ' + err + ' ' + body);
              rejData = { code: response.statusCode, body: body };
              rej(rejData);
            } else {
              resData = { body: body };
              res(resData);
            }
          });
        });
      });
    }
    
    exports.main = main;
```
3. Save your changes to `action.js`.
4. Update the `doorOpenAgent` in the `homeSecurity.js` file to run the `conditions.js` and `actions.js` functions.  Update the agent as follows:
```javascript
    // create the agents to connect to the Message Hub and subscribe to object update events.
   var doorOpenAgent = new Agent('object-update',
      conditions.main,
      actions.main);

    function runAgent() {
      Promise.all([
        doorOpenAgent.connect(),
      ]).then(function () {
        doorOpenAgent.subscribe();
        console.log('Subscription created\n\n');
      }, cleanup); //cleanup if the sub fails
    }
```
> **What to do next?**<br/>
Learn how to [invoke the rule]({{site.baseurl}}/knowledge/create-rule).

