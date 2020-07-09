---
layout: post
title: Running BIAT extention from Qulatrics
author: Gal Maimon
---

In this post, I explain how to run the Brief IAT (BIAT) in Qualtrics, using Minno.js. First, let’s make sure we all know all the concepts. You are probably reading this because you want to run an BIAT in Qualtrics. So, it is likely that you know what the BIAT is (see [this paper](https://psycnet.apa.org/journals/zea/56/4/283.pdf) and/or [this paper](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0110938)), and what [Qualtrics](https://www.qualtrics.com/uk/customer-experience/surveys/) is.  [Minno.js](https://minnojs.github.io/) is the software developed by [Project Implicit](http://projectimplicit.net/) to run web studies, including reaction-time tasks. 

In previous posts, Elad explained [how to run a Minno script from Qualtrics](https://minnojs.github.io/minnojs-blog/qualtrics/), and [Yoav](https://www.tau.ac.il/~baranan/index.html) explained [how to run Project Implicit’s IAT](https://minnojs.github.io/minnojs-blog/qualtrics-iat/) from Qualtrics. This is the same post as Yoav's but about the BIAT. 


### Project Implicit's MinnoJS BIAT extension 
We will use a [modification](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/BIAT/qualtrics/qbiat6.js) of a BIAT MinnoJS [script](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/BIAT/biat6.js) that Project Implicit has developed. In Project Implicit, we use that script to run the study from Open Minno Suite, our platform for running web studies. Before you decide to use that script in Qualtrics, please consider using our free platform as an alternative to Qualtrics (you can read more about it [here](https://minnojs.github.io/docsite/minnosuitedashboard/)). 

The script that we created for building BIATs is an extension, implemented as a function that creates an BIAT from a few arguments (i.e., parameters) that the researcher defines. You can read more about the basic idea of using extensions in Minno on [this page](https://github.com/baranan/minno-tasks/blob/master/implicitmeasures.md).

### Into Qualtrics
The BIAT will run from a single question in your survey, separated from any other question by a Page Break, like this:
![Qualtrics image](https://github.com/minnojs/minnojs-blog/blob/master/images/quiat1.png)

After creating that question, click the JS icon in the BIAT question, and add the following code: 
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
        minnoJS(canvas, 'https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/BIAT/qualtrics/exampleBIATimage.js');
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

This code will run our example Qualtric BIAT from [this page](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/BIAT/qualtrics/exampleBIATimage.js). Start with that example. After you verify that it runs fine on your Qualtrics survey, replace it with your own BIAT (we’ll get to that shortly). 

**IMPORTANT NOTE:** to skip blocks when you’re testing the BIAT, use the key combination: Esc, Enter.

### How we define the BIAT

If you go to [our example](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/BIAT/qualtrics/exampleBIATimage.js), you will see how simple it can be to define your own BIAT:
```js
define(['pipAPI', 'https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/BIAT/qualtrics/qbiat6.js'], function(APIConstructor, iatExtension){
  var API = new APIConstructor();

	
	return iatExtension({
		practiceCategory1 : 
			{
				name : 'Mammals', //Will appear in the data.
				title : {
					media : {word : 'Mammals'}, //Name of the category presented in the task.
					css : {color:'#31b404','font-size':'1.8em'}, //Style of the category title.
					height : 4, //Height (because we need to know where to put the next item in the title)
					startStimulus : { 
					//If you're using a startStimulus, set here. If not, set the parameter showStimuliWithInst to false (see later below)
						media : {word : 'Dogs, Horses, Cows, Lions'}, 
						css : {color:'#31b404','font-size':'1em'}, 
						height : 2
					}
				}, 
				stimulusMedia : [ //Stimuli content as PIP's media objects
					{word : 'Dogs'}, 
					{word : 'Horses'}, 
					{word : 'Lions'}, 
					{word : 'Cows'}
				], 
				//Stimulus css (style of the stimuli)
				stimulusCss : {color:'#31b404','font-size':'2em'}
			},	
			practiceCategory2 : 
			{
				name : 'Birds', 
				title : {
					media : {word : 'Birds'}, 
					css : {color:'#31b404','font-size':'1.8em'}, 
					height : 4,
					startStimulus : {
						media : {word : 'Pigeons, Swans, Crows, Ravens'}, 
						css : {color:'#31b404','font-size':'1em'}, 
						height : 2
					}
				}, 
				stimulusMedia : [ //Stimuli content as PIP's media objects
					{word : 'Pigeons'}, 
					{word : 'Swans'}, 
					{word : 'Crows'}, 
					{word : 'Ravens'}
				], 
				//Stimulus css
				stimulusCss : {color:'#31b404','font-size':'2em'}
			},
			categories : [  //As many categories you need.
				{
					name : 'Black People', //Will appear in the data.
					title : {
						media : {word : 'Black People'}, //Name of the category presented in the task.
						css : {color:'#31b404','font-size':'1.8em'}, //Style of the category title.
						height : 4, //Height (because we need to know where to put the next item in the title)
						startStimulus : { 
						//If you're using a startStimulus, set here. If not, set the parameter showStimuliWithInst to false (see later below)
						media : {image : 'blacks.jpg'}, 
							css : {color:'#31b404','font-size':'1em'}, 
							height : 2
						}
					}, 
					stimulusMedia : [ //Stimuli content as PIP's media objects
					{image : 'black1.jpg'}, 
        			{image : 'black2.jpg'}, 
        			{image : 'black3.jpg'}, 
        			{image : 'black4.jpg'}, 
        			{image : 'black5.jpg'}, 
        			{image : 'black6.jpg'}
					], 
					//Stimulus css (style of the stimuli)
					stimulusCss : {color:'#31b404','font-size':'2em'}
				},	
				{
					name : 'White people', 
					title : {
						media : {word : 'White people'}, 
						css : {color:'#31b404','font-size':'1.8em'}, 
						height : 4,
						startStimulus : {
							media : {image : 'whites.jpg'},  
							css : {color:'#31b404','font-size':'1em'}, 
							height : 2
						}
					}, 
					stimulusMedia : [ //Stimuli content as PIP's media objects
					{image : 'yf1.jpg'}, 
        			{image : 'yf4.jpg'}, 
        			{image : 'yf5.jpg'}, 
        			{image : 'ym2.jpg'}, 
        			{image : 'ym3.jpg'}, 
        			{image : 'ym5.jpg'}
					], 
					//Stimulus css
					stimulusCss : {color:'#31b404','font-size':'2em'}
				}
			],
			base_url : {//Where are your images at?
			image : 'https://baranan.github.io/minno-tasks/images/'
				
			} 
	});
});




```
In the first line, we tell Minno where the [full BIAT script](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/BIAT/qualtrics/qbiat6.js) is:
`define(['pipAPI', 'https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/BIAT/qualtrics/qbiat6.js'], function(APIConstructor, iatExtension){`

At the beginning of [that script](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/BIAT/qualtrics/exampleBIATimage.js), you will see the practice category. Each BIAT task begins with a practice block that includes 8 trials. In our case, the practice categories are Mammals/Birds. 
After the practice categories, you will see the categories in the actual test. You can set as many categories as you would like, but it is common to use the BIAT like the IAT is used: with two categories and two attributes. If you include more than two categories, each block will show one of those categories as the focal category and all the other categories as the non-focal categories. In our example there are only two categories, called *White people* and *Black people* (the famous Race BIAT).


### Use your own script file

Did you run our race BIAT successfully from your Qualtrics account? If you did, it's time to create your own BIAT. For that, you need to create a script like our example script, and post it online. If you have your own website (e.g., a home directory hosted by your university), you can put that script on that website. It is a static file so pretty much any server will be able to serve it with no issues. Or, you can use a free online service for that. One option is GitHub Pages, which is free and [easy to setup](https://help.github.com/en/github/working-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site). 

First, just copy my code and put it on your server. Change the url in the JavaScript code you entered to Qualtrics earlier, and see that my BIAT runs fine from your Qualtrics study. 

### Define your BIAT

Now, change your code to build the BIAT you need. If you’re using photos, put them in your own directory and change your BIAT script to search for images there: base_url: {image:’YOUR URL GOES HERE’}.
If you’re using words rather than photos, you need to update the [media](https://minnojs.github.io/minno-time/0.5/time/API.html#media) object of the categories. For instance: {word: 'Tyrone'}.

If you want to update the attributes, see how the attributes are defined in the [extension script](https://cdn.jsdelivr.net/gh/baranan/minno-tasks@0.*/BIAT/qualtrics/qbiat6.js) and override them by defining your own attributes in your BIAT script.


### Processing the BIAT data

The original [post](https://minnojs.github.io/minnojs-blog/qualtrics/) about running MinnoJS scripts from Qualtrics explains the how to process the data saved by Qualtrics. 

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

If you have any questions, you post questions in our MinnoJS Google Group, [here](https://groups.google.com/forum/?utm_medium=email&utm_source=footer#!forum/minnojs). 

### Cite Us

You used MinnoJS platform, which is cited like this:

`Zlotnick, E., Dzikiewicz, A. J., & Bar-Anan, Y. (2015). Minno.js (Version 0.3) [Computer software].`

To cite our Qualtrics-BIAT script, cite this blog post:

`Maimon, G. (2020, July 9). Running Project Implicit’s BIAT from Qulatrics [Blog post]. Retrieved from https://minnojs.github.io/minnojs-blog/qualtrics-biat/`



