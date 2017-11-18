---
title: "Property-based testing in Python"
image: "/assets/img/events/prop-based.jpg"
tags:
  - Testing
  - Python
last_modified_at: 2017-11-10T22:17:49-04:00
---



Writing tests helps to define more precisely a method intent. This is widely considerate necessary, and people look at
the code coverage ratio as an important metric to estimate your code quality. However, testing may be really hard. 
As developers, we all know that very great test cases can avoid hours of debugging but do we always take time to create
rigorous and comprehensive test? Even if we want to, being responsible of the code design make you quite unlikely to 
think about what could happen in extremely scenarios.

 
## Quick introduction to Property-based testing

The concept of property-based testing is not new. It appeared in the 90s, at this moment the need of test automation
was already there. As I said in this post introduction, A software developer is unlikely to detect a big flaw with a
test that he would have design himself. The idea behind property-based testing is to use precises requirement to verify
the proper program behaviour. 

> Property-based testing uses property specifications and a data-flow analysis of the program to
guide evaluation of test executions for correctness and completeness. <br> <b>George Fink and Matt Bishop (1997)</b>


## Hypothesis

> Hypothesis is a Python library for creating unit tests which are simpler to write and more powerful when run,
finding edge cases in your code you wouldn’t have thought to look for. It is stable, powerful and easy to add to any
existing test suite.

Install Hypothesis with pip :

```
pip install hypothesis
```
 
For your information, I am currently using the version 3.36.0 of this module. To be honest, I have discover what
property-based testing is last weak, thanks to the CommBank's Data Engineer team. As it sounded awesome, I decided to
learn more about it and this blog post is a way to go deeper into this testing method.

## Basic example and implementation

Let's see an example, given by Hypothesis in their official [Github](https://github.com/HypothesisWorks/hypothesis-python/blob/master/examples/test_rle.py).

This example shows how to test a very simple operation: encoding and decoding a list of integers. 

```python encoding_functions https://github.com/HypothesisWorks/hypothesis-python/blob/master/examples/test_rle.py source
def run_length_encode(seq):
    """Encode a sequence as a new run-length encoded sequence."""
    result = [[seq[0], 0]]
    for elem in seq:
        if elem == result[-1][0]:
            result[-1][1] += 1
        else:
            result.append([elem, 1])
    return result


def run_length_decode(seq):
    """Take a previously encoded sequence and reconstruct the original from
    it."""
    result = []
    for s, i in seq:
        for _ in range(i):
            result.append(s)
    return result
```

Now, let's use a basic test using Hypothesis. If we encode a sequence and then decode the result, we should get the
original sequence back. The ```@given``` decorator will help us define the test strategy. So the input can be any list
of Integers in the rang 0 to 10 to increase the probability of having identical successive values in the list, otherwise there
would not be any compression and the encoding method will return the input list.

```python
import hypothesis.strategies as st
from hypothesis import given

@given(st.lists(st.integers(0, 10)))
def test_decodes_to_starting_sequence(ls):
    assert run_length_decode(run_length_encode(ls)) == ls
```

Running the ```pytest```command will provide the following output:

```sh
 seq = []

    def run_length_encode(seq):
        """Encode a sequence as a new run-length encoded sequence."""
>       result = [[seq[0], 0]]
E       IndexError: list index out of range

prop_based_test.py:7: IndexError
------------------------------------------------------------------------- Hypothesis --------------------------------------------------------------------------
Falsifying example: test_decodes_to_starting_sequence(ls=[])
You can add @seed(150582365458376902792718888049345413633) to this test or run pytest with --hypothesis-seed=150582365458376902792718888049345413633 to reproduce this failure.
```

Of course, it is quite obvious that we need to check either the input list is empty or not, but hypothesis saw it
immediately whereas we could have missed it in a test case. Let's do this easy fix right now using ```if``` on a python
list will prevent both None and empty.

```python
def run_length_encode(seq):
    """Encode a sequence as a new run-length encoded sequence."""
    if not seq:
        return []
    result = [[seq[0], 0]]
    for elem in seq:
        if elem == result[-1][0]:
            result[-1][1] += 1
        else:
            result.append([elem, 1])
    return result
```

Now the first test is ok. But we need to make sure that the we are actually using the encoding process. To do so, we
will take a list, and add duplicate an element in this list. To get rid of dimension problem, let's say that the input
list can not have less than 5 elements using the Hypothesis strategies option ```min_size```. 

