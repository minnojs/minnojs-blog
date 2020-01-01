---
layout: post
title: Creating a custom logger
---

MinnoJS automatically sends data from questionnaires (miQuest) and about tasks (which task was visited). 
It also sends data from reaction-time tasks (those programmed in miTime), about all the trials that triggered the [‘log’ action](https://minnojs.github.io/minno-time/0.3/time/API.html#interactions-actions). 
But there is more. You can create custom loggers for integration with other systems.
In this post we will create a logger that takes data from a minno-time task and saves it as a JSON string into an input element.

In this post, we will create a logger that instead of sending the data to the server, will save it in an html `&lt;input&gt;` element.
This is usefull because many times it is easyes to submit the outcome of your study within a simple form provided by a third party.
For example, [Qualtrics](https://www.qualtricfs.com/), a popular survery service provides an easy way to create forms (including `&lt;input&gt;` elements),
but does not provide time sensitive tasks.
If you want to add a MinnoJS task to Qualtrics you could save the data into Qualtrics by injecting it into an existing `&lt;input&gt;` element.

First, we need to understand how the logger works 
(full documentation can be found [here](https://github.com/minnojs/minno-quest/blob/0.3/src/taskManager/logger/readme.md)).

MinnoJS allows you to manually modify its logging behaviour by using the `logger` setting.
The logger takes a settings object with four properties: `onRow`, `onEnd`, `serialize`, and `send`.
They are all required and each refers to a different part of the logging proccess.
We will go through each one and explain how it works.

```js
API.addSettings('logger', {
    onRow: ...,
    onEnd: ...,
    serialize: ...,
    send: ...
});
```

**`onRow`**: 
Each time Minno decides to log data (e.g., at the end of a questionnaire implemented with miQuest, or after the user used the ‘log’ action in miTime), 
it calls `onRow` with the data to be logged.
This is our chance to manipulate each data row and extract the specific data that we want (in this case only latency and trial score).
If the onRow function returns anything, it gets immediately serialized and sent to the server.
In our case we want to “send” the data only at the end, so what we want to do is set aside the data until we are ready to save it.
What we do is save it into the context until the end of the task.

```js
function(logName, log, settings, ctx){
    // ctx is an object that is constant across calls to the logger
    // we keep a logs array in ctx to collect al the logs until the end of the trial
    if (!ctx.logs) ctx.logs = [];
    ctx.logs.push({latency:log.latency, score:log.score});
}
```

**`onEnd`**: This function gets called once at the end of the task.
It is our chance to wrap up logging and do something with the logs that we saved into the context.
We tell the logger to send the logs simply by returning them 
(sending is usually to the server, in our case it will be to the `&lt;input&gt;` ):

```js
function(name, settings, ctx){
    return ctx.logs;
}
```

**`serialize`**: This function allows you to transform the log objects that are returned by `onRow` and `onEnd`.
In this case we simply transform them into [JSON](https://en.wikipedia.org/wiki/JSON) strings.

```js
function(name, logs, settings){
    return JSON.stringify(logs);
}
```

**`send`**: This function performs the actual sending of the serialized data.
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
    // Transform logs into a string
    serialize: function(name, logs, settings){
        return JSON.stringify(logs);
    },
    // Set logs into an input
    send: function(name, serialized, settings, ctx){
        document.getElementById('el').value = serialized;
    }
});
```