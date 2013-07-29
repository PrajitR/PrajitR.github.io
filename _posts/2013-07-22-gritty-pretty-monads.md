---
layout: post
title: Gritty Pretty Monads
tags:
- haskell
- monads
- tutorial
url: /gritty-pretty-monads
summary: A[nother] monad tutorial full of examples and details. Instead of analogies such as burrito or spacesuit, we dig down. 
---
Maecenas vel aliquam ipsum, eu mollis velit. Etiam sagittis arcu et fermentum rhoncus. Praesent urna dui, scelerisque sit amet magna a, interdum suscipit enim. Curabitur suscipit tempor consectetur. Suspendisse eu felis tortor. Mauris consequat tempus eros, eget porta mauris vulputate ut. Nullam vitae dui at ligula mollis mattis at ac eros. Donec aliquet non erat vitae suscipit. Cras rhoncus ligula in cursus aliquam. Vestibulum et mauris at est ultrices dictum. Maecenas suscipit varius eros, non feugiat nunc placerat lacinia.

Donec tincidunt arcu vehicula diam ultricies, et porta metus pretium. Nam eros nisi, viverra nec congue at, tincidunt at mauris. Ut consequat erat sit amet justo cursus vehicula. Interdum et malesuada fames ac ante ipsum primis in faucibus. Pellentesque habitant morbi tristique senectus et netus et malesuada fames ac turpis egestas. Cras imperdiet eleifend nulla, nec posuere enim bibendum eu. Aliquam non sollicitudin arcu. Cras condimentum dolor quis lectus consectetur, eget consequat risus venenatis.

Sed accumsan eu urna eu lacinia. Aliquam quis risus nec enim molestie fermentum. Mauris malesuada dapibus nisi, a fermentum nisl hendrerit non. Nullam fermentum, orci eget porta dictum, felis tellus elementum nisl, nec iaculis erat enim lobortis nisi. Donec augue tellus, sagittis vel mauris id, porttitor congue ante. Vestibulum dapibus accumsan felis ut tincidunt. Fusce in est libero. Pellentesque habitant morbi tristique senectus et netus et malesuada fames ac turpis egestas.

{% highlight haskell %}
map :: (a -> b) -> [a] -> [b]
map _ [] = []
map f (x:xs) = f x:map f xs

filter :: (a -> b) -> [a] -> [b]
filter _ [] = []
filter f (x:xs)
  | f x = x:filter f xs
  | otherwise = filter f xs
{% endhighlight %}

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Ut sodales metus quis massa sagittis, sed dapibus orci vulputate. Phasellus sollicitudin imperdiet leo consectetur iaculis. Fusce a dignissim massa, nec feugiat felis. Curabitur porta pharetra lacinia. Pellentesque bibendum massa nisi, at gravida odio dignissim vel. Maecenas ac sem libero. Sed eleifend pretium odio ut consectetur. In vitae tellus sit amet turpis porta lobortis. Quisque molestie venenatis mauris quis ultricies. Aenean vestibulum eros quis nisi varius consectetur. Aliquam tristique sed dolor sed gravida. Morbi tellus arcu, varius blandit erat in, blandit tincidunt libero. Proin nec lectus bibendum sem congue pharetra.

Sed cursus mauris suscipit velit tincidunt gravida. Sed aliquet, sem eu varius rutrum, ipsum justo malesuada ante, ut auctor orci urna in arcu. Fusce ut mauris in magna laoreet tempor at at lacus. Nam eu ullamcorper tellus. Mauris facilisis nisi vulputate mi interdum, sed molestie ipsum vulputate. Vestibulum scelerisque erat sed leo consectetur, non eleifend enim facilisis. Proin sit amet porta est. Morbi scelerisque tempor lectus, sed imperdiet sem. Ut purus eros, pulvinar ac adipiscing vitae, aliquam quis sapien. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae;
