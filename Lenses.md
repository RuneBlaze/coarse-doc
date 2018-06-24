# Common Lenses and Traversals

<pre class="hidden"><code
    class="lang-eval-clojure"
    data-gist-id="RuneBlaze/12dc8fa157094483d4c07100df56f82d">
</code></pre>

<pre class="hidden"><code
    class="lang-eval-clojure"
    data-gist-id="RuneBlaze/345bb25a7bc3525472a3f414d93208b7">
</code></pre>

<pre class="hidden"><code
    class="lang-eval-clojure">
(ns coarse.docs
  (:require [coarse.core :refer 
             [& *= *- *= *> += -= <%- <<%- =% _1 _2 _3 _4 _5 _all _butlast _dropping _filtering _first _index _last _pred _ranging _rest _taking attrs bind chain div= each eval-state exec-state filtered has? hcomp hcomp2 ix ix-default join lens lens-do magnify nx over preview quot= run-state sett state state-get state-gets state-modify state-put state-return state-view to to-list-of view views wrap-lens zoom ]]))
</code></pre>


## Lenses

Lenses are "functional pointers", getters and setters combined accessing some larger structure.
Lenses can be converted into getters using the ```view``` function, as setters ```sett```, as
"updaters" using ```over```.

 vanilla Clojure | lens
 --- | ---
 ```(nth [0 1] 0)``` | ```(view (nx 0) [0 1])```
 ```(assoc [0 1] 0 99)``` | ```(sett (nx 0) 99 [0 1])```
 ```(update [0 1] 0 inc)``` | ```(over (nx 0) inc [0 1])```

While Clojure's vanilla functions are succinct and sufficient
in simple cases, lens are
much more powerful and composable as the data complexity grows.

### ix k

```ix k``` uses ```k``` as a key to retrieve and
update any indexed data.

```eval-clojure
[
    (over (ix :a) inc {:a 100 :b 99})
    (sett (ix 0) 42 [-99 100])
]
```

### _1 _2 _3 _4 _5

```_1``` is simply ```(ix 0)```,
```_2``` simply ```(ix 1)```, and so on, providing
shorthand for accessing common elements.

### nx k

```eval-clojure
[
    (sett (nx 0) 42 [-99 100])
]
```

```ix``` variant, where the key is an integer, and
uses ```nth``` under the hood. This lens throws error upon
accessing out of index elements.

### *>

Function composition, Haskell style, used in composing lenses.

One of the great properties of lenses is that they compose
just under the vanilla function composition, enabling Haskell
programmers to compose lenses like ```player.pos.x```.

In Haskell, function composition, such as ```(f . g) a b```
is evaluated to be ```(f g a) b```, where Clojure's
```((comp f g) a b)``` is ```(f (g a b))```, rendering
it unusable for lens composition.

```*>``` mimics Haskell's function composition, so that
```((*> f g) a b)``` is ```((f g a) b)```, usable
for lens composition.

```eval-clojure
(over (*> (ix 0) (ix 1)) inc [[0 1] [2 3]])
```

If you pass integers and keywords to ```*>```,
they will automatically be converted into lens using ```ix```.

```eval-clojure
(over (*> 0 1 :foo) inc [[0 {:foo 42}] [2 3]])
```

```*>``` somewhat resembles vanilla Clojure's sequence of keys
used in ```update-in``` and ```assoc-in```.

```eval-clojure
(->> {:pos [0 0]}
  (over (*> :pos 0) inc)
  (over (*> :pos 1) dec))
```

```*>``` is also called ```hcomp```.


### lens

```lens``` constructs a lens from a getter and setter. The result
might not be the most efficient, but it is certainlly the
most familiar method to construct lenses.

```eval-clojure
(let [l (lens first
              (fn [coll e] (assoc coll 0 e)))]
    [
        (view l [99 98 97 96 95])
        (over l inc [99 98 97 96 95])
    ])
```

### to

```to``` converts a getter into a "lens" that can only get.

```eval-clojure
(view (to first) [3 5 7 9])
```

