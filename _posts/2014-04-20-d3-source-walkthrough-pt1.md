---
layout: post
title: D3.js Source Code Walkthrough Part 1
tags:
- d3
- javascript
- tutorial
url: /d3-source-code-part1
summary: For those curious how the internals of d3.js is implemented.
---
If you've ever seen an interactive visualization on a website (ex. New York Times), chances are that it is being powered by d3.js. d3, created by Mike Bostock and maintained by Bostock and Jason Davies, is the [fourth most starred repository on GitHub](https://github.com/search?q=stars%3a%3E1&s=stars&type=Repositories). It is an amazing data visualization library that makes it easy to create interactive graphs on the web. It has a relatively simple interface that is backed by a powerful and extensible engine. But if you've ever used d3, you've probably wondered how the magic _really_ is created.

This post is for those who want to understand d3's internals. I assume you have some familiarity with d3 (a couple of [tutorials](https://github.com/mbostock/d3/wiki/Tutorials) should be good enough), because I won't spend time explaining the basics.

## Approach

The d3 source code is [hosted on GitHub](https://github.com/mbostock/d3). All the relevant code is in the `src/` folder. Each subfolder in `src/` has an `index.js` which imports the relevant of the files.

My approach of reading the code was to follow the order of imports in each `index.js`. If a file didn't make sense, I looked at the corresponding tests and the [d3 wiki](https://github.com/mbostock/d3/wiki).

## Subfolders

`src/` has 20 subfolders in it. A short description of each follows:

* `arrays/`: Various array methods such as `min`, `bisect`, `zip`, and `nest`. Since the array is the main form of data representation in d3, these methods serve as the basis for manipulating data. Also defines `Map` and `Set` classes.
* `behavior/`: Behavior of drag (including touch drags) and zoom events. For zoom, it handles redisplaying of data.
* `color/`: Different color representations, such as rgb, hcl, and hsl. Also defines convenience functions for each color representation, like `brighter` and `darker` methods.
* `compat/`: Compatibility between browsers, namely getting the current time and the ability to set styles on DOM elements.
* `core/`: General purpose methods and variables that are used throughout the d3 source, such as a class subtyping utility and a method rebinding utility.
* `dsv/`: Opening and parsing of various file types, such as csv and tsv. Important because most data comes in these formats.
* `event/`: Catching and dispatching of events such as touch, drag, click. Also defines an interal timer utility that efficiently handles parallel timers.
* `format/`: Convenient formatting functions for various data types.
* `geo/`: Handles geography functions and algorithms. If you're working with maps, this folder serves as the backbone.
* `geom/`: Functions and algorithms for 2D geometry. Also has a `voronoi` folder for creating [Voronoi diagrams](en.wikipedia.org/wiki/Voronoi_diagram). 
* `interpolate/`: Interpolation functions for various data types, such as colors, strings, and numbers. Important for transitions.
* `layout/`: "Ready-made" classes for popular charts, such as bar charts, pie charts, force diagrams, and treemaps. Useful to read if you're planning on creating your own unique chart. 
* `locale/`: Defaults for various locales, such as US, GB, BR, RU, and formatting for times.
* `math/`: General math functions like random distributions and trigonometry.
* `scale/`: Various scale types that map inputs to outputs, such as linear, log, ordinal scales.
* `selection/`: The real meat of d3 - selections of elements and binding of data. Allows you to select an element or group of elements, apply transformations, and bind data. Binding data is split into update, enter, and exit selections. If you could only read one folder, you should read this.
* `svg/`: The low level code that generates the svg markup (since d3 produces raw svg).
* `time/`: Time issues, such as minutes, months, scales, formatting.
* `transition/`: Handles transitions of elements. Keeps track of all the transitions for a particular element, and uses interpolation, tweening, and easing to generate the actual transition.
* `xhr/`: Creating and sending of XMLHttpRequests, which are needed get data from a server.

And that's the general overview of d3's components! For the remainder of this post, we'll look in-depth at three folders: `core/`, `array/`, and `math/`. I have created a general description for each file that I have commented. I'll only cover a couple main files from each folder. I would recommend reading the rest of the files on your own.

## Core

`core/` defines general purpose utilities and variables that are used within the d3 source.

{% highlight javascript %}
function d3_functor(v) {
  return typeof v === "function" ? v : function() { return v; };
}

d3.functor = d3_functor;
{% endhighlight %}

`functor.js` takes one parameter: if it is a function, return it; otherwise, return a wrapper function that always returns the value. This abstracts away the need to know if a value is a function or not: instead, just call the function returned from `d3_functor`. This is used mainly in the `svg/` and `geom/` folders.

{% highlight javascript %}
function d3_class(ctor, properties) {
  try {
    for (var key in properties) {
      Object.defineProperty(ctor.prototype, key, {
        value: properties[key],
        enumerable: false
      });
    }
  } catch (e) { // Fallback.
    ctor.prototype = properties;
  }
}
{% endhighlight %}
`class.js` defines a helper method that adds properties of an object to a class. It tries to use the Javascript built-in `Object.defineProperty`, but falls back to prototype assignment if it fails. This function is used mainly when defining new classes (for example, `d3.map` in `array/map.js` and `d3.set` in `array/set.js`).

{% highlight javascript %}
var d3_subclass = {}.__proto__?

// Until ECMAScript supports array subclassing, prototype injection works well.
function(object, prototype) {
  object.__proto__ = prototype;
}:

// And if your browser doesn't support __proto__, we'll use direct extension.
function(object, prototype) {
  for (var property in prototype) object[property] = prototype[property];
};
{% endhighlight %}
`subclass.js` subtypes a class using another prototype. Used in `selection/` (where each selection also inherits the Javascript array methods), `transition/`, and `geom/`.

{% highlight javascript %}
d3.rebind = function(target, source) {
  var i = 1, n = arguments.length, method;
  while (++i < n) target[method = arguments[i]] = d3_rebind(target, source, source[method]); // source[method] returns a function.
  // `method` is first being assigned to arguments[i]. Then it gets passed into `d3_rebind` (as source[method]).
  return target;
};

function d3_rebind(target, source, method) {
  return function() {
    var value = method.apply(source, arguments); // Make `source` the method's "this" value.
    return value === source ? target : value; // setter : getter
  };
}
{% endhighlight %}
`rebind.js` copies methods (which are passed in as string arguments) from `source` to `target`. For example, if you wanted to copy the the `foo` and `bar` methods from `a` to `b`, you would call `d3.rebind(b, a, "foo", "bar")`. The code loops through all the string arguments and rebinds these methods to `target`. In `d3_rebind`, a wrapper function is returned. This function first calls the method (using `source` as the context). Then it determines the return value using d3's convention for getters and setters: if a value is set, return the object for chaining (in this case, `target`); otherwise, return the value.

{% highlight javascript %}
import "array"; // Also part of `core/`.

var d3_document = document,
    d3_documentElement = d3_document.documentElement,
    d3_window = window;

// Redefine d3_array if the browser doesn’t support slice-based conversion.
// d3_array is originally defined as function (list) { return [].slice.call(list); }
try {
  d3_array(d3_documentElement.childNodes)[0].nodeType;
} catch(e) {
  d3_array = function(list) {
    var i = list.length, array = new Array(i);
    while (i--) array[i] = list[i];
    return array;
  };
}
{% endhighlight %}
`document.js` defines d3 variables for the DOM document, window, and documentElement (which is normally the `<html></html>` tag). It also handles browser compatibility issues for array slicing. 

There are other functions in `core/` that I skip over which are also useful. For example, `ns.js` determines if a namespace prefix is used. Others are convenience functions, like `noop.js`, which defines a function that does nothing.

## Array

`array/` defines functions on arrays, which serve as the main data representation. It also defines `Map` and `Set` classes. For the most part, these functions are self explanatory and self contained.

{% highlight javascript %}
import "ascending"; // Comparator function that helps sort arrays in ascending order.

function d3_bisector(compare) {
  return {
    left: function(a, x, lo, hi) { // a = array, x = desired value
      // Optional arguments.
      if (arguments.length < 3) lo = 0;
      if (arguments.length < 4) hi = a.length;
      while (lo < hi) {
        var mid = lo + hi >>> 1; // Divides lo + hi by 2 and truncates the quotient.
        if (compare(a[mid], x) < 0) lo = mid + 1;
        else hi = mid;
      }
      return lo;
    },
    right: function(a, x, lo, hi) {
      if (arguments.length < 3) lo = 0;
      if (arguments.length < 4) hi = a.length;
      while (lo < hi) {
        var mid = lo + hi >>> 1;
        if (compare(a[mid], x) > 0) hi = mid; // Different than `left`.
        else lo = mid + 1;
      }
      return lo;
    }
  };
}

var d3_bisect = d3_bisector(d3_ascending); // Default bisect.
d3.bisectLeft = d3_bisect.left;
d3.bisect = d3.bisectRight = d3_bisect.right;

d3.bisector = function(f) {
  return d3_bisector(f.length === 1
      ? function(d, x) { return d3_ascending(f(d), x); } // Function on one variable (like function (d) { return d.date; })
      : f); // Function on two variables (like function (a, b) { return a.age - b.age; });
};
{% endhighlight %}
`bisect.js` defines functions that determine where in a sorted array an element should be inserted. It has `left` and `right` forms, which differ when the insert position is ambiguous (ex. inserting `2` in the array `[0, 2, 2, 2, 4]`). It is implemented with a binary search. The difference between the `left` and `right` is whether the `lo` or `hi` is updated when `a[mid] == x`. This has the effect of choosing either the right most or left most valid position to insert the value. There is also `d3.bisector`, which allows the user to pass in a custom comparator function (instead of using the default `ascending` function).

{% highlight javascript %}
import "../math/abs";

d3.range = function(start, stop, step) {
  // Optional parameters.
  if (arguments.length < 3) {
    step = 1;
    if (arguments.length < 2) {
      stop = start;
      start = 0;
    }
  }
  if ((stop - start) / step === Infinity) throw new Error("infinite range");
  var range = [],
       k = d3_range_integerScale(abs(step)), // Used to prevent rounding errors.
       i = -1,
       j;
  start *= k, stop *= k, step *= k; // Scale up by magnitude to deal with integer steps (instead of floating point steps).
  if (step < 0) while ((j = start + step * ++i) > stop) range.push(j / k); // Scale back down.
  else while ((j = start + step * ++i) < stop) range.push(j / k);
  return range;
};

// Finds the magnitude (base 10) that multiplies into a number to make it an integer.
function d3_range_integerScale(x) {
  var k = 1;
  while (x * k % 1) k *= 10; // While number * magnitude is still a decimal, increase magnitude by factor of 10.
  return k;
}
{% endhighlight %}
`range.js` is similar to Python's range function. Given start, stop, and step values, it computes the corresponding range of numbers. The interesting part of this implementation is the use of integer scaling. This prevents rounding errors when using floating points (`[1.0, 2.1, 3.2] != [10/10, 21/10, 32/10]`). So we must first compute the scale factor, scale the parameters up, run a regular range interval, and scale each number back down when added to the range array. This is a useful trick to keep in mind if you ever have to deal with floating point precision in the future.

{% highlight javascript %}
import "../core/class";

d3.map = function(object) {
  // Set all properties of object in a map and return it.
  var map = new d3_Map;
  if (object instanceof d3_Map) object.forEach(function(key, value) { map.set(key, value); });
  else for (var key in object) map.set(key, object[key]);
  return map;
};

function d3_Map() {}

d3_class(d3_Map, { // `d3_class` is from `core/class.js`
  has: d3_map_has,
  get: function(key) {
    return this[d3_map_prefix + key];
  },
  set: function(key, value) {
    return this[d3_map_prefix + key] = value;
  },
  remove: d3_map_remove,
  keys: d3_map_keys,
  values: function() {
    var values = [];
    this.forEach(function(key, value) { values.push(value); });
    return values;
  },
  entries: function() {
    var entries = [];
    this.forEach(function(key, value) { entries.push({key: key, value: value}); });
    return entries;
  },
  size: d3_map_size,
  empty: d3_map_empty,
  forEach: function(f) {
    for (var key in this) if (key.charCodeAt(0) === d3_map_prefixCode) f.call(this, key.substring(1), this[key]);
  }
});

var d3_map_prefix = "\0", // Prevent collision with built-in properties.
    d3_map_prefixCode = d3_map_prefix.charCodeAt(0);

function d3_map_has(key) {
  return d3_map_prefix + key in this;
}

function d3_map_remove(key) {
  key = d3_map_prefix + key;
  return key in this && delete this[key];
}

function d3_map_keys() {
  var keys = [];
  this.forEach(function(key) { keys.push(key); });
  return keys;
}

function d3_map_size() {
  var size = 0;
  for (var key in this) if (key.charCodeAt(0) === d3_map_prefixCode) ++size;
  return size;
}

function d3_map_empty() {
  for (var key in this) if (key.charCodeAt(0) === d3_map_prefixCode) return false;
  return true;
}
{% endhighlight %}
`map.js` defines a Map class. According to the d3 wiki, maps are extremely similar to regular Javascript objects, but they [avoid issues](https://github.com/mbostock/d3/wiki/Arrays#d3_map) when using built-in property names as keys (ex. `obj["__proto__"] = 42`). This is an implementation of the proposed Map in EMCAScript 6. The most important thing to note is the `d3_map_prefix` is added to every key to prevent collisions with built-in properties (ex. `map.set("__proto__", 42)` would create an internal key of `"\0" + "__proto__"`). The actual map class is defined using `d3_class` (from `core/class.js` highlighted earlier). It also offers convenience functions like `keys` and `values`.

{% highlight javascript %}
import "../core/class";
import "map";

d3.set = function(array) {
  // Add to set the array's values.
  var set = new d3_Set;
  if (array) for (var i = 0, n = array.length; i < n; ++i) set.add(array[i]);
  return set;
};

function d3_Set() {}

d3_class(d3_Set, {
  has: d3_map_has,
  add: function(value) {
    this[d3_map_prefix + value] = true; // d3_map_prefix is from `array/map` (used to prevent key collisions).
    return value;
  },
  remove: function(value) {
    value = d3_map_prefix + value;
    return value in this && delete this[value]; 
  },
  values: d3_map_keys,
  size: d3_map_size,
  empty: d3_map_empty,
  forEach: function(f) {
    // Only values added through set.add().
    for (var value in this) if (value.charCodeAt(0) === d3_map_prefixCode) f.call(this, value.substring(1));
  }
});
{% endhighlight %}
`set.js` defines a set class (also proposed in ES6) that holds unique values of an array. It uses the same namespace prefix as `d3.map` to prevent collisions. Keys are internally represented like light switches that can't be turned off after being turned on (with the exception of calling `remove()`). `Set` leverages `array/map.js` for most of its functions.

{% highlight javascript %}
d3.nest = function() {
  var nest = {},
      keys = [],
      sortKeys = [],
      sortValues,
      rollup;

  // Helper function that creates the tree.
  function map(mapType, array, depth) {
    // Deals with base case: the leaves of the tree.
    if (depth >= keys.length) return rollup 
        ? rollup.call(nest, array) : (sortValues // If a rollup function is defined, call it.
        ? array.sort(sortValues) // If no rollup, sort the leaves if requested (sortValues is a comparator function).
        : array); // If user doesn't want it sorted, return the bare array.

    var i = -1,
        n = array.length,
        key = keys[depth++], // Equivalent to: key = keys[depth]; depth++;
        keyValue,
        object,
        setter,
        valuesByKey = new d3_Map,
        values;

    // Add object to the correct key array (create array if is none).
    while (++i < n) {
      // Get the requested key value of the object, and check if it is already in the map.
      if (values = valuesByKey.get(keyValue = key(object = array[i]))) {
        values.push(object);
      } else {
        // If not in the map, add the object in an array (which allows for pushing more objects).
        valuesByKey.set(keyValue, [object]);
      }
    }

    // Define appropriate tree object and setting function for next level.
    if (mapType) {
      object = mapType();
      setter = function(keyValue, values) {
        object.set(keyValue, map(mapType, values, depth));
      };
    } else {
      object = {};
      setter = function(keyValue, values) {
        object[keyValue] = map(mapType, values, depth);
      };
    }

    // Recurse over next level.
    valuesByKey.forEach(setter);
    return object;
  }

  // Converts regular tree into tree of objects with "key" and "value" properties.
  // Also allows you to sort the keys of each level.
  function entries(map, depth) {
    // Base case of leaves.
    if (depth >= keys.length) return map;

    var array = [],
        sortKey = sortKeys[depth++];

    // Create an array of k-v pairs, using recursion for the values.
    map.forEach(function(key, keyMap) {
      array.push({key: key, values: entries(keyMap, depth)});
    });

    // Sort current level if needed (sortKey is a comparator function).
    return sortKey
        ? array.sort(function(a, b) { return sortKey(a.key, b.key); })
        : array;
  }

  nest.map = function(array, mapType) {
    return map(mapType, array, 0);
  };

  nest.entries = function(array) {
    return entries(map(d3.map, array, 0), 0);
  };

  // A new level for the tree.
  nest.key = function(d) {
    keys.push(d);
    return nest;
  };

  // Specifies the order for the most-recently specified key.
  // Note: only applies to entries. Map keys are unordered!
  // order is a comparator.
  nest.sortKeys = function(order) {
    sortKeys[keys.length - 1] = order;
    return nest;
  };

  // Specifies the order for leaf values.
  // Applies to both maps and entries array.
  nest.sortValues = function(order) {
    sortValues = order;
    return nest;
  };

  // Define a rollup function to be used on each group of leaves.
  nest.rollup = function(f) {
    rollup = f;
    return nest;
  };

  return nest;
};
{% endhighlight %}
`nest.js` is probably the most tricky function in the `array/` folder. `d3.nest` allows you to transform an array of object into a nested tree with multiple levels split on property values. It allows you to order the keys of a sublevel and apply functions to the leaves of the tree. It recursively creates deeper sublevels of the tree, until it ends up with the leaves. I would recommend checking out the [nest entry on the d3 wiki](https://github.com/mbostock/d3/wiki/Arrays#-nest) and [d3.nest examples](http://bl.ocks.org/phoebebright/raw/3176159/) to better understand how d3.nest works.

There are a lot of other functions defined in `array/`, but I'll let you look over them yourself. These are great functions to read if you find yourself needing to manipulate arrays.

## Math

The `math/` folder defines various mathematical functions, some of which are exposed and some that are internal. Like `array/`, they are self contained, though they require some mathematical knowledge.

{% highlight javascript %}
// Converts an SVG transform string into matrix representation.
import "../core/document";
import "../core/ns"; // Namespace check.

d3.transform = function(string) {
  var g = d3_document.createElementNS(d3.ns.prefix.svg, "g"); // Create svg "g" tag.
  return (d3.transform = function(string) {
    if (string != null) {
      g.setAttribute("transform", string);
     // Multiplies all transformation matrices together to get one matrix.
     var t = g.transform.baseVal.consolidate(); // SVGTransformList 
    }
    return new d3_transform(t ? t.matrix : d3_transformIdentity);
  })(string);
};

// Compute x-scale and normalize the first row.
// Compute shear and make second row orthogonal to first.
// Compute y-scale and normalize the second row.
// Finally, compute the rotation.
// Parameter m is of the form:
// [a c e]
// [b d f]
function d3_transform(m) {
  var r0 = [m.a, m.b],
      r1 = [m.c, m.d],
      kx = d3_transformNormalize(r0), // x-scale
      kz = d3_transformDot(r0, r1), // shear
      ky = d3_transformNormalize(d3_transformCombine(r1, r0, -kz)) || 0; // y-scale
  // Determinant check (ad - bc). If determinant < 0, multiply through with -1 to make determinant > 0.
  if (r0[0] * r1[1] < r1[0] * r0[1]) {
    r0[0] *= -1;
    r0[1] *= -1;
    kx *= -1;
    kz *= -1;
  }
  this.rotate = (kx ? Math.atan2(r0[1], r0[0]) : Math.atan2(-r1[0], r1[1])) * d3_degrees;
  this.translate = [m.e, m.f];
  this.scale = [kx, ky];
  this.skew = ky ? Math.atan2(kz, ky) * d3_degrees : 0;
};

d3_transform.prototype.toString = function() {
  return "translate(" + this.translate
      + ")rotate(" + this.rotate
      + ")skewX(" + this.skew
      + ")scale(" + this.scale
      + ")";
};

// Dot product.
function d3_transformDot(a, b) {
  return a[0] * b[0] + a[1] * b[1];
}

// Normalize (so values add up to 1).
function d3_transformNormalize(a) {
  var k = Math.sqrt(d3_transformDot(a, a));
  if (k) {
    a[0] /= k;
    a[1] /= k;
  }
  return k;
}

// Used to make `a` orthogonal to `b` using Gram-Schmidt (k is negative dot-product).
function d3_transformCombine(a, b, k) {
  a[0] += k * b[0];
  a[1] += k * b[1];
  return a;
}

// Identity matrix
// [a c e]  -> [1 0 0]
// [b d f]  -> [0 1 0]
var d3_transformIdentity = {a: 1, b: 0, c: 0, d: 1, e: 0, f: 0};
{% endhighlight %}
`transform.js` converts an SVG transform string into actual matrix representation. Used for matrix interpolation in `interpolate/`, which enables [impressive geometric transitions](bl.ocks.org/mbostock/1345853). The details of this file requires some basic linear algebra knowledge (how matrix transformations work).

{% highlight javascript %}
d3.random = {
  normal: function(µ, σ) {
    // Optional arguments.
    var n = arguments.length;
    if (n < 2) σ = 1;
    if (n < 1) µ = 0;
    return function() {
      var x, y, r;
      do {
        x = Math.random() * 2 - 1;
        y = Math.random() * 2 - 1;
        r = x * x + y * y;
      } while (!r || r > 1);
      return µ + σ * x * Math.sqrt(-2 * Math.log(r) / r);
    };
  },
  // Random variable whose logarithm is normally distributed.
  logNormal: function() {
    var random = d3.random.normal.apply(d3, arguments);
    return function() {
      return Math.exp(random()); // log(exp(x)) = x
    };
  },
  // Mean of m independent and identically distributed random variables between [0,1].
  bates: function(m) {
    var random = d3.random.irwinHall(m);
    return function() {
      return random() / m;
    };
  },
  // Sum of m independent and identically distributed random variables between [0,1].
  irwinHall: function(m) {
    return function() {
      for (var s = 0, j = 0; j < m; j++) s += Math.random();
      return s;
    };
  }
};
{% endhighlight %}
`random.js` defines various random number generators. The definition of `normal` requires mathematical knowledge of normal (Gaussian) distributions. 

Again, there are other functions in `math/` that I recommend you look at, though I personally don't think they are as interesting as `array/`. 

## Conclusion

In this post, we examined the roles of each folder in d3's source. We also looked in depth into three folders: `core/`, `array/`, and `math/`. Hopefully, the internals of d3 are starting to make sense. In the next post, we'll finally look at the real meat of d3: selections and transitions. Stay tuned!
