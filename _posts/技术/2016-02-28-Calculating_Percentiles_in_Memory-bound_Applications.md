---
layout: post
title: Calculating Percentiles in Memory-bound Applications
category: 技术
tags: 算法
keywords: 
description: 

---

##Calculating Percentiles in Memory-bound Applications
###Introduction
Suppose you have a long list of numbers and you want to find the 5th and 95th percentiles. If the list is small enough, you can read the list into memory, sort it, and see which numbers are 5% of the way from each end. Simple. But what do you do if the list is being generated sequentially and the entire list is too large to fit into memory at once? This article presents a simple C++ class for solving this problem. The class takes template parameters so it can be used with any data type.

###Motivating Application
The solution presented here could be used in a variety of memory-bound problems, but I'll describe briefly the problem that originally motivated the code. As part of its processing, the statistical package WFMM generates a huge matrix one row at a time, then reports a couple percentiles of each column. It would be nice if it were possible to produce the data one column at a time, but that's not the case. The nature of the problem is that rows can be computed independently but columns cannot. The size of this matrix varies according to the input, and the amount of available memory determines the maximum problem size the software can handle. The code presented here was used in the WFMM project to conserve memory.

###Background
To make the problem tangible, suppose we have a 1000 numbers and we want to know what the 50th number would be if we were to sort the list in increasing order. Imagine we want to do this using as little memory as possible. We could read in the first 50 numbers and sort them. Then when we read in the 51st number, we compare it to the largest of the 50 numbers we've saved. If the new number is larger, we throw it away because we know it cannot be the 50th smallest number of the full list. If the new number is smaller, we throw away the largest number we were keeping and insert the new number into our cache. This gives us a way to compute the 50th smallest number using 50 memory locations. Obviously we could use a similar approach to find the 50th largest number as well.

Could we do any better? No. Until we see the last 50 numbers, we can't rule out the possibility of any one of the numbers we're saving being the one we're after. When we see the 951st number, we can rule out one of the numbers in our cache. When we see the 952nd number we can throw out another number from the cache, etc. But for the majority of the runtime of the algorithm, we need to hold on to 50 numbers.

In general, if you want to find the nth smallest number in a list of M numbers, you need to save n numbers along the way. (Unless n is larger than half of M. Then you could turn the problem around and find the (M-n)th largest number in the list.) So if you want to calculate the 5th or 95th percentile of a list of M numbers, you need to store 0.05 M numbers. If you want to find both the 5th and the 95th percentiles you'd need to store 0.10 M numbers along the way. By storing just the numbers you need, you can solve a problem 10 times larger than you could if you were to load everything into memory first and then sort.

###Using the Code
The class TailKeeper keeps track of the “tails” of a sequence of numbers, the largest and smallest values, in order to compute upper and lower percentiles as described above. Values are inserted into a hash data structure as they arrive. This allows us to keep the stored numbers sorted and make fast inserts without requiring additional memory.

The class uses templates to work with any data type that supports comparison, not just numeric types. Also, the input and storage types may differ. In our application, the input data had type double but was stored as type float. This allowed us to save twice as much data in the same memory. There was enough noise in the data that the loss of precision in casting double to float did not matter.

To use TailKeeper in your application, #include the files TailKeeper.h and FixedSizeHeap.h. The class needs two constants to specify which values you're looking for. If you want to find the mth smallest and nth largest values, you can specify m and n either as arguments to the constructor or as arguments to the Initialize function.

For example, suppose you're wanting to find the 40th smallest and the 10th largest element in a list of doubles and you are willing to cast your input to floats to save space. You could declare a TailKeeper class as follows:
```
TailKeeper<double, float> tk(40, 10);
```
The input type defaults to double and the storage type defaults to float and so <double, float> could be left out in this case.

As each value in the list becomes available, insert it into TailKeeper by calling the AddSample method:
```
tk.AddSample( x );
```
To find the 40th smallest element along the way, call GetMaxLeftTail(). Similarly, to find the 10th largest element, call GetMinRightTail(). (The smallest 40 elements seen at any point constitute the left tail, so the maximum of these is the 40th smallest. The largest 10 elements are the right tail, so the minimum of these is the 10th largest.)

###Testing the Code
The demo project first tests the FixedSizeHeap class that the TailKeeper depends on by checking and inserting several values and comparing the internal state of the heap to the results of manual calculations.

The project then creates a list of 1000 random integers and finds the 5th and 96th percentiles using TailKeeper directly by sorting the list.

```
///////////////////////////////////////////////////////////////////////
// Generate a list of random integers.
int length = 1000;
std::vector<int> data(length);
data[0] = 137; // arbitrary seed
for (int i = 1; i != length; ++i)
{
    // This is George Marsaglia's "CONG" random number generator.
    data[i] = 69069*data[i-1] + 1234567;
}

///////////////////////////////////////////////////////////////////////
// Find the 50th smallest and 40th largest elements using TailKeeper.
int lower = 50, upper = 40;
TailKeeper<int,><int, int> tk(lower, upper);
for (int i = 0; i != length; ++i)
    tk.AddSample( data[i] );
int leftTailTK = tk.GetMaxLeftTail();
int rightTailTK = tk.GetMinRightTail();

///////////////////////////////////////////////////////////////////////
// Find the same values directly by sorting the entire list.
std::sort(data.begin(), data.end());
int leftTailSort = data[lower-1];
int rightTailSort = data[length - upper];

///////////////////////////////////////////////////////////////////////
// Compare the results.
assert(leftTailTK == leftTailSort);
assert(rightTailTK == rightTailSort);
```
For further testing, you could vary the random number generator seed value data[0] or the values of the length, upper, and lower param

[原文地址][1]
[代码][2]
[1]:http://www.codeproject.com/Articles/25656/Calculating-Percentiles-in-Memory-bound-Applicatio
[2]:https://github.com/CoterJiesen/CalculatingPercentiles
