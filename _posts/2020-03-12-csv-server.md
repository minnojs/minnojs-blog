---
layout: post
title: Simple CSV server
author: elad_zlotnick
hidden: true
---

After setting up your account on AwardSpace, we can start creating our study.
The first page that you see when you log into your account is the dashboard.
You can always get back here using the navigation bar at the top of the screen (Image 1 a).

{% include image.html url="../images/dashboard.png" description="Image 1: the dashboard" %}

In AwardSpace, you need to create a subdomain before you can start uploading your files, let’s do that first.
Go into the domain manager (image 1 b), and choose “Create a Free Subdomain” (image 2 a). In the text input (image 2 b), choose a name for your server. We chose “minnojs-simple”, so that the URL for our study will be http://minnojs-simple.atwebpages.com/. And finally click “Create” (image 2 c).

{% include image.html url="../images/create.subdomain.png" description="Image 2: creating a subdomain" %}
 
At the bottom of the page you can now see your subdomain. If you like, you can visit it already by clicking “Open Website” (Image 3 a), though your study has not been uploaded yet.
 
{% include image.html url="../images/ready.subdomain.png" description="Image 3: the ready subdomain (you will have your own site name here)" %}

Now that we’ve created a subdomain for our study, we can start setting up the study.
Studies in the simple server are created simply by uploading files. 
We need a simple directory structure, with two files at the base directory (`index.html`, `csv.php`), and the study files in a directory of their own (`study`). In order to upload the files, return to the dashboard (image 1 a), and go into the File Manager (image 1 c). You will now see a file explorer with a single folder with the same name as the subdomain that you created (image 4).

{% include image.html url="../images/files.root.png" description="Image 4: the file manager, in the root folder" %}

Double click the folder name to navigate into it and click “Upload” (image 5 a) in order to start uploading your files. You can drag your files into the box in order to upload them (image 5 b). We first upload [index.html](), and [csv.php](). Next we’ll create a folder to hold the files of our study. Click “Create” (image 5 c) and create a directory named “study”. 
 
{% include image.html url="../images/files.site.png" description="Image 5: the file manager, in the site folder" %}

Finally, double click the “study” directory, click “Upload” again, and upload the study files from https://github.com/minnojs/simple-minno-server/tree/master/study.

{% include image.html url="../images/files.study.png" description="Image 6: the file manager, in the study folder" %}

You can now see the study running at [minnojs-simple.atwebpages.com](http://minnojs-simple.atwebpages.com/).
In order to run your study, you can go back to the dashboard, choose Domain Manager and then click “Open website”.
Go ahead, run your study twice or thrice.

You will see that a new folder has appeared in your site folder called “results” (near `index.html` and `csv.php`).
Double click this folder to see two types of files. 
There is an `.htaccess` file that has to do with server permissions. You can safely ignore it - just leave it there. Each of the other files in this folder is a [CSV](https://en.wikipedia.org/wiki/Comma-separated_values
) file with the record for a single participant in you study.

This is basically all you need in order to deploy a study on AwardSpace. The general idea should apply to any php server you deploy to. 
Experience shows that it is easiest to create new studies by transforming existing ones. 
You can use the study we created in the current tutorial, or you can use the example study in the [study.race]() folder. 
Try copying it’s content into the “study” folder and see what happens.

You can edit your files using the “edit” button in the “File Manager”, but if you intend to do a lot of editing you might want to start using a program that allows easier communication with the server (i.e. FTP client. I highly recommend [FileZilla]( https://filezilla-project.org/
)).