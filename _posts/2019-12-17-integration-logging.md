---
layout: post
title: Creating a custom logger
---

MinnoJS has several standard loggers that are used to log data that is gathered.
One of its less known capabilities it to create custom loggers for integration with other systems.
In this post we will create a logger that takes data from a minno-time task and saves it as a JSON string into an input element.
This sort of logger is useful for the integration of MinnoJS with external services.

First we need to understand how the logger works 
(full documentation can be found [here](https://github.com/minnojs/minno-quest/blob/0.3/src/taskManager/logger/readme.md)).

The logger takes a settings object with four properties: `onRow`, `onEnd`, `serialize`, and `send`.
They are all required and each refers to a different part of the logging proccess.
We will go through each on and explain how it works.

*`onRow`*: Each log gets called once with `onRow`.
This is our chance to manipulate each data row and extract the specific data that we want (in this case only latency and trial score).
If the `onRow` function returns anything it gets immidiately serialized and sent.
In our case we want to "send" the data only at the end, so what we want to do is set aside the data until we are ready to save it.
What we do is save it into the context until the end of the task.

```js
function(logName, log, settings, ctx){
    if (!ctx.logs) ctx.logs = [];
    ctx.logs.push(log);
}
```

*`onEnd`*: This function gets called once at the end of the task.
It is our chance to wrap up logging and do something with the logs that we saved into the context.
We tell the logger to log the logs simply by returning them:

```js
function(name, settings, ctx){
    return ctx.logs;
}
```

*`serialize`*: This function allows you to transform the log objects that are returned by `onRow` and `onEnd`.
In this case we simply transform them into [JSON](https://en.wikipedia.org/wiki/JSON) strings.

```js
function(name, logs, settings){
    return JSON.stringify(logs);
}
```

*`send`*: This function performs the actual sending of the serialized data.
In our case all we want to do is set the results in a text input.
If you are integrating with a server, this is where you would put the code to update it.

```js
function(name, serialized, settings, ctx){
    document.getElementById('el').value = serialized;
}
```

Following is the ful code for updating the logger.

```js
API.addSettings('logger', {
    // gather logs in array
    onRow: function(logName, log, settings, ctx){
        if (!ctx.logs) ctx.logs = [];
        ctx.logs.push(log);
    },
    // onEnd trigger save (by returning a value)
    onEnd: function(name, settings, ctx){
        return ctx.logs;
    },
    serialize: function(name, logs, settings){
        return JSON.stringify(logs);
    },
    send: function(name, serialized, settings, ctx){
        document.getElementById('el').value = serialized;
    }
});
```