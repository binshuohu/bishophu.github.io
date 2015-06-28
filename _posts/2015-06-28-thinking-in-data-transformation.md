---
---

A couple of days ago, I came across [this problem][Contains Duplicate III] on [LeetCode] [LeetCode]. Here's the description of it:

> Given an array of integers, find out whether there are two distinct indices i and j in the array such that the difference between nums[i] and nums[j] is at most t and the difference between i and j is at most k.

I tried to solve it in the old imperitive way, thinking about how to maintain the intermidiate result while traversing the array, corner cases and all those details. My code got dirtier and dirtier as I put in all those boundary checking logic. Of course I didn't complete this programming puzzle. I lost myself in the details and forgot the big picture.

I've read about something called *Wholemeal Programming* when I was reading this marverlous [Haskell Tutorial][Wholemeal Programming] today(Well, actually I've read this tutorial a couple of month ago but I don't have very good memory, OK?).

So I decided to make a second attemp on this problem, but with a different approch this time, in a functional way, to be precise. :)

Without worrying about the comlexity analysis, the first thing I did was trying to simplify the question. Instead of thinking in term of indices and individual elements, I refrase the question like this:

> Given an array of integers and a sliding window of size k on it, find out whether there are any two integers whose difference is lessEqual to t in the window as it slide through the array.

Suppose we already have all the windows of size k on the array, all we need to do is simply decide whether there is any window that contains two numbers `a` and `b` such that `abs(a - b) <= t`.

Let's define a function for that. I'm going to use Scala for all the code snippet we need to solve this problem. It's a concise and beautiful language I am currently learning and not as daunting as Haskell.
A window on the array is still an array, right? So let's just define the window as an array. I mean, List, of course.

{% highlight scala %}
type Window = List[Int]
{% endhighlight %}

The Signature of the function that decide whether any window contains duplicate would be:

{% highlight scala %}
def anyWindowContainsDuplicate(t: Int)(windows: List[Window]): Boolean = ???
{% endhighlight %}

Now, the question comes down to how to find out duplicate. Hmm, it seems that if the numbers in the window are sorted, all we need to do is find the smallest differnece of any two consecutive numbers. Let's just assume numbers in any window are sorted. So the previous function's definition would be like this:

{% highlight scala %}
def anyWindowContainsDuplicate(t: Int)(windows: List[Window]): Boolean = {
  windows exists (minimumAdjacentGap(_) <= t)
}

def minimumAdjacentGap(xs: Window): Int = xs match {
  case hd::Nil => 0
  case hd::tl => (tl, xs).zipped.map(_ - _).min
  case Nil => sys.error("can't happen")
}
{% endhighlight %}

`minimumAdjacentGap` is the only function that contains logic for corner cases.  The reason for special processing logic for single element Window is that if the window size `k == 1`, then `i` and `j` in the original problem must be equal, thus the difference being `0`.  

OK, we got the first part done(or the last part, because we are taking a top-down approach), let's move on. How do we acquire a list of windows each of which is a sorted list itself from the original array given in the problem? Well, it's almost a no brainer in Scala. Just use the `List.sliding(size: Int)` method to get a list of sliding windows and sort the numbers in each.

{% highlight scala %}
def makeWindows(k: Int)(nums: List[Int]): List[Window] = {
  nums.sliding(k).toList
}

def orderEachWindow(groups: List[Window]): List[Window] = groups map (_.sorted)
{% endhighlight %}

With all necessary parts ready to go, all that's left is to assemble them for good.

{% highlight scala %}
def containsDuplicate(nums: List[Int], k: Int, t: Int): Boolean = {
  (makeWindows(k) _
    andThen orderEachWindow
    andThen anyWindowContainsDuplicate(t))(nums)
}
{% endhighlight %}

Alas, if the eta expasion could be elimanated and if scala provide the 'pipe' operator as F# and Elixir do, the code above would be more readable. But anyway, it's still clear enough to explain itself.

To recap, the whole process of this solution is merely a series of data transformation.

> array => transform to windows => sort each window => examine each window => the final result

This solution isn't optimal in terms of space and time complexity, but having break down the problem into several small part, we can optimize accodingly. The Functional way of solving problem is just the old 'divide and conquer' method, but without putting the programmer in the tar pit of nasty low level details. 

[Contains Duplicate III]: https://leetcode.com/problems/contains-duplicate-iii/
[LeetCode]: https://leetcode.com/
[Wholemeal Programming]: https://www.fpcomplete.com/user/byorgey/introduction-to-haskell/1-haskell-basics#wholemeal-programming
