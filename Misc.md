# Misc Information

## Example Namespace
```clojure
(ns coarse.docs
  (:require [coarse.core :refer 
             [& *= *- *= *> += -= <%- <<%- =% _1 _2 _3 _4 _5 _all _butlast _dropping _filtering _first _index _last _pred _ranging _rest _taking attrs bind chain div= each filtered has? hcomp ix ix-default join lens magnify nx over preview quot= run-state sett to to-list-of view views]]))
```

## Implementation, Terminologies

While the implementation is somewhat faithful to the original,
a combination of Clojure's differences to Haskell and the
fact that Clojure is a dynamic language has made my implementation rather hacky,
and in retrospect, in need of a rewrite.

The ```ix``` lens should in fact be called a traversal, but I feel
that due to Clojure's treatment of ```nil``` as a default value
upon failure, it can be called a lens. I am not sure about this,
and I have not checked the lens laws on the lens I implemented.

In the end, I have no idea if the lenses and traversals
are degenrate or not (and I really should know).

## Roadmap

 - Cleanup the code, especially all the pseudotypeclass code smell
 - add transient support?
 - add more key-value data support
 - detect more bugs
 - test for lens laws

## All Functions under coarse.core

TODO