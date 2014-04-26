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

Is your brain fried? For the remainder of this post, let's look at three folders: `core/`, `array/`, and `math/`.

## Core

`core/` defines various general purpose functions and variables that are used within the d3 source, so it's a good idea to look at it.

{% highlight javascript %}
function d3_functor(v) {
  return typeof v === "function" ? v : function() { return v; };
}

d3.functor = d3_functor;
{% endhighlight %}

`functor.js` defines a function that checks if the parameter is a function. If it is, return it, otherwise, return a function that returns the constant. This function is useful when another function supports both functions and values as parameters: the business logic can just call a function without needing to know the parameter type.

{% highlight javascript %}
function d3_class(ctor, properties) {
  try {
    for (var key in properties) {
      Object.defineProperty(ctor.prototype, key, {
        value: properties[key],
        enumerable: false
      });
    }
  } catch (e) {
    ctor.prototype = properties;
  }
}
{% endhighlight %}
`class.js` defines a helper method that adds properties of an object to a function/class. This function is used mainly when defining new classes (for example, d3.map in `array/map.js` and d3.set in `array/set.js`).

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
`subclass.js` subtypes an object using another prototype. The implementation depends on browser compatibility (either prototype injection or direct extension). 

{% highlight javascript %}
d3.rebind = function(target, source) {
  var i = 1, n = arguments.length, method;
  while (++i < n) target[method = arguments[i]] = d3_rebind(target, source, source[method]); // source[method] returns function
  return target;
};

function d3_rebind(target, source, method) {
  return function() {
    var value = method.apply(source, arguments); 
    return value === source ? target : value; // setter : getter
  };
}
{% endhighlight %}
`rebind.js` copies methods (which are passed in as string arguments) from `source` to `target`. For example, if you wanted to copy the the `foo` and `bar` methods from `a` to `b`, you would call `d3.rebind(b, a, "foo", "bar")`. The code loops through all the string arguments passed in and rebinds these methods to target. In `d3_rebind`, a wrapper function is returned. This function first calls the method (using `source` as the context). Then it determines the return value using d3's convention for getters and setters: if a value is set, return the object/class (in this case, `target`); otherwise return the value.

{% highlight javascript %}
import "array"; // also part of `core/`

var d3_document = document,
    d3_documentElement = d3_document.documentElement,
    d3_window = window;

// Redefine d3_array if the browser doesn’t support slice-based conversion.
// d3_array is originally defined as [].slice.call(list)
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
`document.js` defines d3 variables for the DOM document, window, and documentElement (which is normally the `<html></html>` tag). It deals with browser compatibility issues for array slicing. 

There are other functions in `core/` that you can look at if you're interested. For example, `ns.js` determines if a namespace prefix is passed and returns an appropriate object. Other functions are merely convenience functions (ex. `noop.js` defines a function that does nothing).

## Array

`array/` defines functions on arrays that serve as the backbone for dealing with data. It also defines `Map` and `Set` classes. For the most part, these functions are self explanatory and self contained.

{% highlight javascript %}
import "ascending"; // sorts array in ascending array if passed to d3.sort

function d3_bisector(compare) {
  return {
    left: function(a, x, lo, hi) {
      if (arguments.length < 3) lo = 0;
      if (arguments.length < 4) hi = a.length;
      while (lo < hi) {
        var mid = lo + hi >>> 1;
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
        if (compare(a[mid], x) > 0) hi = mid;
        else lo = mid + 1;
      }
      return lo;
    }
  };
}

var d3_bisect = d3_bisector(d3_ascending);
d3.bisectLeft = d3_bisect.left;
d3.bisect = d3.bisectRight = d3_bisect.right;

d3.bisector = function(f) {
  return d3_bisector(f.length === 1
      ? function(d, x) { return d3_ascending(f(d), x); }
      : f);
};
{% endhighlight %}
`bisect.js` defines functions that determine where in a sorted array an element should be placed. It has `left` and `right` forms, which differ when the insert position is ambiguous (ex. inserting 2 in the array `[0, 2, 2, 2, 4]`). It is implemented by using a simple binary search, and the difference between the `left` and `right` functions is whether the `lo` or `hi` is updated. Furthermore, there is also `d3.bisector`, which takes a function that maps values before being compared (either taking 1 or 2 arguments).

