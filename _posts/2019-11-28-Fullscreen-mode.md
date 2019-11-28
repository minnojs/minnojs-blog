---
layout: post
title: Fullscreen mode
---

# Fullscreen mode in MinnoJS

Sometimes it is usefull to have your task run in fullscreen mode.
One way to do this, is to ask your participants to press `F11` (or `^Cmd+F` on Safari).
A much more fluent way to do this is by having full screen activated by the player automatically.
There are two problems that we need to sovle in order for this to happen.
First we need to figure out how to request fullscreen mode using raw Javascript.
Then, we need to find the way to integrate it into our tasks.

Getting to fullscreen should have been easy: simply call `document.documentElement.requestFullscreen()` and *presto* wer're done.
But the [fullscreen API](https://developer.mozilla.org/en-US/docs/Web/API/Fullscreen_API) varies somewhat across browsers,
with many requiring vendor prefixes and/or not implementing the latest specification.
Therefore we cannot use it directly and have to find a more comprehensive solution.
A quick google search brought me to [this](https://stackoverflow.com/a/7966541/1400366) Stackoverflow answer, which offers a simple script
to normalize all vendors and even support legacy browsers (read: Internet explorer prior to IE11).
With some modifications, it looks like this:

```js
var el = document.documentElement;
var rfs = el.requestFullscreen || el.webkitRequestFullScreen || el.mozRequestFullScreen || el.msRequestFullscreen;
if (rfs) rfs.call(el);
else if(window.ActiveXObject){
    // for Internet Explorer
    var wscript = new window.ActiveXObject('WScript.Shell');
    if (wscript!=null) wscript.SendKeys('{F11}');
}
```

Note however that this API only works during user interaction, so that it cannot be used maliciously.
Therefore we need to call it from within a user event such as a mouse click or keypress.

We now, start integration with MinnoJS.
The most likely place for us to activate full screen is at the debriefing phase.
We could add a [message](https://minnojs.github.io/minno-quest/0.2/manager/messages.html) with a fullscreen button and have users click it,
but wouldn't it be nice if proceeding from the debriefing page would automatically activate full screen?
Let's do exactly that. 
We begin by creating a standard message task.
We then add a [`load` hook](https://minnojs.github.io/minno-quest/0.2/manager/API.html#tasks) to run our custom Javascript
*after* the message has been loaded and its elements are available.
Within the hook, we [grab](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector) the proceed button (`[pi-message-done]`), 
and [attach a click listener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener) to it.
When the proceed button is clicked the listener will be triggered, and the code (described above will run).

```js
{
    type:'message',
    piTemplate:true,
    template: [
        'This is where the debriefing text goes',
        'Be sure to note that comencing the study will move to transfer the browser to fullscreen mode'
    ].join(''),
    load: function(){
        document.querySelector('[pi-message-done]').addEventListener('click', function(){
            var el = document.documentElement;
            var rfs = el.requestFullscreen || el.webkitRequestFullScreen || el.mozRequestFullScreen || el.msRequestFullscreen;
            if (rfs) rfs.call(el);
            else if(window.ActiveXObject){
                // for Internet Explorer
                var wscript = new window.ActiveXObject('WScript.Shell');
                if (wscript!=null) wscript.SendKeys('{F11}');
            }
        });
    }
}
```

Some older versions of Safari do not support keyboard input when in fullscreen mode.
If you are writing a study using keyboard input, and expect users to use older devices you should explicitly
note this chalenge within the instructions.
