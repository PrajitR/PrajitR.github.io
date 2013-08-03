---
layout: post
title: A Quick Tour of Haskell Syntax 
tags:
- haskell
- tutorial
url: /quick-tour-haskell-syntax
summary: For programmers who want a quick and dirty way to parse Haskell syntax. 
---
If you're a programmer who wants to parse Haskell for a blog post or wants a cursory overview of the language, this post is for you. It's going to be somewhat longer than the [learnxinyminutes][] style, as it will go a little more in depth. Without further ado, let's get started.
[learnxinyminutes]: learnxinyminutes.com/docs/haskell/ "LearnXinYMinutes Haskell"

Starting with the typical basic types:
{% highlight haskell %}
-- inline comment
{- block comment -}
7 -- numbers
3.0
True -- booleans
False 
'p' -- characters
'j'
"The slow brown fox." -- strings
[1, 2, 3] -- lists
{% endhighlight %}

Strings are actually just a list of characters. `"Hello"` is the same as `['H','e','l','l','o']`. Moving on to basic functions.
{% highlight haskell %}
7 + 12 -- 19
3 * 11 -- 33
6 - 2 -- 4
11 / 8 -- 1.375

-- Integer division
11 `div` 8 -- 1

not True -- False
10 == 9 -- False
10 /= 9 -- True
True && False -- False
False || True -- True
4 > 2 -- True
18 <= 13 -- False
{% endhighlight %}

Couple of things to notice. `not` is a function: it takes a boolean value, and negates it. Functions in Haskell do not require parentheses. The general syntax is `func arg1 arg2 arg3 ...`. However, parentheses may be required in some expressions: `not (3 < 5)`, evaluates to `True`, but `not 3 < 5` is invalid, as Haskell will interpret the expression as `(not 3) < 5`, erroring out when it tries to boolean negate a number.

Also, `` 11 `div` 8 `` is the same thing as `div 11 8`. Backticks around any function call will allow you to use it in infix style, as opposed to prefix. It's just syntactic sugar for some functions that read better in infix style. 

Here are some functions on lists: 

{% highlight haskell %}
1 : [2, 3] -- [1, 2, 3]
'C' : "at" -- "Cat" (Remember strings are lists of characters)
True : [] -- [True]
1 : 2 : 3 : [] == [1, 2, 3] -- True

elem 3 [1, 4, 73, 12] -- False
'H' `elem` "Highgarden" -- True 
[1, 2] ++ [3, 4] -- [1, 2, 3, 4]
"Hello " ++ "World" -- "Hello World" 
head [1, 2, 3] -- 1
tail [1, 2, 3] -- [2, 3]
last [1, 2, 3] -- 3
init [1, 2, 3] -- [1, 2]
"Casterly Rock" !! 3 -- 't'

null [] -- True
null [1, 2, 3] -- False
length "Dorne" -- 5

(1, 2)
("Hello", True, 2)
fst (6, "Six") -- 6
snd (6, "Six") -- "Six" 
{% endhighlight %}

Notice how `1 : 2 : 3 : []` is the same as `[1, 2, 3]`. Lists are just elements `:` together, with an empty list (`[]`) at the end, similar to Lisp. `(elem1, elem2, elem3, ...)` are called tuples. A tuple can contain different types, whereas a list may only contain the same type for all its elements.

Let's define our own function.

{% highlight haskell %}
id :: a -> a
id x = x
{% endhighlight %}

This function takes a value, and returns the value untouched.

The top line is the type declaration. The name of the function (`id`), followed by `::`, and finally the type signature. It says, "Take a value of type `a`, which can be anything, and return another value of the same type `a`". 

The bottom line is the computation. It takes a parameter and returns the parameter untouched. This might be a little confusing so let's look at another example.

{% highlight haskell %}
const :: a -> b -> a
const x y = x
{% endhighlight %}

This function takes two values, and returns the first one, ignoring the second.

The type declaration says, "Take a value of any type `a`, take another value of any type `b` (can be the same or different type as `a`), and return a value of type `a`". The type after the final `->` is the return type, and all the types before it are parameter types, also delimited by `->`s. 

The function takes in two variables, `x` and `y`, and just returns `x` untouched, similar to `id` above. These functions have been extremely simple, so let's look at a couple of slightly more challenging functions.

{% highlight haskell %}
concatenate3 :: String -> String -> String -> String
concatenate3 x y z = x ++ y ++ z