{% highlight javascript %}
import "../math/abs";

d3.range = function(start, stop, step) {
  // optional parameters
  if (arguments.length < 3) {
    step = 1;
    if (arguments.length < 2) {
      stop = start;
      start = 0;
    }
  }
  if ((stop - start) / step === Infinity) throw new Error("infinite range");
  var range = [],
       k = d3_range_integerScale(abs(step)), // used to prevent rounding errors
       i = -1,
       j;
  start *= k, stop *= k, step *= k; // scale up by magnitude to deal with only integer steps
  if (step < 0) while ((j = start + step * ++i) > stop) range.push(j / k); // scale back down
  else while ((j = start + step * ++i) < stop) range.push(j / k);
  return range;
};

// Finds magnitude (10 base) that multiplies into a number to make it an integer
function d3_range_integerScale(x) {
  var k = 1;
  while (x * k % 1) k *= 10; // while number * magnitude is still a decimal, increase magnitude by factor of 10
  return k;
}
{% endhighlight %}
`range.js` is similar to Python's range function. Given a start, stop, and step, it computes the range of numbers that fit the specified parameters. The interesting part of this code is that it scales everything up to integers, and then scales back down to decimals. This is to prevent rounding errors that accumulate if you use floating points (`[1.0, 2.1, 3.2] == [10/10, 21/10, 32/10]` will give you `false`!). So the code must first compute the scale factor, scale everything up, run a regular range interval, and scale each number back down when added to the range array. This is a useful piece of code to keep in mind if you ever have to deal with pesky floating points in the future.

{% highlight javascript %}
import "../core/class";

d3.map = function(object) {
  var map = new d3_Map;
  if (object instanceof d3_Map) object.forEach(function(key, value) { map.set(key, value); });
  else for (var key in object) map.set(key, object[key]);
  return map;
};

function d3_Map() {}

