---
title: "Minifying JS and CSS files with Rider File Watchers"
datePublished: Mon Aug 13 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iwkeg00zezfs18khka5q2
slug: minifying-js-and-css-files-with-rider-file-watchers
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617381026394/mcyP9w5cf.jpeg
tags: css, js, javascript, ides

---


[In the past](http://kenbonny.net/2018/05/28/generating-specflow-files-in-rider/), I've used [File Watchers](https://www.jetbrains.com/help/rider/Settings_Tools_File_Watchers.html) in [JetBrains Rider](https://www.jetbrains.com/rider/) to automatically generate [SpecFlow](https://specflow.org/) feature files, now I want to automate the minification of javascript and CSS files.

Lets start with the easiest part: minifying javascript files. I'm using the [npm](https://www.npmjs.com) package [uglify-js](https://www.npmjs.com/package/uglify-js) for this. A quick `npm install uglify-js` adds the package to my website project and I'm ready to create a file watcher.

Rider has a "default setup" to use uglify-js as a file watcher, but it won't work with the version I installed while writing this blog post.

![default uglify js watcher](https://kenbonnyblog.files.wordpress.com/2018/08/default-uglify-js-watcher1.jpg?w=546&h=328)

![correct uglify js watcher](https://kenbonnyblog.files.wordpress.com/2018/08/correct-uglify-js-watcher1.jpg?w=546&h=328)

The default (first picture) uses `uglifyjs` as the program, but that doesn't work as `uglify` isn't a script or program. I have to specify `node` as the program and refer to the  `uglifyjs` package as an argument. The _Scope_ is also incorrect, there is no notion of `Project Files`. I chose `Open Files` as the file I'm editing always has to be open.

Personally, I prefer a descriptive name over the package name, so I set the name of the watcher to something more descriptive: Minify JS. That is just a personal preference.

> Name: Minify JS Program: node Arguments: $ContentRoot$\\node\_modules\\uglify-js\\bin\\uglifyjs $FileName$ -o $FileNameWithoutExtension$.min.js Scope: Open Files

With these settings, every time I save a javascript file, it gets minified, placed beside the normal file and has the extension _.min.js_.

> I have an update, I don't recommend css-minify anymore because it depends on about 50 packages. I recommend uglifycss.
> 
> Skip to the update at the bottom of the page to see the updated section that describes how to use uglifycss.

Minifying CSS files was a bit trickier to discover. My current setup uses the [css-minify npm package](https://www.npmjs.com/package/css-minify). After I install the package (`npm install css-minify`), I set up a new file watcher as there is no predefined template.

![minify-css](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381023903/AcUrrhM3n.jpeg)

So, again, I start with a descriptive name. This makes recognising what the watcher does easier. The _File type_ I'm using is `Cascading Style Sheet` and I set the _Scope_ again to `Open Files`.

The _Program_ is again `node` and I have to refer to the `css-minify` package with the correct arguments so the minification happens correctly. This is where it gets a little strange as the `css-minify` package only outputs to the directory `css-dist`. If I would put the file directory as the _Working directory_, like the default setting for minifing javascript, then I would need a folder structure `css\css-dist`. I prefer two folders next to eachother. In an ideal world, I'd put the minified file next to the original (like the javascript example) because then Rider nicely [nests the files](https://www.jetbrains.com/help/rider/File_Nesting_Dialog.html). Unfortunately, I can't have it all, so this is the best solution for now.

That is the reason I have to put the _Working directory_ to the content root. This also has the consequence that all my paths need to be relative to the content root.

> File type: Cascading Style Sheets Scope: Open Files Program: node Arguments: .\\node\_modules\\css-minify\\bin\\css-minify --file .\\$FilePathRelativeToProjectRoot$ Ouput paths to refresh: $ContentRoot$\\css-dist\\$FileNameWithoutExtension$.min.js Working directory: $ContentRoot$

The bigger problem that I have with the `css-minify` is that it depends on a ton of packages. I went from 2 or 3 node packages to 51. For a simple css minificaton framework, that seems to be a lot.

An alternative would be [the uglifycss package](https://www.npmjs.com/package/uglifycss) which only needs one npm package install. The reason I don't use uglifycss is that it doesn't output to a file, but just returns the minified css to the console. According to the uglyfycss documentation, all I would need to do is to write the output to a file: `node uglifycss style.css &gt; style.min.css`. And therein lies the problem, I can't let a file watcher pipe the output to a file. If I add `&gt; style.min.css` to the  _Arguments_ then I get an error from uglifycss saying it doesn't recognise the ">" parameter. I have opened a [feature request](https://github.com/fmarcia/UglifyCSS/issues/66) in the uglifycss project to add an option to write to a file. Add your voice to the request if you'd like a cleaner way to minify css. _Outdated, check the update._

Hopefully this benefits not just me, but also somebody else who wants to use file watchers to minify their javascript or CSS.

**UPDATE**: Apparently, wearing glasses isn't helping and I'm blind as a bat. [Frank Marcia](https://github.com/fmarcia), the maintainer of the uglifycss package, got back to [my request](https://github.com/fmarcia/UglifyCSS/issues/66) and pointed out that his package can write to a file using the `--output` parameter. Since uglifycss uses only one dependency and I can place the minified file next to the original, I'm a big fan of this package. So here are the new parameters to use uglifycss as CSS minifier. The reasoning is similar to the css-minify package, so I'm not going to rehash that.

> File type: Cascading Style Sheets Scope: Open Files Program: node Arguments: $ContentRoot$\\node\_modules\\uglifycss\\uglifycss $FilePath$ --output $FileDir$\\$FileNameWithoutExtension$.min.css Ouput paths to refresh: $FileDir$\\$FileNameWithoutExtension$.min.css Working directory: $ContentRoot$
