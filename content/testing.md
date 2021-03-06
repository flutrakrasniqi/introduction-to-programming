<h1>Contents<span class="tocSkip"></span></h1>
<div class="toc"><ul class="toc-item"><li><span><a href="#Example-program" data-toc-modified-id="Example-program-1">Example program</a></span></li><li><span><a href="#Built-in-tests" data-toc-modified-id="Built-in-tests-2">Built-in tests</a></span></li><li><span><a href="#Test-functions" data-toc-modified-id="Test-functions-3">Test functions</a></span><ul class="toc-item"><li><span><a href="#Assertions" data-toc-modified-id="Assertions-3.1">Assertions</a></span></li><li><span><a href="#Failing-tests" data-toc-modified-id="Failing-tests-3.2">Failing tests</a></span></li></ul></li><li><span><a href="#Test-runners" data-toc-modified-id="Test-runners-4">Test runners</a></span><ul class="toc-item"><li><span><a href="#pytest" data-toc-modified-id="pytest-4.1">pytest</a></span><ul class="toc-item"><li><span><a href="#Command-line-usage" data-toc-modified-id="Command-line-usage-4.1.1">Command line usage</a></span></li><li><span><a href="#Output" data-toc-modified-id="Output-4.1.2">Output</a></span></li><li><span><a href="#Verbosity" data-toc-modified-id="Verbosity-4.1.3">Verbosity</a></span></li><li><span><a href="#Testing-exceptions" data-toc-modified-id="Testing-exceptions-4.1.4">Testing exceptions</a></span></li></ul></li></ul></li><li><span><a href="#Test-driven-development" data-toc-modified-id="Test-driven-development-5">Test-driven development</a></span></li><li><span><a href="#Exercises" data-toc-modified-id="Exercises-6">Exercises</a></span><ul class="toc-item"><li><span><a href="#1" data-toc-modified-id="1-6.1">1</a></span></li><li><span><a href="#2" data-toc-modified-id="2-6.2">2</a></span></li></ul></li></ul></div>

# Testing

