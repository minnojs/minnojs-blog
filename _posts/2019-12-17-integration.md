---
layout: post
title: Manually activating MinnoJS
---

The standard way to activate MinnoJS is by having a dedicated element at page load that has the script URL built in.
It can look for example like so:

```html
<div minno-app pi-manager="/path/to/my/script.js"></div>
```

But this isn't always enough.
In particular, sometimes we want to integrate MinnoJS into an existing webpage.
Sometimes we want to dynamically choose which scripts to load.
For that purpose we can manually load MinnoJS.

In order to do this, we need to load MinnoJS 
(both the [js](https://cdn.jsdelivr.net/gh/minnojs/minno-quest@0.3/dist/pi-minno.js) and the 
[css](https://cdn.jsdelivr.net/gh/minnojs/minno-quest@0.3/dist/main.css)), 
and then activate it with the `window.minnoJS` function.

```html
<!doctype html>
<html lang=en>
<head>
    <meta charset=utf-8>
    <!-- Get css -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/minnojs/minno-quest@0.3/dist/main.css" />
</head>
<body>
    <div id="minno-element"></div>
    <!-- Get js -->
    <script src="https://cdn.jsdelivr.net/gh/minnojs/minno-quest@0.3/dist/pi-minno.js"></script>
    <!-- Activate minnoJS -->
    <script>
        var el = document.getElementById('minno-element');
        minnoJS(el, 'path/to/my/script.js');
    </script>
</body>
</html>
```