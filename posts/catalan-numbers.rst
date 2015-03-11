.. title: Catalan Numbers
.. slug: catalan-numbers
.. date: 2015-03-11 20:00:00 UTC+05:30
.. tags: mathjax, algorithm, dynamic-programming, python
.. link: 
.. description: 
.. type: text

In this post, we will examine the Catalan number sequence and have a look at some of its vast applications in computer science and combinatorics in general. Then we will figure out an efficient way to compute the ``n`` th Catalan number.

.. TEASER_END

The Number Sequence
-------------------

The Catalan number sequence is given by the following formula:

.. math::

    C_n = \frac{1}{n+1} \binom{2n}{n}

Some of the first few terms in the sequence are as :math:`1, 1, 2, 5, 14, 42, 132, 429, \dots`. As you can see, this sequence grows exponentially.

While the formula itself may seem contrived, the Catalan number sequence actually yields very easily to recursively-defined objects (which are found all over the place in computer science thanks to ubiquitous applications of discrete data structures). This is because the numbers satisfy the following recurrence:

.. math::

    C_{n+1} = \sum_{i=0}^{n} C_i C_{n-i}

As a result, the Catalan number sequence can be interpreted as the number of ways to recursively create a structure as follows:

* Take ``n`` elements, ordered as :math:`1, 2, \dots, n`.
* Choose an element ``e`` as a pivot.
* Separate the ``n`` elements into two parts using ``e``.
* Perform the procedure recursively on the two parts.
* Join the two parts together with ``e`` in the middle.

This leads to a vast number of applications of the Catalan number sequence in various areas of computer science and combinatorics in general. You can read a nice list on the `Online Encyclopedia of Integer Sequences`_, but be warned:

    "This is probably the longest entry in the OEIS, and rightly so."

.. _`Online Encyclopedia of Integer Sequences`: http://oeis.org/A000108

We now have a short look at some of the applications below so that you don't have to go through the entire thing (I haven't either!):

1: **Pairs of Balanced Parentheses**

The Catalan number :math:`C_n` gives the number of ways in which ``n`` balanced pairs of parentheses can be arranged. For example, for ``n = 3``, the 5 arrangements are given by :math:`()()(), ()(()), (())(), ((())), (()())`.

2: **Dyck Paths**

A *Dyck Path* in a matrix is a path that starts at :math:`(0, 0)` and ends at :math:`(n, n)` without going past the diagonal of the matrix (touching the diagonal is allowed). All the possible such paths in a matrix where :math:`n = 4` are illustrated in the following figure (from Wolfram_):

.. image:: http://mathworld.wolfram.com/images/eps-gif/StaircaseWalkDiagonal_950.gif
    :scale: 50%

As you can observe, this is :math:`C_4`, the fourth Catalan number.

.. _Wolfram: http://mathworld.wolfram.com/DyckPath.html

3: **Binary Search Trees**

The Catalan number :math:`C_n` also gives the number of possible binary search trees having ``n`` items. As an exercise, construct the 5 possible binary search trees that can be constructed using the items :math:`1, 2, 3`.

Computing The Sequence: A First Attempt
---------------------------------------

Just like `Fibonacci numbers`_ before, we can make an initial attempt to compute the ``n`` th Catalan number by directly converting the definition into a recursive function. However, just like Fibonacci numbers, we will soon realize that this is a highly inefficient approach and can be improved upon...

.. code:: python
    :number-lines:

    def catalan_recursive(n):

    if n == 0:
        return 1

    res = 0
    for i in range(n):
        res += catalan_recursive(i) * catalan_recursive(n-i-1)

    return res

.. _`Fibonacci numbers`: http://djsagarahire.github.io/posts/fibonacci-numbers.html

The code is a rather straightforward implementation of the definition. Unfortunately, it is also very slow, as the following might suggest:

.. code::

    %timeit catalan_recursive(12)
    1 loops, best of 3: 571 ms per loop

Computing merely the 12th Catalan number takes over half a second, and this time keeps going up dangerously fast as the numbers increase. This is even more so than Fibonacci numbers, because while the ``n`` th Fibonacci number requires only the ``n-1`` and ``n-2`` th Fibonacci numbers, the ``n`` th Catalan number requires *all* Catalan numbers from 0 to ``n-1`` !

An examination of the code above reveals that all these required Catalan numbers are recomputed again and again for every single call to the ``catalan_recursive()`` function, which is incredibly wasteful. Just like Fibonacci, we may need to optimize the function by computing these values beforehand and storing them separately.

Dynamic Programming
-------------------

We now make use of Dynamic Programming's idea of using a cache of previously computed values to help in the future execution of the program. This is made possible by observing that once the Catalan numbers from 0 to ``n-1`` are calculated, the ``n`` th Catalan number can be calculated using these values. Thus it makes sense to calculate all Catalan numbers in an increasing order till we reach ``n`` . This is easily possible by looping over 0 to ``n`` as follows, computing a Catalan number in an iteration of the loop:

.. code:: python
    :number-lines:

    def catalan_iterative(n):

    cat = {}
    cat[0] = 1

    for i in range(1, n+1):
        cat[i] = 0
        for j in range(i):
            cat[i] += cat[j] * cat[i-j-1]

    return cat[n]

As no repeated computations are performed here, this code runs extremely fast compared to the previous inefficient direct recursive implementation. For example, even the 500th Catalan number can be computed in under a quarter of a second:

.. code::

    %timeit catalan_iterative(500)
    1 loops, best of 3: 241 ms per loop

However, taking algorithmic complexity into account, this code is still slower than, say, Fibonacci, which was linear in the size of the input :math:`n`, i.e. it was a :math:`O(n)` algorithm. However, ``catalan_iterative()`` actually takes quadratic time in its input, i.e. it is an :math:`O(n^2)` algorithm. This is because computing the ``n`` th Catalan number requires keeping a running product of all previous ``n`` Catalan numbers, which takes :math:`O(n)` time. However, all the previous Catalan numbers have been computed in a similar way, which means there are :math:`n` operations requiring :math:`O(n)` time each, totalling to :math:`O(n^2)` time.

Can we do better? Turns out we can, but it requires some additional knowledge of Catalan numbers.

Product Method
--------------

While the definition of Catalan numbers that ends up being useful in several applications is inherently recursive in nature, there actually exists a closed-form formula to find the :math:`n` th Catalan number that is rather neat in its simplicity. It is given as a product of fractions, as follows:

.. math::

    C_{n} = \prod_{k=2}^{n} \frac{n+k}{k}

This formula allows for an extremely simple and elegant solution that runs in only :math:`O(n)` time!

.. code:: python
    :number-lines:

    def catalan_product(n):

    res = 1
    for i in range(2, n+1):
        res *= ((n+i) / i)

    return round(res)

This is blazingly fast as even the 500th Catalan number is computed in a manner of microseconds! We will usually not need to go much further than 500, because this is what the 500th Catalan number looks like: 539497486917039541523668013532875950744221976926982410442251334092448206102518802405822724731244098680372472697985449906977694838252719562184161888508030810946145210118969919796252331316942596673432265927842786392407044486912076697303449352167645593722354161161400821904867145146775262542096760832.

Yeah, it's *that* big.

**Code**

The code in this post is available on the `github repository`_ accompanying the blog.

.. _`github repository`: https://github.com/DJSagarAhire/blog-code/tree/master/002