As you have probably noticed by now, it is easy to make mistakes when writing a computer program. Computers cannot easily guess what it is that we want of them, nor do they have any idea of what sort of things it is reasonable to want to do. So we must give a computer precise, [syntactically correct](extras/glossary.md#syntax) instructions that represent exactly what we want to do, and if we get this wrong, the computer will either be unable to do as we asked, in which case our program will break off with an [error](extras/glossary.md#error), or it will happily carry out the instructions we gave it, even when these are not actually the instructions that we wanted to give, and even if we have mistakenly instructed it to do something stupid or malicious such as emailing the contents of our downloads folder to our boss. It is therefore important to check our programs carefully for mistakes.

Up until now, we have been able to check a program fairly easily by simply running it, if it is just a [script](extras/glossary.md#script), or importing it and then using its functions in the console if it is a [module](extras/glossary.md#module). But this sort of informal testing can only take us so far. Once we start writing programs that may take multiple different actions depending on user input, subtle mistakes may be difficult to detect in a quick informal test, because they only arise for some inputs and not others. For these reasons, most software developers test their programs systematically as they go along. Instead of testing a program manually, they write a second program whose only purpose is to test the main program. This test program tries out the main program with different inputs, and verifies that its behavior is as expected. In other words, a test program automates the process of testing the main program.

Writing test programs can be tedious. As if we didn't already have enough work to do writing the main program, we now have to write another one that won't even be part of the finished product. It is tempting to dispense with testing, especially if a project is small. But we should not. Good tests act as a reasonable guarantee that our program functions correctly. And as the saying goes: Most customers prefer a product that actually works.

## Example program

In the spirit of the class so far, we won't actually be developing a product that any healthy customer would want to download. Instead, we have a program for producing [spoonerisms](https://en.wikipedia.org/wiki/Spoonerism). A spoonerism is a play on words in which the initial consonants (or groups of consonants) of two words are swapped. For example:

* crushing blow → blushing crow
* cosy nook → nosy cook
* wasted term → tasted werm

Since this lesson is about testing, we will look at the finished example program already. Our task will be to write some tests for the program. Let's import the program as [module](extras/glossary.md#module) and make some informal tests first. (If you need to refresh your understanding of modules and imports, take a look back at the [lesson on modules](modules.md).)


```python
import spoonerisms

dir(spoonerisms)
```




    ['VOWELS',
     '__builtins__',
     '__cached__',
     '__doc__',
     '__file__',
     '__loader__',
     '__name__',
     '__package__',
     '__spec__',
     'find_first_vowel',
     'spoonerize']



The main function is `spoonerize()`. We can look at its [docstring](extras/glossary.md#docstring) using the `help()` function:


```python
help(spoonerisms.spoonerize)
```

    Help on function spoonerize in module spoonerisms:
    
    spoonerize(wordpair)
        Spoonerize a word pair.
        
        A spoonerism switches the initial consonant clusters of words.
        
        Example:
            >>> spoonerize('smart fella')
            'fart smella'
        
        Argument:
            wordpair: A string containing exactly two words.
        
        Returns:
            A string containing the spoonerized phrase.
        
        Raises ValueError:
            If wordpair does not contain exactly two words.
    


Let's test the example input given in the docstring:


```python
spoonerisms.spoonerize('smart fella')
```




    'fart smella'



This module is very slightly more complex than those we have encountered so far. In addition to the main function, it includes an additional 'helper function' that is [called](extras/glossary.md#call) from within the main function, and whose purpose is to find the location of the first vowel in a word (according to Python's zero-based [indexing](extras/glossary.md#index)). For example:


```python
spoonerisms.find_first_vowel('smart')
```




    2



This helper function in turn depends on a variable called `VOWELS`, a string containing the characters considered to be vowels. These could just have been defined within the `find_first_vowel()` function, but including them as a variable allows us later to add more vowels easily if we want to extend the module's capabilities.

This is a good point to start writing tests; our program has gone beyond just a single function, and we might want to add to its capabilities in the future. Before we begin writing the tests, you might want to take a look at the module file [spoonerisms.py](examples/spoonerisms.py) and quickly assure yourself that you understand how it works.

## Built-in tests

The module in fact already incorporates one short test of its workings. It uses the `__name__` special variable to print out a result when the module is run as the main program. This is something that we learned about in the [lesson on modules](modules.md#__name__). This sort of 'built-in test' can take us quite far, and is especially helpful early on in the development of a program. We will learn about a few ways in which a separate test program can be more convenient as development goes on. But first we need to cover the basics of test programs.

## Test functions

A test program goes in a separate file. This file usually has the same name as the file that it tests, but prefixed with *test_*. So in our case our test file will be called *test_spoonerisms.py*. Among the first things that the test program should do is import the module to be tested, since it will need to [call](extras/glossary.md#call) that module's functions and check their results. After importing the module to be tested, the test program defines functions, each of which tests one aspect of the module.

Test functions look a little different from the functions we have written so far. They do not have any [return value](extras/glossary.md#return), and in most cases they have no input [arguments](extras/glossary.md#argument) either. In place of a return value, most test functions instead make an [assertion](extras/glossary.md#assertion).

### Assertions

What is an assertion? An assertion is yet another kind of [control statement](extras/glossary.md#control). You can think of it as a special kind of [condition](extras/glossary.md#condition), like in an `if` statement. Similar to an `if` statement, an assertion checks whether a particular condition is true or false, for example checking whether two things are equal using `==`. But unlike an `if` statement, we do not specify what happens when the condition is true and what happens when it is false. Instead, an assertion has fixed consequences: If the condition is true, nothing happens and the program continues as normal; if the condition is not true, an [exception](extras/glossary.md#exception) is raised.

An assertion begins with the [keyword](extras/glossary.md#keyword) `assert`. Here is an example of an assertion that is true:


```python
assert 2 + 2 == 4
```

As you can see, nothing happens when Python runs this line. A true assertion just 'passes' and the Python interpreter moves on to the next line of the program.

Compare this with what happens when an assertion is untrue. We get an `AssertionError`:


```python
assert 2 + 2 == 5
```


    ---------------------------------------------------------------------------

    AssertionError                            Traceback (most recent call last)

    <ipython-input-7-71c257695901> in <module>
    ----> 1 assert 2 + 2 == 5
    

    AssertionError: 


We can therefore use an assertion to check whether the [return value](extras/glossary.md#return) of a function is as expected. For example for the `spoonerize()` function from our example module:


```python
result = spoonerisms.spoonerize('smart fella')

assert result == 'fart smella'
```

The simplest test functions are just functions that contain one assertion. No input arguments or return value are needed; the function's only purpose is to alert us by raising an exception if things are not working as expected. Like test files, the names of test functions should be prefixed with `test_`, and then state what is being tested. Here is an example for our `spoonerize()` function, turning the assertion above into a test function.

(If you need first to remind yourself of the syntax for functions, take a look back at the [lesson on functions](functions.md#Defining-functions).)


```python
def test_spoonerize():
    result = spoonerisms.spoonerize('smart fella')
    assert result == 'fart smella'
```

When we [call](extras/glossary.md#call) our test function, no exception is raised, confirming that the assertion was true.


```python
test_spoonerize()
```

Take a look at the finished test file for our example module, [test_spoonerisms.py](examples/test_spoonerisms.py). You will see that it first imports the *spoonerisms* module, then defines several test functions like the one above. Each of these tests one aspect of the workings of the *spoonerisms* module. (There is one slightly different test function at the very end of the file. Ignore this for now; we will come to it in a moment.)

If we would like to test any aspect of the *spoonerisms* module, we can import the test module and run its functions, confirming that no exceptions occur:


```python
import test_spoonerisms

test_spoonerisms.test_find_first_vowel()
test_spoonerisms.test_spoonerize()
```

### Failing tests

For demonstration purposes, the test module includes one test that checks a feature of the *spoonerisms* module that has not yet been implemented (treating the character combination 'qu' as a consonant), so that you can see what happens when a test fails:


```python
test_spoonerisms.test_spoonerize_with_qu()
```


    ---------------------------------------------------------------------------

    AssertionError                            Traceback (most recent call last)

    <ipython-input-12-21a514241976> in <module>
    ----> 1 test_spoonerisms.test_spoonerize_with_qu()
    

    ~/GitHub/introduction-to-programming/content/examples/test_spoonerisms.py in test_spoonerize_with_qu()
         41     result = spoonerisms.spoonerize('faint quartz')  # tenuous, I know
         42 
    ---> 43     assert result == 'quaint fartz'
         44 
         45 


    AssertionError: 


## Test runners

Our solution so far isn't entirely satisfactory. We wanted to have an *automated* test of our module, so that we can, for example, quickly run the tests each time we make changes, and make sure that our changes haven't broken anything. Importing the test module and then running each of its tests individually is very laborious. Because automated testing is such a common task in programming, most programming languages provide time-saving tools for running multiple tests with a single command. These tools are sometimes termed 'test runners'.

### pytest

Python's built-in test runner is provided in a module called `unittest` in the [standard library](standard_library.md). However, there are also a few excellent [third-party packages](standard_library.md#Third-party-packages) that provide test runners with additional features. We will look at one of these instead, called *pytest*. pytest is a very popular third-party test runner package for Python. Since it is provided as part of the default installation of Anaconda, you should not need to install it before trying out the examples below.

If you want to check whether pytest is installed and available for you, you can try [importing](extras/glossary.md#import) it. Check that you do not see a `ModuleNotFoundError` when you try the following command:


```python
import pytest
```

#### Command line usage

pytest is typically run from the operating system's [command line](command_line.md). The command `pytest`, followed by the name of the test file that we would like to run, will run all of the functions within that file whose names begin with `test_`. 

For example:


```
! pytest test_spoonerisms.py
```

    ============================= test session starts ==============================
    platform linux -- Python 3.6.9, pytest-5.3.2, py-1.8.1, pluggy-0.13.1
    rootdir: /home/lt/GitHub/introduction-to-programming/content/examples
    collected 6 items                                                              
    
    test_spoonerisms.py ....F.                                               [100%]
    
    =================================== FAILURES ===================================
    ___________________________ test_spoonerize_with_qu ____________________________
    
        def test_spoonerize_with_qu():
        
            result = spoonerisms.spoonerize('faint quartz')  # tenuous, I know
        
    >       assert result == 'quaint fartz'
    E       AssertionError: assert 'qaint fuartz' == 'quaint fartz'
    E         - qaint fuartz
    E         ?        -
    E         + quaint fartz
    E         ?  +
    
    test_spoonerisms.py:43: AssertionError
    ========================= 1 failed, 5 passed in 0.03s ==========================


Remember a couple of important things from the previous lesson on the [command line](command_line.md):

* For this command to work, the test file *test_spoonerisms.py* and the module to be tested *spoonerisms.py* need to be located in your current working directory.
* Commands beginning with `!` are not standard Python commands, they are commands for the operating system's command line. They only work in the Spyder console, not in a Python program.

pytest can even be used to run tests from more than one test file. If we do not specify the name of a test file, then pytest will search for all files in the current working directory whose names begin with *test_*, and run all the functions within those files whose names begin with `test_`. This feature becomes very useful once we have a more complex program spread over multiple files. A typical pattern of organization is to create separate test files for each major component of the program to be tested.

Here for demonstration purposes I have just included a second test file [test_math_is_working_as_normal.py](examples/test_math_is_working_as_normal.py) containing some spurious tests:


```
! pytest
```

    ============================= test session starts ==============================
    platform linux -- Python 3.6.9, pytest-5.3.2, py-1.8.1, pluggy-0.13.1
    rootdir: /home/lt/GitHub/introduction-to-programming/content/examples
    collected 8 items                                                              
    
    test_math_is_working_as_normal.py ..                                     [ 25%]
    test_spoonerisms.py ....F.                                               [100%]
    
    =================================== FAILURES ===================================
    ___________________________ test_spoonerize_with_qu ____________________________
    
        def test_spoonerize_with_qu():
        
            result = spoonerisms.spoonerize('faint quartz')  # tenuous, I know
        
    >       assert result == 'quaint fartz'
    E       AssertionError: assert 'qaint fuartz' == 'quaint fartz'
    E         - qaint fuartz
    E         ?        -
    E         + quaint fartz
    E         ?  +
    
    test_spoonerisms.py:43: AssertionError
    ========================= 1 failed, 7 passed in 0.05s ==========================


#### Output

pytest provides a lot of additional useful information that we did not get when just running the test functions manually. Let's go through the output from the pytest session we ran above and see what it all means.

> `rootdir: /home/lt/GitHub/introduction-to-programming/content/examples`

This is a confirmation of the working directory that we ran pytest from. The pytest session will have included test files from this directory and its subdirectories.

> `collected 8 items`

We can read here how many test functions pytest found in our test files.

> `test_math_is_working_as_normal.py ..`
> `test_spoonerisms.py ....F.`

Here we see each test file listed individually. After the name of the file, each dot represents a test that passed, and each 'F' represents a test that failed.

The *FAILURES* section then provides details of each failed test. As well as just the standard `AssertionError`, pytest tells us some more about the specific nature of the failure. In complex cases, this output can be very detailed indeed, for example listing all the variables that were available at the time of the failure, what their values were, and so on. In this fairly simple case the crucial line is this one:

> `E       AssertionError: assert 'qaint fuartz' == 'quaint fartz'`

Here we see what the result of the `spoonerize()` function really was, compared to what our test expected it to be.

#### Verbosity

If you would like to see even more information about the tests, you can run pytest in '[verbose](extras/glossary.md#verbose)' mode. In computing, the term 'verbose' has approximately the same meaning as in everyday English, but in reference to the output of a computer program rather than a human being's use of language. A program that is run in verbose mode will 'talk more', i.e. give more detailed printed output.

The option for running pytest in verbose mode is `--verbose`, or in abbreviated form `-v` (look back at [the previous lesson](command_line.md#options) if you need to remind yourself about command line options). The main difference in a simple case like ours is that we see the names of the individual test functions alongside their passed/failed status:


```
! pytest --verbose
```

    ============================= test session starts ==============================
    platform linux -- Python 3.6.9, pytest-5.3.2, py-1.8.1, pluggy-0.13.1 -- /home/lt/GitHub/introduction-to-programming/venv/bin/python3
    cachedir: .pytest_cache
    rootdir: /home/lt/GitHub/introduction-to-programming/content/examples
    collected 8 items                                                              
    
    test_math_is_working_as_normal.py::test_two_plus_two_equals_four PASSED  [ 12%]
    test_math_is_working_as_normal.py::test_division_by_zero_is_undefined PASSED [ 25%]
    test_spoonerisms.py::test_find_first_vowel PASSED                        [ 37%]
    test_spoonerisms.py::test_spoonerize PASSED                              [ 50%]
    test_spoonerisms.py::test_spoonerize_with_multiple_consonants PASSED     [ 62%]
    test_spoonerisms.py::test_spoonerize_with_initial_vowel PASSED           [ 75%]
    test_spoonerisms.py::test_spoonerize_with_qu FAILED                      [ 87%]
    test_spoonerisms.py::test_spoonerize_exception PASSED                    [100%]
    
    =================================== FAILURES ===================================
    ___________________________ test_spoonerize_with_qu ____________________________
    
        def test_spoonerize_with_qu():
        
            result = spoonerisms.spoonerize('faint quartz')  # tenuous, I know
        
    >       assert result == 'quaint fartz'
    E       AssertionError: assert 'qaint fuartz' == 'quaint fartz'
    E         - qaint fuartz
    E         ?        -
    E         + quaint fartz
    E         ?  +
    
    test_spoonerisms.py:43: AssertionError
    ========================= 1 failed, 7 passed in 0.04s ==========================


#### Testing exceptions

The [docstring](extras/glossary.md#docstring) for our `spoonerize()` function mentions an [exception](extras/glossary.md#exception) (see [here](examples/spoonerisms.py#L44)). If we have promised our users that certain situations will result in an exception, then we should also test this. But testing the occurrence of an exception presents a slight problem. We would like our test to continue as normal if the expected exception is raised, but we would like it to raise an exception otherwise, which is the opposite of what exceptions normally do.

pytest provides a function called `raises()` to handle this kind of test. This function works together with the Python `with` [keyword](extras/glossary.md#keyword) to create a context in which to test for the occurrence of an exception. Our program will not be interrupted if the expected exception is raised within the `raises()` context, but an exception will be raised if we reach the end of the context and the expected exception has *not* been raised.

If this sounds a little abstract, it may be clearer with an example (and if you need to remind yourself about the `with` keyword, look back at the [lesson on files](files.md#Context-managers), where we first encountered it):


```python
with pytest.raises(ValueError):
    int('fifty-two')
```

Nothing happens here, because the indented line `int('fifty-two')` does indeed raise a `ValueError`. Compare what happens with a line that does not raise a `ValueError`. The final line of the error message is nicely informative:


```python
with pytest.raises(ValueError):
    int('52')
```


    ---------------------------------------------------------------------------

    Failed                                    Traceback (most recent call last)

    <ipython-input-18-8e4b5f8643d9> in <module>
          1 with pytest.raises(ValueError):
    ----> 2     int('52')
    

    ~/GitHub/introduction-to-programming/venv/lib/python3.6/site-packages/_pytest/python_api.py in __exit__(self, exc_type, exc_val, exc_tb)
        743         __tracebackhide__ = True
        744         if exc_type is None:
    --> 745             fail(self.message)
        746         assert self.excinfo is not None
        747         if not issubclass(exc_type, self.expected_exception):


    ~/GitHub/introduction-to-programming/venv/lib/python3.6/site-packages/_pytest/outcomes.py in fail(msg, pytrace)
        126     """
        127     __tracebackhide__ = True
    --> 128     raise Failed(msg=msg, pytrace=pytrace)
        129 
        130 


    Failed: DID NOT RAISE <class 'ValueError'>


pytest's `raises()` function even allows us to test for specific content in the exception's error message. The additional `match` [argument](extras/glossary.md#argument) specifies a string pattern to search for in the error message. If the pattern is not found, the final line of the error message again describes the failure in detail:


```python
with pytest.raises(ValueError, match='OMG that is not valid. You have broken your computer FOR EVER.'):
    int('fifty-two')
```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    <ipython-input-19-a8e5f8cbc3a4> in <module>
          1 with pytest.raises(ValueError, match='OMG that is not valid. You have broken your computer FOR EVER.'):
    ----> 2     int('fifty-two')
    

    ValueError: invalid literal for int() with base 10: 'fifty-two'

    
    During handling of the above exception, another exception occurred:


    AssertionError                            Traceback (most recent call last)

    <ipython-input-19-a8e5f8cbc3a4> in <module>
          1 with pytest.raises(ValueError, match='OMG that is not valid. You have broken your computer FOR EVER.'):
    ----> 2     int('fifty-two')
    

    ~/GitHub/introduction-to-programming/venv/lib/python3.6/site-packages/_pytest/python_api.py in __exit__(self, exc_type, exc_val, exc_tb)
        753         self.excinfo.fill_unfilled(exc_info)
        754         if self.match_expr is not None:
    --> 755             self.excinfo.match(self.match_expr)
        756         return True


    ~/GitHub/introduction-to-programming/venv/lib/python3.6/site-packages/_pytest/_code/code.py in match(self, regexp)
        640         __tracebackhide__ = True
        641         if not re.search(regexp, str(self.value)):
    --> 642             assert 0, "Pattern {!r} not found in {!r}".format(regexp, str(self.value))
        643         return True
        644 


    AssertionError: Pattern 'OMG that is not valid. You have broken your computer FOR EVER.' not found in "invalid literal for int() with base 10: 'fifty-two'"


The final test in our example test file uses the `raises()` function to test that the exception that we wrote into the `spoonerize()` function is working properly. Take a look at it [here](examples/test_spoonerisms.py#L46) and check that you understand how it works.

There is a *lot* more that pytest can do, but that covers our basic needs at this level. If you would like to explore some more useful pytest features, you can read the guides at the [pytest documentation site](https://docs.pytest.org/en/latest/).

## Test-driven development

In the example above, I presented a completed example program, and we learned about the process of writing tests for the program. In reality, many software developers follow the opposite pattern: They first write a test for a new feature that they would like their main program to have, and only then do they write that part of the main program.

This approach to software development is broadly termed 'test-driven development' (or [TDD](extras/glossary.md#tdd)), because it uses tests to guide the development of the program. The general working pattern in test-driven development is as follows:

* First write a test.

This might be a test that checks for a new feature of the main program that hasn't yet been written. Or it might be a test that tests a mistake or [bug](extras/glossary.md#bug) that somebody has reported in our program.

* Run the test and check that it fails.

Importantly, we should not only check that the test fails, but also check that it fails *for the reason we expected*. For example, our test should not fail because we have made some mistake in the test itself, such as naming a variable incorrectly. Instead, the test should fail by encountering exactly the problem or omission that we wrote the test to identify. A test that fails for the reason that we expect it to fail is known as an 'expected failure'. Expected failures are important for first assuring ourselves that we have written the test itself correctly before we continue.

* Now write part of the main program so that it passes the test.

This is the 'normal' part of test-driven development, in which we write a part of our program.

* Run *all* your tests.

When we think we have got the main program right, we should run not just the new test that we just wrote but all of the tests that we have written so far. This ensures that not only have we got the new part of our program right, but we have also not inadvertently broken any of the existing parts of our program in the process. A situation in which a change to one part of a program unexpectedly breaks other aspects of the program is termed a '[regression](extras/glossary.md#regression)', and becomes more likely as our program becomes more complex. One of the purposes of testing in general is to help spot regressions as soon as they occur.

* Once the test passes, write the next test.

There is rather more to test-driven development, and to testing in general, than what we have covered here, but these are the basic starting points. Try some exercises.

## Exercises

### 1

The test function `test_spoonerize_with_qu()` in our test file [test_spoonerisms.py](examples/test_spoonerisms.py) fails. Improve the main [spoonerisms module](examples/spoonerisms.py) so that this test passes. When you think that you have got it right, run all the tests in the test file using pytest. Check that `test_spoonerize_with_qu()` now passes, and make sure that your changes have not introduced any [regressions](extras/glossary.md#regression) that cause the other tests to fail.

### 2

The letter *y* presents some problems for spoonerisms in English. Sometimes it is a vowel and sometimes it is a consonant. Assume the following simple rule which will work in most cases:

> At the beginning of a word, *y* is a consonant. Otherwise it is a vowel.

According to this rule, a spoonerism should treat an initial *y* as the 'head' of a word and should swap it with the head of the other word in the spoonerism as normal. For example:

* four years → your fears

But a *y* elsewhere should be treated as a vowel and not included in the 'head' of the word. For example:

* sly drinkies → dry slinkies

Modify the [spoonerisms module](examples/spoonerisms.py) to take this new rule into account. But do so according to the test-driven development pattern as described above. Write a new test function in the [test_spoonerisms.py](examples/test_spoonerisms.py) test file first, and verify that you get the expected failure. Only then may you write your planned improvement.

You might want to write more than one test function to make sure that you have covered all the relevant possibilities.

When you have made your changes to *spoonerisms.py*, use pytest to run all of the test functions in the test file. Check that your new test function passes and that your changes have not caused any of the other tests to fail.
