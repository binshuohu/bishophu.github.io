---
comments: True
---

# Syntax
## Forms
Also known as *expression*. Clojure evaluate every form to produce a value. There are two kinds of forms
1. Literal representations of data structures such as numbers, strings, maps and vectors.
2. Operations

```clojure
;data
1
"ring"
["to" "rule" "them" "all"]

;what operations are like
(operator operand1 operand2 ... operandn)
```

## Control Flow
### if
```clojure
(if boolean-form
  then-form
  optional-else-form)
```

Else branch could be omitted. In that case, it's just like you put a `nil` in that branch.
### do
`do` operator could sequence multiple forms into a single form.

```clojure
(if true
  (do (println "Hello")
      (println "world"))
  (println "Leave me alone."))
```

### when
It's like a combination of `if` and `do` without the `else` branch.
```clojure
(when true
  "abc"
  "def")
```

## Naming Values with def
You can *bind* a name to a value using `def` in Clojure:
```clojure
(def my-favorite-movie "Twelve Angry Men")
```

# Data Structures
## Numbers
```clojure
42
42.0
1/42
```

## Strings
```clojure
"Matt Damon is a hell of an actor"
;single quoted string is illegal in clojure
```

## Maps
```clojure
{} ;empty map
{:me "so horny" :price "5 dollars"} ; :me and :price are keywords(like atoms in elixir)
{"shit" "happens"}
{:maps {"can" "be nested"
        :and "are heterogenous"}}

;how to create a map
(hash-map :a 1 :b 2)
; => {:a 1 :b 2}

;how to look up values in maps
(get {:a 1 :b 2} :b)
; => 2

(get {:a 1 :b 2} :c)
; => nil

;The get-in function lets you look up values in nested maps:
(get-in {:a 0 :b {:c "ho hum"}} [:b :c])
; => "ho hum"

;An alternative way to find values in a map is to treat the map itself as a function
({:name "The Human Coffeepot"} :name)
; => "The Human Coffeepot"
```

## Keywords
Keywords are primarily used as keys in maps. They can be used as function to look up values in maps
```
(:b {:a 1 :b 2})
; => 2
```

## Vectors
```clojure
[1 2 "go"]

;look up by index
(get [2 4 6] 0)
; => 2

(vector 1 2 3 "done")
; => [1 2 3 "done"]

use conj to append element to a vector
(conj [1 2] 3)
; => [1 2 3]
```

## List
```clojure
;this is list literal
'(1 2 3 "go")
; => (1 2 3 "go")

(nth '(:a :b :c) 0)
; => :a

(list 1 "two" {3 4})
; => (1 "two" {3 4})

(conj '(1 2 3) 4)
; => (4 1 2 3)
```

## Sets
There are two kinds of sets, namely hash sets and sorted sets. Now we only talk about hash sets because they are used more often.

```clojure
#{"me" "so" :horny}
(hash-set 1 2 3 2)
; => #{3 1 2} ;sets can deduplicate

;use conj to add element into sets
(conj #{1 3} 2)
; => # {2 1 3}

(set [3 3 3 4 4])
; => #{3 4}

(set '(3 3 3 4 4))
; => #{3 4}

(contains? #{:a :b} :a)
; => true

(contains? #{:a :b} 3)
; => false

(contains? #{nil} nil)
; => true

(:a #{:a :b})
; => :a

(:c #{:a :b})
; => nil

```

# Functions
```clojure
(+ 1 2 3 4) ; => 10
(* 1 2 3 4) ; => 24
(first [1 2 3 4]) ; => 1

;the operator could be a function or an expression that returns a fucntion
((if true + *) 1 2)
; => 3
```

In Clojure, functions are eager, which means all parameters are evaluated before being passed to the fucntion.

How to define a function:

```clojure
(defn is-horny?
  "tell whether a person is needy" ; a doc string that describes what this function does
  [first-name last-name] ;parameter list
  (println "Yes, you shameless slut")
  true)
  
(is-horny? "bishop" "hu")
; => Yes, you shameless slut
; => true
```

The parameter list is a pattern, so you can overload the function based on arity.

```clojure
(defn multi-arity
  ;; 3-arity arguments and body
  ([first-arg second-arg third-arg]
     (do-things first-arg second-arg third-arg))
  ;; 2-arity arguments and body
  ([first-arg second-arg]
     (do-things first-arg second-arg))
  ;; 1-arity arguments and body
  ([first-arg]
     (do-things first-arg)))
```
Clojure also allows functions to have variadic parameters. You can use a `&` to bind the rest parameters.

```clojure
(defn favorite-things
  [name & things]
  (str "Hi, " name ", here are my favorite things: "
       (clojure.string/join ", " things)))

(favorite-things "Doreen" "gum" "shoes" "kara-te")
; => "Hi, Doreen, here are my favorite things: gum, shoes, kara-te"
```

### destructuring (pattern matching?)
```clojure
;; Return the first element of a collection
(defn my-first
  [[first-thing]] ; Notice that first-thing is within a vector
  first-thing)

(my-first ["oven" "bike" "war-axe"])
; => "oven"

(defn chooser
  [[first-choice second-choice & unimportant-choices]]
  (println (str "Your first choice is: " first-choice))
  (println (str "Your second choice is: " second-choice))
  (println (str "We're ignoring the rest of your choices. "
                "Here they are in case you need to cry over them: "
                (clojure.string/join ", " unimportant-choices))))

(chooser ["Marmalade", "Handsome Jack", "Pigpen", "Aquaman"])
; => Your first choice is: Marmalade
; => Your second choice is: Handsome Jack
; => We're ignoring the rest of your choices. Here they are in case you need to cry over them: Pigpen, Aquaman
```

You can also destructure a map.

```clojure
(defn announce-treasure-location
   [{lat :lat lng :lng}]
   (println (str "Treasure lat: " lat))
   (println (str "Treasure lng: " lng)))

(announce-treasure-location {:lat 28.22 :lng 81.33})
; => Treasure lat: 100
; => Treasure lng: 50

;a more concise way to do this
(defn announce-treasure-location
  [{:keys [lat lng]}]
  (println (str "Treasure lat: " lat))
  (println (str "Treasure lng: " lng)))
  
;use `as` to bind the whole thing, like `@` in Scala
(defn receive-treasure-location
  [{:keys [lat lng] :as treasure-location}]
  (println (str "Treasure lat: " lat))
  (println (str "Treasure lng: " lng))

  ;; One would assume that this would put in new coordinates for your ship
  (steer-ship! treasure-location))
```

### anonymous functions (lambda expressions)
```clojure
(fn [param-list]
  function body)

((fn [& xs] xs) 1 2 3)
; => (1 2 3)
  
;of course there is a more compact way to do this
#(* % 3)

(#(* % 3) 8)
; => 24

(#(str %1 " and " %2) "cornbread" "butter beans")
; => "cornbread and butter beans"

;use %& to bind rest params
(#(identity %&) 1 "blarg" :yip)
; => (1 "blarg" :yip)
```
