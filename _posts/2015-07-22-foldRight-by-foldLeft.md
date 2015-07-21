---
comments: True
---

I've been reading the book [Functional Programming in Scala](http://manning.com/bjarnason) for the last a few days. The first mind twisting exercise in this book is to implement `foldRight` in terms of `foldLeft`. I couldn't wrap my head around this even after I looked at the [answer](https://github.com/fpinscala/fpinscala/blob/master/answerkey/datastructures/13.answer.scala). After staring at the solution really hard for a really long time and picturing it in my head, I finally got it. Using a little metaphor and analogy helped me understand how this works. I'm just going to assume the readers of this post know what a [fold](https://en.wikipedia.org/wiki/Fold_(higher-order_function)) operation is, but for the purpose of reference, I'll just copy the implementations of `foldRight` and `foldLeft` from the book.

```scala

sealed trait List[+A] // `List` data type, parameterized on a type, `A`
case object Nil extends List[Nothing] // A `List` data constructor representing the empty list
/* Another data constructor, representing nonempty lists. Note that `tail` is another `List[A]`,
which may be `Nil` or another `Cons`.
 */
case class Cons[+A](head: A, tail: List[A]) extends List[A]

def foldRight[A,B](as: List[A], z: B)(f: (A, B) => B): B =
as match {
  case Nil => z
  case Cons(x, xs) => f(x, foldRight(xs, z)(f))
}

@annotation.tailrec
def foldLeft[A,B](l: List[A], z: B)(f: (B, A) => B): B = l match {
    case Nil => z
    case Cons(h,t) => foldLeft(t, f(z,h))(f)
}

def foldRightViaFoldLeft_1[A,B](l: List[A], z: B)(f: (A,B) => B): B =
    foldLeft(l, (b:B) => b)((g,a) => b => g(f(a,b)))(z) // This is where I got lost.
```

How the heck did this code do the job? I couldn't find a damn clue when I first got my eyes on it. One of the reasons why it is hard to understand this code for me was that the short names of variable didn't help me a single bit to comprehend. But clearly, this kind naming convention is quite common in functional programming, according to what the authors say in the book.

>Variable-naming conventions

>It's a common convention to use names like f, g, and h for parameters to a higher-order function. In functional programming, we tend to use very short variable names, even one-letter names. This is usually because HOFs are so general that they have no opinion on what the argument should actually do. All they know about the argument is its type. Many functional programmers feel that short names make code easier to read, since it makes the structure of the code easier to see at a glance.

The other reason is that I am an 'intelligence below average level Joe'. Anyway, I find it useful to just materialize the abstract `f`, `g` and `h` into some real world example. 
A `foldLeft` operation is processing the list from left to right and `foldRight` is the other way around, right? So, during the `foldLeft` procedure, how do we process the rest of the list before we handle the current element? The answer is: WE DON'T. Instead of trying to calculate the partial result with the current element and accumulator while traversing the list from left to right, we build up a pipeline about 'how to process the list from right to left'. Sounds unclear? I thought so too.

Let's imagine there is a company that is dedicated to processing lists from right to left, but is required to not blow up the stack when dealing with long lists(hence, using `foldLeft`). When a customer comes to the company to use the service provided by the company(by calling `foldRightViaFoldLeft_1`), the CEO calls out her deputy and said to him: "You, arrange to process the list, and give me the result". When the CEO gets the result from her deputy, she'll just handle the result to the customer. All she has to do is to pass the result around. Expressed in scala, it's like this: `val ceo = (result :B) => result`. When the deputy is done, he'll tell the CEO like this `ceo(result)`.

When the deputy is assigned to process the list, he's like:"I have subordinates too, I'll let one of them process the tail of the list and report to me later. All I have to do is calculate the result from the head of the list and the partial result done by my subordinate and report the result to my boss".

```scala
val command_chain = foldLeft(l, ceo) {
  (my_direct_boss, element) => partial_result => my_direct_boss(f(element, partial_result))
}
```

We set up a command chain by using `foldLeft` like this. But notice, by this point, we only built up a pipeline about how to process the list from right to left, by chaining many similar small ~~functions~~ workers, each of which only knows how to process one element of this list when it receives a partial result of the rest of the list and who to report. When we fire off the command chain by `command_chain(z)`, the guy with the lowest pay  grade, Bob, will combine the last element and the seed value `z` and report to his team leader. Then the team leader combines the second to last element and the result given by Bob and report to the manager. After passing up the partial results up many times, the CEO finally gets the result and simply gives it to the customer( the caller of the function).
