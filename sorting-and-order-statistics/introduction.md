## Introduction

This part presents several algorithms that solve the following sorting problem:

Input: A sequence of n numbers [a1, a2, ..., an].  
Output: A permutation (reordering) [a'1, a'2, ..., a'n] of the input sequence such
that a'1 <= a'2 <= ... <= a'n.  
The input sequence is usually an n-element array, although it may be represented
in some other fashion, such as a linked list.

### The structure of the data

In practice, the numbers to be sorted are rarely isolated values. Each is usually part
of a collection of data called a record. Each record contains a key, which is the
value to be sorted. The remainder of the record consists of satellite data, which are
usually carried around with the key. In practice, when a sorting algorithm permutes
the keys, it must permute the satellite data as well. If each record includes a large
amount of satellite data, we often permute an array of pointers to the records rather
than the records themselves in order to minimize data movement.

In a sense, it is these implementation details that distinguish an algorithm from
a full-blown program. A sorting algorithm describes the method by which we
determine the sorted order, regardless of whether we are sorting individual numbers
or large records containing many bytes of satellite data. Thus, when focusing on the
problem of sorting, we typically assume that the input consists only of numbers.
Translating an algorithm for sorting numbers into a program for sorting records
is conceptually straightforward, although in a given engineering situation other
subtleties may make the actual programming task a challenge.

### Why sorting?

Many computer scientists consider sorting to be the most fundamental problem in
the study of algorithms. There are several reasons:

* Sometimes an application inherently needs to sort information. For example,
  in order to prepare customer statements, banks need to sort checks by check
  number.

* Algorithms often use sorting as a key subroutine. For example, a program that
  renders graphical objects which are layered on top of each other might have
  to sort the objects according to an "above" relation so that it can draw these
  objects from bottom to top. We shall see numerous algorithms in this text that
  use sorting as a subroutine.

* We can draw from among a wide variety of sorting algorithms, and they employ
  a rich set of techniques. In fact, many important techniques used throughout
  algorithm design appear in the body of sorting algorithms that have been
  developed over the years. In this way, sorting is also a problem of historical
  interest.

* We can prove a nontrivial lower bound for sorting. Our best upper bounds match
  the lower bound asymptotically, and so we know that our sorting algorithms are
  asymptotically optimal. Moreover, we can use the lower bound for sorting to prove
  lower bounds for certain other problems.

* Many engineering issues come to the fore when implementing sorting algorithms.
  The fastest sorting program for a particular situation may depend on
  many factors, such as prior knowledge about the keys and satellite data, the
  memory hierarchy (caches and virtual memory) of the host computer, and the
  software environment. Many of these issues are best dealt with at the algorithmic
  level, rather than by "tweaking" the code.

### Sorting algorithms

Insertion sort takes Θ(n^2) time in the worst case. Because its inner loops are tight,
however, it is a fast in-place sorting algorithm for small input sizes. (Recall that
a sorting algorithm sorts in place if only a constant number of elements of the input
array are ever stored outside the array.) Merge sort has a better asymptotic running
time, Θ(n*lgn), but the merge procedure it uses does not operate in place.

Heap sort sorts n numbers in place in O(n*lgn) time. It uses an important data structure,
called a heap, with which we can also implement a priority queue.

Quick sort also sorts n numbers in place, but its worst-case running time is Θ(n^2).
Its expected running time is Θ(n*lgn), however, and it generally outperforms heapsort
in practice. Like insertion sort, quick sort has tight code, and so the hidden constant
factor in its running time is small. It is a popular algorithm for sorting large input arrays.

Insertion sort, merge sort, heap sort, and quick sort are all comparison sorts: they
determine the sorted order of an input array by comparing elements. By introducing
the decision-tree model in order to study the performance limitations of comparison
sorts. Using this model, we prove a lower bound of Ω(n*lgn) on the worst-case running
time of any comparison sort on n inputs, thus showing that heap sort and merge sort
are asymptotically optimal comparison sorts.

We then goes on to show that we can beat this lower bound of Ω(n*lgn)
if we can gather information about the sorted order of the input by means other
than comparing elements. The counting sort algorithm, for example, assumes that
the input numbers are in the set {0, 1, ..., k}. By using array indexing as a tool
for determining relative order, counting sort can sort n numbers in Θ(k + n) time.
Thus, when k = O(n), counting sort runs in time that is linear in the size of the
input array. A related algorithm, radix sort, can be used to extend the range of
counting sort. If there are n integers to sort, each integer has d digits, and each
digit can take on up to k possible values, then radix sort can sort the numbers
in Θ(d(n + k)) time. When d is a constant and k is O(n), radix sort runs in
linear time. A third algorithm, bucket sort, requires knowledge of the probabilistic
distribution of numbers in the input array. It can sort n real numbers uniformly
distributed in the half-open interval [0, 1) in average-case O(n) time.

The following table summarizes the running times of the sorting algorithms. As usual,
n denotes the number of items to sort. For counting sort, the items to sort are
integers in the set {0, 1, ..., k}. For radix sort, each item is a d-digit number,
where each digit takes on k possible values. For bucket sort, we assume that the keys
are real numbers uniformly distributed in the half-open interval [0, 1). The rightmost
column gives the average-case or expected running time, indicating which it gives
when it differs from the worst-case running time. We omit the average-case running
time of heap sort because we do not analyze it.

| Algorithm  | Worst-case running time  | Average-case/expected running time  |
|---|---|---|
| Insertion sort  | O(n^2)  |  Θ(n^2) |
|  Merge sort |  O(n*lgn) |  Θ(n*lgn) |
| Heap sort  |  O(n*lgn) |  - |
| Quick sort  |  O(n^2) | Θ(n*lgn) (expected)  |
| Counting sort  | O(k + n)  | Θ(k + n)  |
| Radix sort  |  O(d(n + k)) |  Θ(d(n + k)) |
| Bucket sort  | O(n^2)  | Θ(n) (average-case) |

#### Order statistics

The ith order statistic of a set of n numbers is the ith smallest number in the set.
We can, of course, select the ith order statistic by sorting the input and indexing
the ith element of the output. With no assumptions about the input distribution,
this method runs in Ω(n*lgn) time, as the lower bound proved in the decision-tree model shows.
In [order-statistics](order-statistics.md), we show that we can find the ith smallest element in O(n) time,
even when the elements are arbitrary real numbers. We present a randomized algorithm
with tight code that runs in Θ(n^2) time in the worst case, but whose
expected running time is O(n^2). We also give a more complicated algorithm that
runs in O(n) worst-case time.