### join l1 l2 ...

Constructs a lens, with the view of the original lenses combined into a vector.

```eval-clojure
(def _13 (join (nx 0) (nx 2)))

(let [v [:a :b :c :d :e :f]]
    [
      (view _13 v)
      (over _13 (fn [[x y]] [y x]) v)
      (sett _13 [42 99] v)
    ])
```

You could easily construct degenerate lenses if the original
views of the lenses overlap.


### Maths Operators

```eval-clojure
(->> {:pos [0 0]}
  (+= (*> :pos 0) 1)
  (-= (*> :pos 1) 2))
```

Shorthands, ```(+= l n s)``` is ```(over l (partial + n) s)```.

### Auto Lifting

```eval-clojure
[
    (view 0 [1 2 3 4 5])
    (over 0 inc [1 2 3 4 5])
    (sett :hello :world {:hello :earth})
]
```

```view```, ```sett```, and ```over``` (and some other functions)
implicitly converts keywords and integers to lenses using ```ix```.

## Traversals

Traversals are like lens, but access multiple values.
Thus they cannot be simply be converted to getters using ```view```.
```to-list-of``` will retrieve all the values.

```over``` and ```sett``` can still operate on traversals.

### each

Access all values in a collection.

```eval-clojure
[
    (sett each 100 [[:c :d] [:b :c] [:a] [:m]])
    (to-list-of each [[:c :d] [:b :c] [:a] [:m]])
    (sett (*> each each) 100 [[:c :d] [:b :c] [:a] [:m]])
    (over (*> each each) inc [[1 2 3] [4 5 6] [7 8 9 10] []])
    (to-list-of (*> each each) [[1 2 3] [4 5 6] [7 8 9 10] []])
]
```

### _ranging, _taking, _dropping

These traversals access all elements,
but only operates on values whose indexes
match their predicates.

```eval-clojure
[
    (range 0 30 4)
    (to-list-of (_ranging (range 3 5)) (range 0 30 4))
    (to-list-of (_taking 5) (range 0 30 4))
    (over (_dropping 5) dec (range 0 30 4))
]
```

### _index

Generalized range based traversal, takes a
custom predicate that traverses all elements, and only operates on indexes
that satisfies the predicate.

```eval-clojure
[
    (sett (_index even?) 42 (range 0 5))
    (sett (_index odd?) 42 (range 0 5))
]
```

### _filtering, filtered pred

```_filtering``` accesses all elements,
and only operates on values that
satisfy the predicate.

```eval-clojure
(to-list-of (_filtering even?) [1 1 2 3 5 8])
```

```filtered``` only operates on values that
satisfy the predicate.

```eval-clojure
[
    (to-list-of (*> each (filtered even?)) [1 1 2 3 5 8])
    (to-list-of (filtered even?) 2)
]
```

## "state" macros, pseudo-imperative constructs

Haskell's lens library provides operators
and constructs dealing with the state monad.

State monad can be unidiomatic in Clojure
and the need for the state monad is to some extent
mitigated via the threading macros, ```->``` and ```->>```,
which enables us writing code remniscient of imperative programming.

```eval-clojure
(->> {:pos [0 0]}
     (+= (*> :pos 0) 1)
     (-= (*> :pos 1) 2))
```

This library provides additional macros
to facilitate this pseudo-imperative style.

### +>>, the zooming operator

```+>>``` "zooms" into a substructure using a lens,
remniscient of the original library's ```zoom```
function.

```clojure
(def game-state
  {:player {:pos [0 0]}})

(->> game-state
     (+>> (*> :player :pos)
        (+= (nx 0) 1)
        (-= (nx 1) 2)))
; {:player {:pos [1 -2]}}
```

### using

```using``` is a variant of ```let``` that
binds results retrieved via lenses
to local variables.

```clojure
; swaps the two elements of the position
(->> {:player {:pos [-3 7]}}
     (+>> (*> :player :pos)
        (using [x _1
                y _2]
            (=% _1 y)
            (=% _2 x))))
; {:player {:pos [7 -3]}}
```