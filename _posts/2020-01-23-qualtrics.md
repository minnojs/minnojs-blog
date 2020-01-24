---
layout: post
title: Using Minno with Qualtrics
author: elad_zlotnick
---

[Qualtrics](https://www.qualtrics.com/) is a service offering online surveys, 
and a straightforward graphical user interface in order to create them.
Although Minno is a more flexible platform, Qualtrics offers an accessable server, and its interface is already familiar to many people.
Following is a detailed walk-through for running [minno-time](https://minnojs.github.io/minno-time/0.5/time/overview.html) studies in Qualtrics.
Running a MinnoJS task in Qualtrics requires effort on two levels:
First we need to set qualtrics to run the task and save the data.
then we need to set up Minno logger to save to data to qualtrics.

### Setting up Qualtrics
Let's begin by setting up Qualtrics.
We'll set up Minno to take over a single question, hide it and when it is done submit it.
First create a new "*Text Entry*" question, and set it up as a multiline question (in the setting box on the right).
We need a multiline question because we are going to use a CSV format to save the data, 
and single line questions do not save new lines.
Now, make sure that you have a *Page Break* before and after the question so that it appears on a separate page (you can do this in the actions section of the settings box on the right).

![Qualtrics image](./images/qualtrics.jpg)

Next we will setup the Javascript.
The general idea is that qualtrics activates MinnoJS, 
then MinnoJS returns the results as a string to Qualtrics which logs it into the "*Text Entry*" question.
To the left of your question there is a cog-wheel icon.
Click it and choose "Add Javascript...".
A window with a Javascript editor should pop up - this is where we do the actual integration.
The interesting part for us is the `Qualtrics.SurveyEngine.addOnload` function.
This is where we hide the original question, load MinnoJS ([and then your task](./integration)), 
[save the results](./integration-logging) returned from MinnoJS and finaly progress to the next question in your survey.
The details are described in the code's comments.

```js
Qualtrics.SurveyEngine.addOnload(function () {
    // hide question and next button
    var container = this.getQuestionContainer();
    container.querySelector('.Inner').style.display = 'none';
    this.hideNextButton();

    // load MinnoJS from the CDN (you probably don't need to change this)
    var scriptTag = document.createElement('script');
    scriptTag.src = 'https://cdn.jsdelivr.net/gh/minnojs/minno-quest@0.3/dist/pi-minno.js';
    scriptTag.onload = onLoad;
    scriptTag.onreadystatechange = onLoad;
    container.appendChild(scriptTag);

    // create the root element for Minno
    var canvas = document.createElement('div');
    container.appendChild(canvas);

    // function to proceed to next question
    var proceed = this.clickNextButton.bind(this);

    // This function gets activated only after MinnoJS is loaded
    function onLoad() {
        // Run your study (just set the correct URL)
        minnoJS(canvas, 'https://pcplab.sfo2.digitaloceanspaces.com/ezlot/settings.js');

        // MinnoJS doesn't know about Qualtrics, we pass a function to inject the results into the question
        // For some reason `piGlobal` isn't available so we attach it to `minnoJS`
        minnoJS.logger = function (value) {
            var el = container.querySelector('textarea');
            el.value = value;
        }

        // At the end of the study let MinnoJS proceed to the next question
        // We need to wait a few miliseconds for Qualtrics to register the value that we entered
        minnoJS.onEnd = function () { setTimeout(proceed, 100); }
    }
});
```

Save the Javascript, you can always come back to change it.

### Setting up your task
We now proceed to setting up MinnoJS.
First, the easy part: when the task ends, proceed to the next question.
In Qualtrics, we set the proceed function into `minnoJS.onEnd`.
Now, we want to activate it when the task ends:

```js
API.addSettings('onEnd', window.minnoJS.onEnd);
```

Next we want to setup the Logger.
The general idea is very similar to what we did [here](./integration-logging/).
The only differences are how we deal with serialization 
([CSV](https://en.wikipedia.org/wiki/Comma-separated_values) instead of [JSON](https://en.wikipedia.org/wiki/JSON)),
and saving the data (call `minnoJS.logger` instead of directly saving into an element).

JSON is a structured data format that is very flexible and easy to use.
Normaly it would by my goto format for saving data from MinnoJS.
However, Qualtrics has a hard limit of 
[20,000 characters](https://www.qualtrics.com/support/survey-platform/survey-module/editing-questions/question-types-guide/standard-content/text-entry/) 
for its text entry questions, and I wanted to use a more frugal format.
In this case I decided to use plain CSV.
This transformation occurs in the `serialize` function of the logger (see below for the full integration).
Note that I'm creating a file based on my needs (with specific headers and values),
you should fit the data that you are saving into the CSV to your specific needs.

```js
function serialize(name, logs) {
    var headers = ['group', 'latency', 'block', 'stimulus', 'correct'];
    var content = logs.map(function (log) { return [log.data.alias, log.latency, log.data.block, log.data.stimIndex, log.data.score]; });
    content.unshift(headers);
    return toCsv(content);

    function toCsv(matrice) { return matrice.map(buildRow).join('\n'); }
    function buildRow(arr) { return arr.map(normalize).join(','); }
    // wrap in double quotes and escape inner double quotes
    function normalize(val) {
        var quotableRgx = /(\n|,|")/;
        if (quotableRgx.test(val)) return '"' + val.replace(/"/g, '""') + '"';
        return val;
    }
}
```

Next we save the results by calling `minnoJS.logger` with our serialized CSV.
As we know the code that we wrote in Qualtrics will inject it into the question.

```js
function send(name, serialized){
    window.minnoJS.logger(serialized);
}
```

Put together, the logger looks like this:

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
    // we save as CSV because qualtrics limits to 20K characters and this is more efficient.
    serialize: function (name, logs) {
        var headers = ['group', 'latency', 'block', 'stimulus', 'correct'];
        var content = logs.map(function (log) { return [log.data.alias, log.latency, log.data.block, log.data.stimIndex, log.data.score]; });
        content.unshift(headers);
        return toCsv(content);

        function toCsv(matrice) { return matrice.map(buildRow).join('\n'); }
        function buildRow(arr) { return arr.map(normalize).join(','); }
        // wrap in double quotes and escape inner double quotes
        function normalize(val) {
            var quotableRgx = /(\n|,|")/;
            if (quotableRgx.test(val)) return '"' + val.replace(/"/g, '""') + '"';
            return val;
        }
    },
    // Set logs into an input (i.e. put them wherever you want)
    send: function(name, serialized){
        window.minnoJS.logger(serialized);
    }
});
```

### Getting your data
Now, when you access your data in Qualtrics, the question that you assigned to MinnoJS will have the text for a CSV file in it.
You can use any software that you like in order to extract the data.
Following is a short script I use in order to extract it in [R](https://www.r-project.org/).
You can find a more involved (and safe) version [here](https://github.com/eladzlot/minnojs-qiat/blob/master/qiat.R).

```R
# load data from Qualtrics where user id is 'ResponseId', and the CSV is in Q_43'
df = read.csv('file-from-qualtrics.csv')

# create list of CSV data frames
csvList = lapply(df[,'Q_43'], function(str) read.csv(text=str,stringsAsFactors = FALSE))

# add id to each data DF
dataPages = mapply(
    function(id, df) cbind(id,df),
    df[,'ResponseId'],
    csvList,
    SIMPLIFY = FALSE
)

# concat pages
do.call(rbind,dataPages)
```