```python
import hypothesis.strategies as st
from hypothesis import given

@given(st.lists(st.integers(0, 10), min_size=5))
def test_duplicating_an_element_does_not_increase_length(ls):
    # Copy the input list
    ls2 = list(ls)
    # Duplicate the element in position 3.
    ls2.insert(3, ls2[3])
    assert len(run_length_encode(ls2)) == len(run_length_encode(ls))
```

The test results are ok:

```sh
[antony@MacBook:~/CodeDirectory/]$ pytest -s --hypothesis-show-statistics
[...]
==================================================================== Hypothesis Statistics ====================================================================
prop_based_test.py::test_decodes_to_starting_sequence:

  - 100 passing examples, 0 failing examples, 0 invalid examples
  - Typical runtimes: 0-1 ms
  - Stopped because settings.max_examples=100

prop_based_test.py::test_duplicating_an_element_does_not_increase_length:

  - 100 passing examples, 0 failing examples, 0 invalid examples
  - Typical runtimes: 0-2 ms
  - Stopped because settings.max_examples=100
```

But the previous test is not a good example of Hypothesis, we could even say that it is quite bad. The reason is simple,
we have limited the set of possible input and added a property just to pass our properties-based test. Let's see how
to make a smarter test in the next section.

## Go a bit further with statistics and settings

Hypothesis provide many great features. In our previous test, we did not have any invalid example because we reduce the
scope of values generated for the test. With the ```assume``` method, we are able to get rid of the min_size condition
because if we try to access an index out of bound, the test is simply considered invalid. So now, the input list can be
empty or with a very small size and the index where we are going to replicate an element of this list is also chosen
randomly between 0 et 10. 

```python
import hypothesis.strategies as st
from hypothesis import given, settings, assume, event
@given(st.lists(st.integers(0, 10)), st.integers(0, 10))
@settings(max_examples=400)
def test_duplicating_an_element_does_not_increase_length(ls, i):
    # We use assume to get a valid index into the list. We could also have used
    # e.g. flatmap, but this is relatively straightforward and will tend to
    # perform better.
    event('Index I was out of bound {}'.format(i > len(ls)))
    assume(i < len(ls))

    ls2 = list(ls)
    # duplicating the value at i right next to it guarantees they are part of
    # the same run in the resulting compression.
    ls2.insert(i, ls2[i])
    assert len(run_length_encode(ls2)) == len(run_length_encode(ls))
```

Using settings, we can increase the number of test runs. Here you can see that most of the time the random index is
out of bound but anyway, we still have 400 passing examples, which is far more than we would have done using handwritten
examples!

```sh
[antony@MacBook:~/CodeDirectory/]$ pytest -s --hypothesis-show-statistics
[...]
prop_based_test.py::test_duplicating_an_element_does_not_increase_length:

  - 400 passing examples, 0 failing examples, 562 invalid examples
  - Typical runtimes: 0-1 ms
  - Stopped because settings.max_examples=400
  - Events:
    * 55.30%, Index I was out of bound True
    * 44.70%, Index I was out of bound False
```   

## Resources and references
 
Among all the provided resources, I would definitely recommend to see Matt Bachman's
[speak](https://www.youtube.com/watch?v=jvwfDdgg93E) at PyCon 2016.   
 
- George Fink & Matt Bishop - "[Property-Based Testing; A New Approach to Testing for Assurance](https://pdfs.semanticscholar.org/8b1f/371310de3e237a994be89393373e27126593.pdf)" - Department
of Computer Science University of California, July 1997 
- Matt Bachmann - "[Better Testing With Less Code: Property Based Testing With Python](https://www.youtube.com/watch?v=jvwfDdgg93E)" - PyCon 2016
- David MacIver - "[Episode #67: Property-based Testing with Hypothesis](https://talkpython.fm/episodes/show/67/property-based-testing-with-hypothesis)" Jul 13, 2016
- David R. MacIver - [Hypothesis Official Documentation](https://hypothesis.readthedocs.io/en/latest/index.html) - Since 2013
- Bill Venners and Artima - [Scala Test Official Documentation](http://www.scalatest.org/user_guide/property_based_testing) - Since 2009
- Kenneth Reitz - [Testing Your Code](http://docs.python-guide.org/en/latest/writing/tests/) - 2016
