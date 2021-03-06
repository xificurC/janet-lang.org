{%
(def title "Looping")
(def description "Looping is a common and essential operation in programming. Most
languages support looping of some kind, either with explicit loops or recursion.
Janet supports both recursion and a primitive `while` loop. While recursion is
useful in many cases, sometimes is more convenient to use a explicit loop to
iterate over a collection like an array.")
(def prev-page {
 :link "/buffers.html"
 :text "Buffers"})
(def next-page {
 :link "/macros.html"
 :text "Macros"})
%}

A very common and essential operation in all programming is looping. Most
languages support looping of some kind, either with explicit loops or recursion.
Janet supports both recursion and a primitive `while` loop. While recursion is
useful in many cases, sometimes is more convenient to use a explicit loop to
iterate over a collection like an array.

## An Example - Iterating a Range

Suppose you want to calculate the sum of the first 10 natural numbers
0 through 9. There are many ways to carry out this explicit calculation
even with taking shortcuts. A succinct way in janet is

```janet
(+ ;(range 10))
```

We will limit ourselves however to using explicit looping and no functions
like `(range n)` which generate a list of natural numbers for us.

For our first version, we will use only the while macro to iterate, similar
to how one might sum natural numbers in a language such as C.

```janet
(var sum 0)
(var i 0)
(while (< i 10)
    (+= sum i)
    (++ i))
(print sum) # prints 45
```

This is a very imperative style program which can grow very large very quickly.
We are manually updating a counter `i` in a loop. Using the macros `+=` and `++`, this
style code is similar in density to C code.
It is recommended to use either macros (such as the loop macro) or a functional
style in janet.

Since this is such a common pattern, Janet has a macro for this exact purpose. The
`(for x start end body)` captures exactly this behavior of incrementing a counter
in a loop.

```janet
(var sum 0)
(for i 0 10 (+= sum i))
(print sum) # prints 45
```

We have completely wrapped the imperative counter in a macro. The for macro, while not
very flexible, is very terse and covers a common case of iteration, iterating over an integer range. The for macro will be expanded to something very similar to our original
version with a while loop.

We can do something similar with the more flexible `loop` macro.

```janet
(var sum 0)
(loop [i :range [0 10]] (+= sum i))
(print sum) # prints 45
```

This is slightly more verbose than the for macro, but can be more easily extended.
Let's say that we wanted to only count even numbers towards the sum. We can do this
easily with the loop macro.

```janet
(var sum 0)
(loop [i :range [0 10] :when (even? i)] (+= sum i))
(print sum) # prints 20
```

The loop macro has several verbs (:range) and modifiers (:when) that let
the programmer more easily generate common looping idioms. The loop macro
is similar to the Common Lips loop macro, but smaller in scope and with a much
simpler syntax. As with the `for` macro, the loop macro expands to similar
code as our original while expression.

## Another Example - Iterating an Indexed Data Structure

Another common usage for iteration in any language is iterating over the items in
some data structure, like items in an array, characters in a string, or key value
pairs in a table.

Say we have an array of names that we want to print out. We will
again start with a simple while loop which we will refine into
more idiomatic expressions.

First, we will define our array of names

```janet
(def names
 @["Jean-Paul Sartre"
   "Bob Dylan"
   "Augusta Ada King"
   "Frida Kahlo"
   "Harriet Tubman"])
```

With our array of names, we can use a while loop to iterate through the indices of names, get the
values, and the print them.

```janet
(var i 0)
(def len (length names))
(while (< i len)
    (print (get names i))
    (++ i))
```

This is rather verbose. janet provides the `each` macro for iterating through the items in a tuple or
array, or the bytes in a buffer, symbol, or string.

```janet
(each name names (print name))
```

We can also use the `loop` macro for this case as well using the `:in` verb.

```janet
(loop [name :in names] (print name))
```

## Iterating a Dictionary

In the previous example, we iterated over the values in an array. Another common
use of looping in a Janet program is iterating over the keys or values in a table.
We cannot use the same method as iterating over an array because a table or struct does
not contain a known integer range of keys. Instead we rely on a function `next`, which allows
us to visit each of the keys in a struct or table. Note that iterating over a table will not
visit the prototype table.

As an example, lets iterate over a table of letters to a word that starts with that letter. We
will print out the words to our simple children's book.

```janet
(def alphabook
  @{"A" "Apple"
    "B" "Banana"
    "C" "Cat"
    "D" "Dog"
    "E" "Elephant"})
```

As before, we can evaluate this loop using only a while loop and the `next` function.

```janet
(var key (next alphabook nil))
(while (not= nil key)
  (print key " is for " (get alphabook key))
  (set key (next alphabook key)))
```

However, we can do better than this with the loop macro using the `:pairs` or `:keys` verbs.

```janet
(loop [[letter word] :pairs alphabook]
  (print letter " is for " word))
```

Using the `:keys` verb and shorthand notation for indexing
a data structure.

```janet
(loop [letter :keys alphabook]
  (print letter " is for " (alphabook letter)))
```

Data structures like tables and structs can be called like functions that look up their
arguments. This allows for writing shorter code than what
is possible with `(get alphabook letter)`.

We can also use the core library functions `keys` and `pairs` to get arrays of the keys and
pairs respectively of the alphabook.

```janet
(loop [[letter word] :in (pairs alphabook)]
  (print letter " is for " word))

(loop [letter :in (keys alphabook)]
  (print letter " is for " (alphabook letter)))
```

Notice that iteration through the table is in no particular order. Iterating
the keys of a table or struct guarantees no order. If you want to
iterate in a specific order, use a different data structure or
the `(sort indexed)` function.
