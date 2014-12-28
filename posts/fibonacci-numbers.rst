.. title: Fibonacci Numbers
.. slug: fibonacci-numbers
.. date: 2014-12-28 20:35:00 UTC+05:30
.. tags: python, algorithm, dynamic-programming, mathjax
.. link: 
.. description: About the Fibonacci number sequence and how to compute it using various approaches, including a basic introduction to dynamic programming.
.. type: text

In this post, we'll examine the Fibonacci number sequence and deal with the problem of finding the ``n`` th Fibonacci number. We'll examine various approaches that steadily increase in efficiency.

.. TEASER_END

The Number Sequence
-------------------

The Fibonacci number sequence is a rather popular sequence of numbers in which every number is the sum of the previous 2 numbers, starting from 0 and 1. Thus, the first few terms of the sequence are :math:`0, 1, 1, 2, 3, 5, 8, 13, \dots`, where, :math:`fib(0) = 0`, :math:`fib(1) = 1`, and so on. Fibonacci numbers have several applications, both in mathematics and computer science, most notably the `Fibonacci Heap`_, although they have been popularized in other places too such as -- *SPOILER ALERT!* -- *The Da Vinci Code*. You can read more about the sequence on the `Online Encyclopedia of Integer Sequences`_.

.. _`Fibonacci Heap`: http://en.wikipedia.org/wiki/Fibonacci_heap

.. _`Online Encyclopedia of Integer Sequences`: http://oeis.org/A000045

Computing The Sequence: A First Attempt
---------------------------------------

Given the definition of the Fibonacci number sequence, it is very easy and very tempting to write a program to compute the ``n`` th Fibonacci number by simply transcribing the definition in code. What this gives is a recursive function for defining a Fibonacci number:

.. code:: python
    :number-lines:

    def fibonacci_recursive(n):

    if n == 0 or n == 1:
        return n
    else:
        return fibonacci_recursive(n-1) + fibonacci_recursive(n-2)

This is a fairly straightforward recursive function. Starting from some value of ``n``, the function is recursively called for smaller values of ``n`` until ``n`` reaches either 0 or 1, which are the base cases for the sequence, i.e. the results for these values is known and, in fact, is used for the rest of the calculations.

Before we move forward, convince yourself that this function is correct, i.e. it gives the right Fibonacci number for any valid value of ``n``.

However, as it turns out, the problem is not the correctness - the problem is something else entirely.

Fire up a Python interpreter, loading up the function above, and evaluate the following:

.. code:: python

    fibonacci_recursive(50)

The function keeps executing, but nothing happens. Now exit by interrupting the execution and try for a lower ``n``, say 5. The code will immediately spit out the correct output. See the problem? This function is inefficient. Not just inefficient, *woefully* inefficient. You'd think computing the 50th Fibonacci number shouldn't be too hard, but the program takes a very long time to perform the task. Before we try to optimize this, we naturally need to find out what is causing this inefficiency.

Let's step through an execution of ``fibonacci_recursive(4)`` and see what happens:

.. code::
    :number-lines:

    fibonacci_recursive(4)
    = fibonacci_recursive(2) + fibonacci_recursive(3)
    = fibonacci_recursive(0) + fibonacci_recursive(1) + fibonacci_recursive(3)
    = 0 + 1 + fibonacci_recursive(3)
    = 1 + fibonacci_recursive(3)
    = 1 + fibonacci_recursive(1) + fibonacci_recursive(2)
    = 1 + 1 + fibonacci_recursive(0) + fibonacci_recursive(1)

And there's our problem.

Notice how, at line 7 above, ``fibonacci_recursive(2)`` resulted in calls to ``fibonacci_recursive(1)`` and ``fibonacci_recursive(0)``? But we already know from a previous call (line 3 onwards) that ``fibonacci_recursive(2)`` evaluates to 2! However, the program will recalculate this fact each and every time it gets a call for ``fibonacci_recursive(2)``, not knowing that it has already been calculated! Not only that, it gets even worse. For ``fibonacci_recursive(5)``, there will be 2 calls to ``fibonacci_recursive(3)``, *each* of which will lead to 2 calls to ``fibonacci_recursive(2)``, and so on. This just explodes, so that by the time 2 digit numbers are reached, the time begins to visibly show up, and for values of ``n`` over 35 or so the program simply stops working in any reasonable timeframe.

