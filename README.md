Mongo Asynchronous Multiple Requests
====================================

This class will permit you to get the result of multiple MongoDB requests in one callback function, very easily.

```js
    var asyncMultiRequest = require('./mongoAsyncMultiRequest.js');

    var task = new asyncMultiRequest("mongodb://127.0.0.1:27017/test",
    {
        insertUser : function(db, next, variables) {

            var user = {
                firstName : variables.firstName,
                lastName : variables.lastName
            }
            db.collection('users').insert(user, null, next);
        },

        insertTicket : {
            dependencies : ['insertUser'],
            task : function(db, next, variables, results) {

                var ticket = {
                    author: results.insertUser[0]._id,
                    message: variables.message
                };
                db.collection('tickets').insert(ticket, null, next);
            }
        },

        users : {
            dependencies : ['insertUser'],
            task : function(db, next, variables) {
                db.collection('users').find().toArray(next);
            }
        },

        tickets : {
            dependencies : ['insertTicket', 'users'],
            task : function(db, next, variables, results) {
                console.log("`tickets` task is ran only when user `task` is done.");
                console.log(results.users);
                db.collection('tickets').find().toArray(next);
            }
        },

    }, function(err, results) {
        if (err) throw err;


    });

    task.run({
        firstName : "John",
        lastName : "Doe",
        message : "Hello Everybody!" });

```