allEqual :: (Eq a) => a -> a -> a
allEqual x y z = x == y && y == z
{% endhighlight %}

In `concatenate3`, the type signature says it takes 3 `String`s and returns a `String`. Notice how `String` is a specific type, whereas `a` and `b` were general. You can mix and match specific and general types in type signatures. The function just concatenates the 3 `String`s using `++`. Again, we do not need parentheses, as Haskell will interpret the statement as `(x ++ y) ++ z`, which is valid.

In `allEqual`, we see: `(Eq a) =>`. For now, just think of this as restricting the domain of types `a` to be types that can be compared for equality. Multiple restrictions are also allowed, i.e. `(Eq a, Ord a, Num b, ...) =>`. The function just compares the parameters to check if they are all equal.

Haskell makes creating anonymous functions simple.

{% highlight haskell %}
(\x -> x + x)
(\x y -> x + y) 
(\x -> 10 + x) 5 -- 15
{% endhighlight %}

Anonymous functions are wrapped by parentheses. They lead with a `\`, then the arguments delimited by spaces, then a `->`, and finally the computation.

In Haskell, functions are first-class, meaning functions can be passed in as arguments to other functions, returned from functions, assigned from variables, and held in data structures, such as lists.

{% highlight haskell %}
applyAndConcat :: (String -> String) -> (String -> String) -> String -> String
applyAndConcat f g x = (f x) ++ (g x)

applyAndConcat tail init "Rickon" -- "ickonRicko"
-- (tail "Rickon") ++ (init "Rickon")
-- "ickon" ++ "Ricko"
-- "ickonRicko"
{% endhighlight %}

`applyAndConcat` takes in two functions, each of which take a `String` and return a `String`, takes in a `String`, and returns a `String`. Functions in type signatures are wrapped by parentheses: `(String -> String)`. `applyAndConcat` applies both functions to the same input `String`, and returns the concatentation of the results. 

{% highlight haskell %}
(.) :: (b -> c) -> (a -> b) -> (a -> c)
(.) f g = (\x -> f (g x)) 
-- alternatively
f . g = (\x -> f (g x))
{% endhighlight %}

The function `.` is called function composition, similar to the definition you see in math ("fog" notation). It returns a function that takes in an input, passes it to the function `g`, and then pipes the result of `g` into `f`. Looking at the type signature, it makes sense. Given a value of type `a`, you pass it into function `g`, which returns a value of type `b`, and the value of type `b` gets passed into the function `f`, which returns a value of type `c`. Looking at it like it's a black box, the type of the function composition is `(a -> c)`.

{% highlight haskell %}
($) :: (a -> b) -> a -> b
f $ x = f x
{% endhighlight %}

You often see `$` in Haskell code. Looking at the definition, it seems as if `$` does nothing. For the most part, `$` is used not for computation, but for reducing the amount of parentheses needed. `not $ 3 < 5` is the exact same as `not (3 < 5)`. It's just a matter of style. Also note that `$` is not a special operator - it's just a simple function, just like everything else. 

{% highlight haskell %}
add x y = x + y

addToThree = add 3
addToThree 6 -- 9
-- add 3 6

multiplyThreeNums x y z = x * y * z

times32 = multiplyThreeNums 8 4
times32 10 -- 320
-- multiplyThreeNums 8 4 10

twoNumsTimes10 = multiplyThreeNums 10
twoNumsTimes10 6 2 -- 120
-- multiplyThreeNums 10 6 2
{% endhighlight%}

In Haskell, you can partially apply a function. That is, given a function that takes `n` arguments, you can partially apply `k` arguments (where `k < n`), and you'll end up with a function that takes `n-k` arguments. Concretely, in the example, we see `add`, which takes two arguments, and adds them together. `addToThree` is a partially applied `add`, where the first argument of `add` is predefined as `3`. Since the first argument is already defined, `addToThree` only requires one more argument. Same idea for `multiplyThreeNums`, where you can partially apply 1 or 2 arguments. 

Note that I didn't add a type signature to any of the functions. In Haskell, if you don't add a type signature, the compiler will automatically derive the type, allowing you the choice of whether or not to document the function.

{% highlight haskell %}
addFour w x y z = 
  let a = w + x
      b = y + a
  in z + b

addFour w x y z =
   z + b
 where
   a = w + x
   b = y + a
{% endhighlight %} 

`let ... in ...` and `... where ...` let you build up computation and store them in local variables. Picking between `let` and `where` is a matter of preference - they achieve the same thing.

{% highlight haskell %}
fib 0 = 1
fib 1 = 1
fib n = fib (n - 1) + fib (n - 2)
{% endhighlight %}

Here we see the typical fibonacci sequence using recursion. It's no different compared to most other languages; the function calls itself. The `fib 0 = 1` and `fib 1 = 1` are interesting though. This is a case of "pattern matching". Haskell goes down the list and tries to find a matching definition. It first checks if n is 0, and if so, returns the value associated with it (`fib 0 = 1`). If n is not 0, then it goes down the list, and checks if n is 1, and returns the associated value if so (`fib 1 = 1`). Otherwise, it goes to the catch-all definition, and returns that value (`fib n = fib (n - 1) + fib (n - 2)`). Pattern matching is extremely useful in Haskell, and you're bound to see it crop up in many places.

{% highlight haskell %}
fib n
  | n < 2 = 1
  | otherwise = fib (n - 1) + fib (n - 2)

fib n = 
  case n of
    0 -> 1
    1 -> 1
    _ -> fib (n - 1) + fib (n - 2)

fib n =
  if n < 2
   then 1
   else fib (n - 1) + fib (n - 2)
{% endhighlight %} 

These are just variations of the same `fib` function, using different syntax. The first one is called "guard expressions", and uses `|` followed by different cases and their values. `otherwise` is just a catch-all value (in fact, `otherwise` is the exact same as `True`). Note that there is no `=` before a guard expression. `case ... of ...` is also pattern matching, except that you can also use it in the middle of a computation. `if ... then ... else ...` is just a regular if statement. Note that every `if` must have a corresponding `else` in Haskell. 

Let's define some bread-and-butter Haskell functions.

{% highlight haskell %}
map :: (a -> b) -> [a] -> [b]
map _ [] = []
map f (x:xs) = (f x) : (map f xs)

addToThree x = x + 3

map addToThree [1, 8, 17] -- [4, 11, 20]
-- (addToThree 1) : (map addToThree [4, 5])
-- 4 : (addToThree 8) : (map addToThree [5])
-- 4 : 11 : (addToThree 5) : (map addToThree [])
-- 4 : 11 : 20 : []
-- 4 : 11 : [20]
-- 4 : [11, 20]
-- [4, 11, 20]
{% endhighlight %}

`map` takes a function and a list, and applies the function to every element of the list. `_` is just ignoring the value of an argument. If we've reached the end of the list (the empty list `[]`), there's no need to bother with the value of the function. `_` is not strictly necessary, but it adds to the readability of the pattern. `(x:xs)` is the pattern matching notation of separating the head of the list (`x`) and the rest of the list (`xs`). So we see that in the case of the empty list, when there are no more elements to apply the function to, we return the empty list. Otherwise, we apply the function to the head of the list, and `:` it onto the result of mapping the tail of the list.

{% highlight haskell %}
filter :: (a -> Bool) -> [a] -> [a]
filter _ [] = []
filter valid (x:xs) 
  | valid x = x : filter valid xs
  | otherwise = filter valid xs

gtThree x = x > 3

filter gtThree [10, 2, 5] -- [10, 5]
{% endhighlight %}

`filter` is similar to `map`, except that it takes a function that converts an `a` into a `Bool`, and keeps a value in the list if `valid x` returns `True`. Otherwise it skips over the element. 

There are more list functions, including the big daddy `foldr`, but researching and implementing them is *left as an exercise for the reader* &trade;. Let's move onto types. There are three ways to create new types.

{% highlight haskell %}
type Email = String
type Name = String
type BigFunction = (c -> d) -> (b -> c) -> (a -> b) -> (a -> d)

validForms :: Name -> Email -> Bool
-- validForms :: String -> String -> Bool
crazyFunction :: BigFunction -> BigFunction -> Bool
-- crazyFunction :: ((c -> d) -> (b -> c) -> (a -> b) -> (a -> d)) ->
--                  ((c -> d) -> (b -> c) -> (a -> b) -> (a -> d)) ->
--                  Bool
{% endhighlight %}

This is called "type synonyms". `type` followed by the name of your type (must begin with a capital letter), followed by `=`, and ended by an already defined type (such as `String` or a function). Type synonyms do nothing other than provide extra documentation for the type signature, and reduce the amount of typing needed if you have a long type. The type synonyms can be used interchangeably with the types the represent. If you think that's useless, just remember `type String = [Char]`.

{% highlight haskell %}
data Bool = True | False
data Maybe a = Nothing | Just a

not :: Bool -> Bool
not True  = False
not False = True

fromMaybe :: a -> Maybe a -> a
fromMaybe x Nothing  = x
fromMaybe _ (Just y) = y
{% endhighlight %}

Here we see Algebraic Data Types - we're creating brand new types. Everything after the `data` and before the `=` is going to be part of the type signature (`func :: type1 -> type2 -> ... -> returntype`). The parts after the `=` are called value constructors, and will be part of the function computation. The `|` works like an "or": a `Bool` can be either a `True` *or* a `False`. `Bool` is always part of the type signature, whereas during the computation, you work with `True` and `False`. Notice how you can both pattern match on the value constructors and use them during the computation (`not True = False`). 

For `Maybe`, `Maybe String`, `Maybe Int`, and `Maybe a` are all valid, but just `Maybe` is invalid. The `a` after the `Maybe` type in `data Maybe a = ...` is called a type parameter. Type parameters allow types to be polymorphic (i.e. `Maybe String` and `Maybe Int` are both supported). Any values after the value constructor name (`a` in `Just a`) are fields. The value constructors do not need to have the same amount of fields under it. `Nothing` has zero fields, while `Just` has one field of type `a`. Again, you can pattern match on value constructors and their fields.

{% highlight haskell %}
data Tree a = Empty | Node (Tree a) a (Tree a)

treeMap :: (a -> b) -> Tree a -> Tree b
treeMap _ Empty = Empty
treeMap f (Node l v r) = Node (treeMap f l) (f v) (treeMap f r)

data Tree a = Empty | Node { left :: Tree a, value :: a, right :: Tree a}

-- left  :: Tree a -> Tree a
-- value :: Tree a -> a
-- right :: Tree a -> Tree a 
{% endhighlight %}  

Algebraic Data Types can be self referencing. `Node` has three fields, two of which are `Tree a`s. The trees will be built up with `Node`s, and `Empty`s will be at the end of every tree. In `treeMap`, we check if we're at the end of a branch (`Empty`) and return an `Empty` if so. Otherwise we pattern match on the `Node` to separate the subtrees and the value, apply the function on the value, apply the function recursively on the subtrees, and join them together with a `Node`. 

The second definition of `Tree` is the same, except "record syntax" is used. Record syntax is an easy way to access fields. Instead of pattern matching to get the left subtree, you can just call the automatically generated function `left` on a `Node`: `left tree = leftSubtree`. Record syntax is a great convenience that makes it easier to work with value constructors.

{% highlight haskell %}
newtype State s a = State { runState :: s -> (s, a) }
{% endhighlight %}

`newtype` is used to generate a thin wrapper over another type. It can only have one value contructor, which can only have one field. There are other [subtleties][], but I'll skip over them for the sake of brevity `newtype` is extremely useful when dealing with typeclasses. What is a typeclass?

[subtleties]: http://www.haskell.org/haskellwiki/Newtype#The_messy_bits "Subtle"

It's similar to an interface: a set of behaviors that a type must follow. For example, if a type is part of the `Eq` typeclass, which deals with equality, it must implement either `==` or `/=`:

{% highlight haskell %}
data Rectangle = Rect Int Int

instance Eq Rectangle where
  (Rect w1 h1) == (Rect w2 h2) = (w1 == w2) && (h1 == h2)
{% endhighlight %}

Here we define a new data type called `Rect`, and make `Rect` an instance of `Eq` by comparing the two fields of different `Rect`s. This is quite a simplification, and typeclasses can get a little complex, but the gist is: typeclasses define a set of rules that other types must follow to be an instance of the typeclass. We can use typeclasses in type signatures if specific behaviors are needed. `func :: (Eq a) => a -> b`. Here, the type for `a` must be an instance of `Eq` for the program to compile.


That's it for basic Haskell. This tutorial probably went a bit fast, so if you don't quite understand something, you might have to look it up. Good resources include [Learn You a Haskell][], [Real World Haskell][], and [Hoogle][]. Hoogle allows you to search using either the function name (`map`) or its type signature (`(a -> b) -> [a] -> [b]`). Hopefully now you can skim some Haskell code and understand it on a rudimentary level.

[Learn You a Haskell]: http://learnyouahaskell.com/
[Real World Haskell]: http://book.realworldhaskell.org/
[Hoogle]: http://www.haskell.org/hoogle/

*Thanks to /u/dave4420, /u/ingolemo, and /u/azeeem for correcting errors!*
