---
layout: post
title: Running Evaluative Priming Task extention from Qulatrics
author: gal_maimon
---

In this post, I will explain how to run the Evaluative Priming Task in Qualtrics, using Minno.js. First, let’s make sure we all know all the concepts. You are probably reading this because you want to run an Evaluative Priming Task in Qualtrics. So, it is likely that you know what the Evaluative Priming Task is (pages 9-10 [here](https://users.ugent.be/~jdhouwer/chapterbertram.pdf)) and what [Qualtrics](https://www.qualtrics.com/uk/customer-experience/surveys/) is.  [Minno.js](https://minnojs.github.io/) is the software developed by [Project Implicit](http://projectimplicit.net/) to run web studies, including reaction-time tasks. 

In previous posts, Elad explained [how to run a Minno script from Qualtrics](https://minnojs.github.io/minnojs-blog/qualtrics/), and [Yoav](https://www.tau.ac.il/~baranan/index.html) explained [how to run Project Implicit’s IAT](https://minnojs.github.io/minnojs-blog/qualtrics-iat/) from Qualtrics. This post is very similar to Yoav's but it's about Evaluative Priming Task . 


### Project Implicit's MinnoJS Evaluative Priming Task extension 
We will use a [modification](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/ep/quep5.js) of a MinnoJS Evaluative Priming Task [script](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/ep/ep5.js) that Project Implicit has developed. In Project Implicit, we use that script to run the study from Open Minno Suite, our platform for running web studies. Before you decide to use that script in Qualtrics, please consider using our free platform as an alternative to Qualtrics (you can read more about it [here](https://minnojs.github.io/docsite/minnosuitedashboard/)). 

The script that we created for building an Evaluative Priming Task is an extension, implemented as a function that creates an Evaluative Priming Task from a few arguments (i.e., parameters) that the researcher defines. You can read more about the basic idea of using extensions in Minno on [this page](https://github.com/baranan/minno-tasks/blob/master/implicitmeasures.md).

### Into Qualtrics
The Evaluative Priming Task will run from a single question in your survey, separated from any other question by a Page Break, like this:

![Qualtrics image](../images/quiat1.png)

After creating that question, click the JS icon in the Evaluative Priming Task question, and add the following code: 

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
        minnoJS(canvas, "https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/ep/qualtrics/examplepriming.js");

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

This code will run our example Qualtrics Evaluative Priming Task from [this page](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.3.9/ep/qualtrics/examplepriming.js). Start with that example by simply copying the code above to your question in your Qualtrics questionnaire. After you verify that it runs fine on your Qualtrics survey, replace it with your own Evaluative Priming Task (we’ll get to that shortly). 

**IMPORTANT NOTE:** to skip blocks when you’re testing the Evaluative Priming Task, use the key combination: Esc, Enter.

### How we define the Evaluative Priming Task

If you go to [our example](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.3.9/ep/qualtrics/examplepriming.js), you will see how simple it can be to define your own Evaluative Priming Task:

```js
define(['pipAPI', 'https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/ep/quep5.js'], function(APIConstructor, epExtension){
	var API = new APIConstructor();
	var global = API.getGlobal();
	return epExtension(
	{
			//The prime categories.
			primeCats :  [
				{
					name : 'Black people', //Will appear in the data.
					//An array of all media objects for this category.
					mediaArray : [
    				    {image : 'b01.jpg'}, 
    					{image : 'b02.jpg'}, 
    					{image : 'b03.jpg'}, 
    					{image : 'b04.jpg'}, 
    					{image : 'b05.jpg'}, 
    					{image : 'b06.jpg'}, 
    					{image : 'b07.jpg'}, 
    					{image : 'b08.jpg'}, 
    					{image : 'b09.jpg'}, 
    					{image : 'b10.jpg'}, 
    					{image : 'b11.jpg'}, 
    					{image : 'b12.jpg'}
				    ]
				}, 
				{
					name : 'White people', //Will appear in the data.
					//An array of all media objects for this category.
					mediaArray : [
    					{image : 'w01.jpg'}, 
    					{image : 'w02.jpg'}, 
    					{image : 'w03.jpg'}, 
    					{image : 'w04.jpg'}, 
    					{image : 'w05.jpg'}, 
    					{image : 'w06.jpg'}, 
    					{image : 'w07.jpg'}, 
    					{image : 'w08.jpg'}, 
    					{image : 'w09.jpg'}, 
    					{image : 'w10.jpg'}, 
    					{image : 'w11.jpg'}, 
    					{image : 'w12.jpg'}
    				]
				}
			],	

			nTrialsPerPrimeTargetPair:15, //How many trials in a block, per prime-target combination (always three blocks).
			fixationDuration : 500, 
			errorFBDuration : 1500, 
			ITIDuration : 500,

			//Set the image folder here.
			base_url : {
				image : "https://baranan.github.io/minno-tasks/images/"
			}
	});
});

```

### Use your own script file

Did you run our race Evaluative Priming Task successfully from your Qualtrics account? If you did, it's time to create your own Evaluative Priming Task. For that, you need to create a script like our example script, and post it online. If you have your own website (e.g., a home directory hosted by your university), you can put that script on that website. It is a static file so pretty much any server will be able to serve it with no issues. Or, you can use a free online service for that. One option is GitHub Pages, which is free and [easy to setup](https://help.github.com/en/github/working-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site). 

First, just copy my code and put it on your server. Change the url in the JavaScript code you entered to Qualtrics earlier, and see that my Evaluative Priming Task runs fine from your Qualtrics study. 

### Define your Evaluative Priming Task

In the first line, we tell Minno where the [full Evaluative Priming Task script](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/ep/quep5.js) is:
`define(['pipAPI', 'https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/ep/quep5.js'], function(APIConstructor, epExtension){`

Now, change your code to build the Evaluative Priming Task you need. If you’re using photos, put them in your own directory and change your Evaluative Priming Task script to search for images there: base_url: {image:’YOUR URL GOES HERE’}.
If you’re using words rather than photos, you need to update the [media](https://minnojs.github.io/minno-time/0.5/time/API.html#media) object of the categories. For instance: {word: 'Tyrone'}.

If you want to change anything in the task, search whether it is a parameter to the task as defined at the top of the [extension script](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/ep/quep5.js) and override the parameter by setting it in your Evaluative Priming Task script (as we did for a few of the parametersin the example above). 

### Processing the Evaluative Priming Task data

The original [post](https://minnojs.github.io/minnojs-blog/qualtrics/) about running MinnoJS scripts from Qualtrics explains how to process the data saved by Qualtrics. To test the data of my example, Yoav created [this R script](https://github.com/baranan/minno-tasks/blob/master/IAT/qualtrics/minno.qualtrics.iat.process.rmd) and it might help you process and score your own data as well.

If you don’t feel comfortable using R, here is how to create a csv file with the IAT data, using Excel. 
1. Download the data from Qualtrics as csv.
2. Open the csv file with Excel
3. Copy only the cells with BIAT data. For instance, because the BIAT was in Q3 in my Qualtrics questionnaire, the relevant column is under Q3. Select only the cells with that data and copy those cells.
4. Open a notepad (or notepad++ or any other simple editor) and paste the data (the cells you copied) to that file.
5. Replace (usually, using ctrl-H) the text *“block,trial* with the text *block,trial* (to remove the “ before block)
6. Replace “”” (three double quotes) with “” (two double quotes)
7. Replace “” (two double quotes) with “ (one double quotes)
8. Save that file as a csv file
9. You can open that csv file to make sure that it is fine. 

The steps in an animated gif:

![Data image](https://minnojs.github.io/minnojs-blog/images/processiat.gif)

### Any questions?

If you have any questions, you can post them in our MinnoJS Google Group, [here](https://groups.google.com/forum/?utm_medium=email&utm_source=footer#!forum/minnojs). 

### Cite Us

You used MinnoJS platform, which is cited like this:

`Zlotnick, E., Dzikiewicz, A. J., & Bar-Anan, Y. (2015). Minno.js (Version 0.3) [Computer software].`

To cite our Qualtrics Evaluative Priming Task script, cite this blog post:

`Maimon, G. (2020, July 27). Running Project Implicit’s Evaluative Priming Task from Qulatrics [Blog post]. Retrieved from https://minnojs.github.io/minnojs-blog/qualtrics-priming/`



