---
layout: post
title: Running ST-IAT extention from Qualtrics
author: elinor_bengayev
---

In this post, I will explain how to run the Single Target IAT (ST-IAT; some call it the Single-Category IAT) in Qualtrics, using Minno.js. First, let’s make sure we all know all the concepts. You are probably reading this because you want to run an ST-IAT in Qualtrics. So, it is likely that you know what the ST-IAT is (see [this paper](https://www.researchgate.net/publication/230881869_Reliability_and_validity_of_the_Single-Target_IAT_ST-IAT_Assessing_automatic_affect_towards_multiple_attitude_objects)) and what [Qualtrics](https://www.qualtrics.com/uk/customer-experience/surveys/) is.  [Minno.js](https://minnojs.github.io/) is the software developed by [Project Implicit](http://projectimplicit.net/) to run web studies, including reaction-time tasks. 

In previous posts, Elad explained [how to run a Minno script from Qualtrics](https://minnojs.github.io/minnojs-blog/qualtrics/), and [Yoav](https://www.tau.ac.il/~baranan/index.html) explained [how to run Project Implicit’s IAT](https://minnojs.github.io/minnojs-blog/qualtrics-iat/) from Qualtrics. This post is very similar to Yoav's but it's about ST-IAT. 


### Project Implicit's MinnoJS ST-IAT extension 
We will use a [modification](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/stiat/qualtrics/qstiat6.js) of a MinnoJS ST-IAT [script](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/stiat/stiat6.js) that Project Implicit has developed. In Project Implicit, we use that script to run the study from Open Minno Suite, our platform for running web studies. Before you decide to use that script in Qualtrics, please consider using our free platform as an alternative to Qualtrics (you can read more about it [here](https://minnojs.github.io/docsite/minnosuitedashboard/)). 

The script that we created for building ST-IATs is an extension, implemented as a function that creates an ST-IAT from a few arguments (i.e., parameters) that the researcher defines. You can read more about the basic idea of using extensions in Minno on [this page](https://github.com/baranan/minno-tasks/blob/master/implicitmeasures.md).

### Into Qualtrics
The ST-IAT will run from a single question in your survey, separated from any other question by a Page Break, like this:

![Qualtrics image](https://github.com/minnojs/minnojs-blog/blob/master/images/quiat1.png)

After creating that question, click the JS icon in the ST-IAT question, and add the following code: 

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
        minnoJS(canvas, "https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/stiat/qualtrics/exampleSTIAT.js");

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

This code will run our example Qualtrics ST-IAT from [this page](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/stiat/qualtrics/exampleSTIAT.js). Start with that example by simply copying the code above to your question in your Qualtrics questionnaire. After you verify that it runs fine on your Qualtrics survey, replace it with your own ST-IAT (we’ll get to that shortly). 

**IMPORTANT NOTE:** to skip blocks when you’re testing the ST-IAT, use the key combination: Esc, Enter.

### How we define the ST-IAT

If you go to [our example](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/stiat/qualtrics/exampleSTIAT.js), you will see how simple it can be to define your own ST-IAT:

```js
define(['pipAPI', 'https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/stiat/qualtrics/qstiat6.js'], function(APIConstructor, stiatExtension){
	
	var API = new APIConstructor();
		  return stiatExtension({
		  category : { 
		    name : 'Black people', //Will appear in the data.
		    title : {
		      media : {word : 'Black people'}, //Name of the category presented in the task.
		      css : {color:'#31b404','font-size':'2em'}, //Style of the category title.
		      height : 7 //Used to position the "Or" in the combined block.
		    }, 
		    media : [ //Stimuli content as PIP's media objects
    		    	{image : 'black1.jpg'}, 
    			{image : 'black2.jpg'}, 
    			{image : 'black3.jpg'}, 
    			{image : 'black4.jpg'}, 
    			{image : 'black5.jpg'}, 
    			{image : 'black6.jpg'}
		    ], 
		    //Stimulus css (style)
		    css : {color:'#31b404','font-size':'3em'}
		  },	

  		attribute1 : 
			{
			name : 'Unpleasant', //Attribute label
			title : {
				media : {word : 'Negative'}, //Name of the category presented in the task.
				css : {color:'#31b404','font-size':'2em'}, //Style of the category title.
				height : 7 //Used to position the "Or" in the combined block.
			}, 
			media : [ //Stimuli
				{word: 'Bomb'},
				{word: 'Abuse'},
				{word: 'Sadness'},
				{word: 'Pain'},
				{word: 'Poison'},
				{word: 'Grief'}
			], 
			//Can change color and size of the targets here.
			css : {color:'#31b404','font-size':'3em'}
			},
		attribute2 : 
			{
			name : 'Pleasant', //Attribute label
			title : {
				media : {word : 'Positive'}, //Name of the category presented in the task.
				css : {color:'#31b404','font-size':'2em'}, //Style of the category title.
				height : 7 //Used to position the "Or" in the combined block.
			}, 
			media : [ //Stimuli
				{word: 'Paradise'},
				{word: 'Pleasure'},
				{word: 'Cheer'},
				{word: 'Wonderful'},
				{word: 'Splendid'},
				{word: 'Love'}
			], 
			//Can change color and size of the targets here.
			css : {color:'#31b404','font-size':'3em'}
			},

  base_url : {//Where are your images at?
    image : 'https://baranan.github.io/minno-tasks/images/'
  }}
  );
  });

```

### Use your own script file

Did you run our race ST-IAT successfully from your Qualtrics account? If you did, it's time to create your own ST-IAT. For that, you need to create a script like our example script, and post it online. If you have your own website (e.g., a home directory hosted by your university), you can put that script on that website. It is a static file so pretty much any server will be able to serve it with no issues. Or, you can use a free online service for that. One option is GitHub Pages, which is free and [easy to setup](https://help.github.com/en/github/working-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site). 

First, just copy my code and put it on your server. Change the url in the JavaScript code you entered to Qualtrics earlier, and see that my ST-IAT runs fine from your Qualtrics study. 

### Define your ST-IAT

In the first line, we tell Minno where the [full ST-IAT script](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/stiat/qualtrics/qstiat6.js) is:
`define(['pipAPI', 'https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/stiat/qualtrics/qstiat6.js'], function(APIConstructor, stiatExtension){`

Now, change your code to build the ST-IAT you need. If you’re using photos, put them in your own directory and change your ST-IAT script to search for images there: base_url: {image:’YOUR URL GOES HERE’}.
If you’re using words rather than photos, you need to update the [media](https://minnojs.github.io/minno-time/0.5/time/API.html#media) object of the categories. For instance: {word: 'Tyrone'}.

If you want to update the attributes, see how the attributes are defined in the [extension script](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/stiat/qualtrics/qstiat6.js) and override them by defining your own attributes in your ST-IAT script.
You also can just follow the comments along the script above and change the variables you want to change (e.g., the category, attributes, images, etc.).

### Processing the ST-IAT data

The original [post](https://minnojs.github.io/minnojs-blog/qualtrics/) about running MinnoJS scripts from Qualtrics explains how to process the data saved by Qualtrics. To test the data of my example, Yoav created [this R script](https://github.com/baranan/minno-tasks/blob/master/stiat/qualtrics/minno.qualtrics.test.stiat.process.rmd) and it might help you process and score your own data as well.

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

To cite our Qualtrics ST-IAT script, cite this blog post:

`Bengayev, E. (2020, July 10). Running Project Implicit’s ST-IAT from Qulatrics [Blog post]. Retrieved from https://minnojs.github.io/minnojs-blog/qualtrics-stiat/`