d3_class(d3_Map, {
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

var d3_map_prefix = "\0", // prevent collision with built-ins
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
`map.js` defines a Map class. According to the d3 wiki, it is extremely similar to regular Javascript objects, but it avoids issues when using built-in property names as keys (ex. `obj["__proto__"] = 42). This is in fact an implementation of the proposed Map in EMCAScript 6. The most important thing to note is the `d3_map_prefix`: "\0" is added to every key to prevent key collisions with built-in properties (ex. `map.set("__proto__", 42)` would result in an internal key of `"\0" + "__proto__"`). The actual map class is defined through using `d3_class` (from `core/class.js` highlight earlier). It also offers convenience functions like `keys` and `values`.

{% highlight javascript %}
import "../core/class";
import "map";

d3.set = function(array) {
  var set = new d3_Set;
  if (array) for (var i = 0, n = array.length; i < n; ++i) set.add(array[i]);
  return set;
};

function d3_Set() {}

d3_class(d3_Set, {
  has: d3_map_has,
  add: function(value) {
    this[d3_map_prefix + value] = true; // d3_map_prefix is "\0" (used to prevent key collisions)
    return value;
  },
  remove: function(value) {
    value = d3_map_prefix + value;
    return value in this && delete this[value]; // check if key is added and delete if so
  },
  values: d3_map_keys,
  size: d3_map_size,
  empty: d3_map_empty,
  forEach: function(f) {
    // pass a value into a function only if the value was passed in through set.add
    for (var value in this) if (value.charCodeAt(0) === d3_map_prefixCode) f.call(this, value.substring(1));
  }
});
{% endhighlight %}
`set.js` defines a set class (proposed in ES6) that which holds the unique values of an array. It uses the same namespace prefix as `d3.map` to prevent collisions. `d3.set` just loops through an array ands adds each element to a constructed Set object.  It leverages `array/map.js` for most of its functions. `set.add` makes the passed in key equal true in the internal object, making it like a switch that can only be turned on when adding (can be turned off only when calling `set.remove` which deletes the key completely). 

{% highlight javascript %}
d3.nest = function() {
  var nest = {},
      keys = [],
      sortKeys = [],
      sortValues,
      rollup;

  // helper function that creates the tree
  function map(mapType, array, depth) {
    // deals with base case: the leaves of the tree
    if (depth >= keys.length) return rollup 
        ? rollup.call(nest, array) : (sortValues // if a rollup function is defined, call it
        ? array.sort(sortValues) // if no rollup, sort the leaves if requested
        : array); // if user doesn't want it sorted, return the bare array

    var i = -1,
        n = array.length,
        key = keys[depth++], // equivalent to: key = keys[depth]; depth++;
        keyValue,
        object,
        setter,
        valuesByKey = new d3_Map,
        values;

    // add values to respective key arrays (create array if is none)
    while (++i < n) {
      // get the requested key value of the object, and check if it is already in the map
      if (values = valuesByKey.get(keyValue = key(object = array[i]))) {
        values.push(object);
      } else {
        // if not in the map, add the object in an array (which allows for pushing more objects)
        valuesByKey.set(keyValue, [object]);
      }
    }

    // define appropriate tree object and setting function for next level
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

    // recurse over next level
    valuesByKey.forEach(setter);
    return object;
  }

  // converts tree into array of key-value pairs
  function entries(map, depth) {
    // base case of leaves
    if (depth >= keys.length) return map;

    var array = [],
        sortKey = sortKeys[depth++];

    // create an array of k-v pairs (recurse if necessary)
    map.forEach(function(key, keyMap) {
      array.push({key: key, values: entries(keyMap, depth)});
    });

    // sort level if needed
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

  nest.key = function(d) {
    keys.push(d);
    return nest;
  };

  // Specifies the order for the most-recently specified key.
  // Note: only applies to entries. Map keys are unordered!
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

  // define a rollup function to be used on each group of leaves
  nest.rollup = function(f) {
    rollup = f;
    return nest;
  };

  return nest;
};
{% endhighlight %}
`nest.js` is probably the most tricky method in the `array/` folder. `d3.nest` allows you to transform an array (usually of objects) into a nested tree with multiple levels. It allows you to order the keys of a sublevel and apply functions to the leaves of the tree. It uses a recursive definition in order to build up deeper sublevels of the tree, until it ends up with the leaves sublevel. 

## Math

The `math/` folder defines various mathematical functions, some of which are exposed and some that are internal. Like `array/`, they are self contained, though they are less readable than the array methods because they require knowledge of mathematical definitions.

{% highlight javascript %}
// Converts an SVG transform string into matrix representation
import "../core/document";
import "../core/ns";

d3.transform = function(string) {
  var g = d3_document.createElementNS(d3.ns.prefix.svg, "g"); // Create svg "g" tag
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
  // Determinant check (ad - bc). If determinant < 0, multiply with column by -1 and swap signs.
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

// Normalize column (so they add up to 1).
function d3_transformNormalize(a) {
  var k = Math.sqrt(d3_transformDot(a, a));
  if (k) {
    a[0] /= k;
    a[1] /= k;
  }
  return k;
}

// Used to make row a orthogonal to row b using Gram-Schmidt (k is negative dot-product).
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
`transform.js` converts an SVG transform string into actual matrix representation. Used for matrix interpolation (in `interpolate/`). 

{% highlight javascript %}
d3.random = {
  normal: function(µ, σ) {
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
  // Mean of m independent and identically distributed random variables between [0,1]
  bates: function(m) {
    var random = d3.random.irwinHall(m);
    return function() {
      return random() / m;
    };
  },
  // Sum of m independent and identically distributed random variables between [0,1]
  irwinHall: function(m) {
    return function() {
      for (var s = 0, j = 0; j < m; j++) s += Math.random();
      return s;
    };
  }
};
{% endhighlight %}
`random.js` defines various random number generators to be easily used.

## Conclusion

In this post, we looked at what each folder in d3's source does. We also looked in depth into three folders: `core/`, `array/`, and `math/`. In the next post, we'll finally look at the real meat of d3: selections. 