A quick ``%timeit`` check using IPython [#]_ confirms this effect of roughly doubling of function calls (your values may vary slightly):

.. code::

    %timeit fibonacci_recursive(5)
    100000 loops, best of 3: 9.05 us per loop

.. code::

    %timeit fibonacci_recursive(6)
    100000 loops, best of 3: 15.2 us per loop

This is a common problem when writing recursive programs - reduplication of work. One needs to check very thoroughly that there is no possibility for the same function call to occur more than once before writing recursive code. However, a good solution that works in many cases is coming up next.

Iterative Approach using Dynamic Programming
--------------------------------------------

We have seen that ``fibonacci_recursive(2)`` was calculated even though its value was already known previously during the execution of the program. The problem was that even though the value was calculated, it was not stored, due to which the program had to recalculate the value for each invocation. This presents a solution: store the values as we obtain them and use them to calculate higher values. This is the essence of the algorithmic approach known as **Dynamic Programming**.

Dynamic Programming mainly involves *storing the solution to smaller problems and using them to calculate the solution to a larger problem*. For the Fibonacci number sequence, this implies computing of the ``n`` th Fibonacci number using the Fibonacci sequence upto ``n-1``, which has been stored previously. Since we already know the values for ``n = 0`` and ``n = 1``, we can use this approach to build up a store of Fibonacci numbers for progressively higher values until we reach the ``n`` th number using an iterative approach. This is called a *bottom-up* approach as we start from the lowest values of ``n`` and move upwards, unlike the previous recursive approach, which is a *top-down apprach*.

This iterative approach can be coded as follows.

.. code:: python
    :number-lines:

    fib = {}            # Storage using dictionary

    # Base values
    fib[0] = 0
    fib[1] = 1

    # Iteration upto 'n'
    for i in range(2, n+1):
        fib[i] = fib[i-2] + fib[i-1]

    return fib[n]

Again, we simply use the definition of Fibonacci numbers, but avoid repetition of work as our loop only moves forwards, using the precomputed values. The time savings clearly show up in ``%timeit`` profiling.

.. code::

    %timeit fibonacci_recursive(20)
    100 loops, best of 3: 13.4 ms per loop

.. code::

    %timeit fibonacci_iterative(20)
    10000 loops, best of 3: 20.4 us per loop

The iterative version is about a thousand times faster at ``n = 20``, and it keeps getting better time savings due to the exponential increase in the recursive function with a linear increase in ``n``. In big-O notation, the time complexity of the recursive function is :math:`O(2^n)`, while for the recursive function, the time complexity is :math:`O(n)`.

While we have saved on time in this iterative function, it has only come at the cost of an extra space requirement for storing all the previous numbers in the Fibonacci sequence. Can we do something about this? Turns out that with just a little bit of thinking, we can!

Efficient Iterative Version
---------------------------

In the iterative implementation, we have stored all the Fibonacci numbers from the 0th to the ``n-1`` th number in order to calculate the ``n`` th Fibonacci number. However, is this really necessary? To check this, we go back to the key computation performed in the iterative function:

.. code:: python
    :number-lines: 9

    fib[i] = fib[i-2] + fib[i-1]

Here, all we are using is the last 2 Fibonacci numbers that were computed. What if we store only those two and keep updating them as we compute the later terms of the Fibonacci sequence? That is exactly what we set out to do next. Instead of a full-blown array/mapping of all the Fibonacci numbers, our storage drops down to merely 3 variables - the current Fibonacci number being computed, and the last 2 numbers already computed, yielding great space savings!

This can be done as follows:

.. code:: python
    :number-lines:

     if n == 0 or n == 1:
        return n

    fib_prev = 0
    fib_curr = 1

    for i in range(2, n+1):
        fib_temp = fib_prev + fib_curr
        fib_prev = fib_curr
        fib_curr = fib_temp

    return fib_curr

Here, ``fib_temp`` is used to calculate the new Fibonacci number using ``fib_curr`` and ``fib_prev``, and then ``fib_prev`` and ``fib_curr`` are updated. The swaps may be slightly tricky to get right on the first try if you haven't used good old pen-and-paper to ascertain their order, but it's rather easy nevertheless to reach a program that is efficient with respect to both time and space if some thought is put in.

It turns out that it is possible to bring the time complexity down to even :math:`O(\log{n})` using Mathematical knowledge of Fibonacci numbers, but that goes beyond the simplicity of using just the basic definition to optimize the program to reach good efficiency levels. You can read more about this here_.

.. _here: http://en.wikipedia.org/wiki/Fibonacci_number#Matrix_form

**Code**

The code in this post is available on the `github repository`_ accompanying the blog, including very few unit tests (``n`` assumed non-negative).

**Exercise**

Write a program to generate the ``n`` th row of the Pascal Triangle. Try to make it as efficient as possible.

.. _`github repository`: https://github.com/DJSagarAhire/blog-code/tree/master/001

.. [#] IPython ``%timeit`` magic: http://ipython.org/ipython-doc/dev/interactive/tutorial.html#magic-functions
