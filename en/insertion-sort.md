## Insertion Sort

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

Why sorting?
Many computer scientists consider sorting to be the most fundamental problem in
the study of algorithms. There are several reasons:
 Sometimes an application inherently needs to sort information. For example,
in order to prepare customer statements, banks need to sort checks by check
number.
 Algorithms often use sorting as a key subroutine. For example, a program that
renders graphical objects which are layered on top of each other might have
to sort the objects according to an “above” relation so that it can draw these
objects from bottom to top. We shall see numerous algorithms in this text that
use sorting as a subroutine.
 We can draw from among a wide variety of sorting algorithms, and they employ
a rich set of techniques. In fact, many important techniques used throughout
algorithm design appear in the body of sorting algorithms that have been
developed over the years. In this way, sorting is also a problem of historical
interest.
 We can prove a nontrivial lower bound for sorting (as we shall do in Chapter 8).
Our best upper bounds match the lower bound asymptotically, and so we know
that our sorting algorithms are asymptotically optimal. Moreover, we can use
the lower bound for sorting to prove lower bounds for certain other problems.

```java
public void sort(int[] arr) {
        for (int j = 1; j < arr.length; j++) {
            // Find a key
            int key = arr[j];
            int i = j - 1;
            while (i >= 0 && arr[i] > key) {
                // Find the right location that insertion current key into the array
                arr[i + 1] = arr[i];
                i--;
            }
            // Here is the location
            arr[i + 1] = key;
        }
    }
```
