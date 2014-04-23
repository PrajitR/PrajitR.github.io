---
layout: post
title: D3.js Source Code Walkthrough Part 1
tags:
- d3
- javascript
- tutorial
url: /d3-source-code-part1
summary: For those curious how the internals of d3.js is implemented
---
d3.js is an amazing data visualization library that makes it easy to create
interactive graphs on the web. It has a very simple interface but is still
extremely powerful. This post is for those who want to understand how d3 is
implemented. I assume you have some familiarity with d3 (a couple of tutorials
should be good enough).

## Approach

The d3 source code is hosted on [GitHub](https://github.com/mbostock/d3). All
the relevant code is in the `src/` and `test/` folders. Each folder in `src/`
has an `index.js` file where other files are imported in. My basic approach
is to just follow the order of imports, reading each file in succession. If
a file doesn't make sense, I just look at the corresponding tests and the 
[d3 wiki](https://github.com/mbostock/d3/wiki).

## Subfolders

`src/` has 20 subfolders in it. A description of each follows:

* `arrays/`: Contains various array methods such as `min`, `bisect`, `zip`, and `nest`. Since the array is the main form of data representation in d3, these methods form the basis for manipulating data. 
* `behavior/`: Deals mainly with the behavior of drag (including touch drags) and zoom events. For zoom, it handles redisplaying of data.
* `color/`: Deals with the different color representations in d3 (such as rgb, hcl, and hsl). Also defines convenience functions for each color representation (like brighter and darker methods).
* `compat/`: Compatibility between browsers, namely getting the current time and the ability to set styles on DOM elements.
* `core/`: General purpose methods and variables that are used throughout the d3 source (such as a class subtyping utility and a method rebinding utility).
* `dsv/`: Handles the opening and parsing of various file types (such as csv and tsv). Useful as most "real" data comes in such formats.
* `event/`: Handles the catching and dispatching of events such as touch, drag, click, and timer.
* `format/`: Convenient formatting functions for various data types.
* `geo/`: Handles geography functions and algorithms. If you're working with maps or globes, this folder serves as the backbone.
* `geom/`: Functions and algorithms for 2D geometry. Also holds the guts for creating the [Voronoi diagram](en.wikipedia.org/wiki/Voronoi_diagram). 
* `interpolate/`: Interpolation functions for various data types (such as colors, strings, and numbers). These functions do a lot of heavy lifting.
* `layout/`: "Ready-made" layouts of popular charts (such as bar charts, pie charts, force diagrams, treemaps). Useful to read if you're planning on creating your own unique type of chart for a d3 plugin.
* `locale/`: Defaults for various locales (ex. US, GB, BR, RU) and formatting for times.
* `math/`: General math functions like random distributions and trigometric functions.
* `scale/`: Defines the various scale types (ex. linear, log, ordinal) that map inputs to outputs.
* `selection/`: The real meat of d3 - selections of elements in the DOM. Allows you to select an element or group of elements, apply transformations to them, and bind data to them. Binding data is split into update, enter, and exit selections. If you could only read one folder, this would be it.
* `svg/`: The low level stuff that deals with the actual generation of svg markup (since d3 produces raw svg).
* `time/`: Dedicated to time issues (ex. minutes, months, scales, formatting).
* `transition/`: Deals with generating transitions of elements (ex. tweening, easing).
* `xhr/`: Deals with XMLHttpRequests (needed for getting data from a server).


