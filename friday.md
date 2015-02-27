# Friday's Python learning schedule

* Morning: Unstructured time, work on projects discuss, stacks and generators. Look at my solution to the train problem if desired.
* Afternoon: Walt will be in the classroom to help with getting started on Django.

## Stacks: what are they good for?

The [Wikipedia page on stacks](http://en.wikipedia.org/wiki/Stack_%28abstract_data_type%29) is actually quite good and not too long or complicated, so go there if you want to learn more.

I think everyone understands what a stack _is_ at this point, so the question remains why one would use them. The example in the train problem is clearly contrived, but there are plenty of non-contrived reasons why one would want to use them.

Most of the time stacks are used it's for the purpose of keeping track of pieces of data that will later be revisited in a specific order. Imagine you are trying to navigate a maze. If you had a stack and every time you had a choice of which way to go you pushed that value onto the stack, then when you ran into a dead end, you could trace your steps backwards, popping the choices off as you went and trying the next option at each intersection. This way you systematically explore the maze and find the solution efficiently. A large class of problems can be solved this way including things like sudoku. The formal name for this method is [backtracking](http://en.wikipedia.org/wiki/Backtracking).

The most pervasive use of stacks in programming is for keeping track of program execution in the form of the call stack. Since a function can be called from potentially many different places, and we need to know where to go when the function returns, we need some place to store that data. We also usually need to keep track of the values of any parameters to the functions because the same function can be called multiple times with different, independent parameter values. Otherwise recursion wouldn't work. The call stack is where we keep all this information. At any given point you can see the entire call stack within `pdb` by typing `bt` for **b**ack**t**race. Here's an example from `howdoi`:

    (Pdb) bt
    -> load_entry_point('howdoi==1.1.7', 'console_scripts', 'howdoi')()
      howdoi.py(257)command_line_runner()
    -> print(howdoi(args).encode('utf-8', 'ignore'))
      howdoi.py(208)howdoi()
    -> return get_instructions(args) or 'Sorry, couldn\'t find any help with that topic\n'
      howdoi.py(184)get_instructions()
    -> answer = get_answer(args, links)
    > howdoi.py(139)get_answer()
    -> def get_answer(args, links):

I've edited out some things for clarity, but we can see the sequence of calls from oldest at the top to newest at the bottom. Somewhat confusingly, the bottom of the trace (`get_answer()`) is the "top" of the stack in that it was the last thing added to the stack, and so will be the first thing removed when it returns and the data associated with it (`args` and `links` and the knowledge to return to line 184 inside of `get_instructions()`) is popped off the stack.

## Generators and coroutines

Generators are really just functions (or methods) that contain `yield` statements. There's a good [explanation of the details](https://docs.python.org/2/tutorial/classes.html#generators) on the Python docs site, but the basics are just:

1. If a function has a `yield` statement in it, calling that function just returns a generator object without actually running any of the code inside the function.
2. Add a `yield foo` statement when you want to return a value to the caller's `next()` call. The `next()` call is implicitly used to assign the `x` in the `for x in y` Python construct.
3. After a `yeild` statement, control returns to the code that called `next()` until the subsequent `next()` call.
4. If the end of the generator function or method is reached, a `StopIteration` exception is raised. This is caught by the `for x in y` construct, so there's no need to worry about it, unless you're calling `next()` explicitly.

Generators are a pretty simple way to get multiple different return values out of a single function when you ask it repeatedly. However, the `next()` call doesn't take any parameters. So, what if you wanted to do something fancier, where you passed some data in? Enter coroutines and the `send()` method.

How and why you would use coroutines is fairly involved, but you can read the [Wikipedia page on coroutines](http://en.wikipedia.org/wiki/Coroutine) if you are interested. As to _how_ to use them in Python, it's not so different from a generator. The same rules about returning a generator object apply, but there are some new wrinkles:

1. The `yield` statement can now have a return value.
2. In addition to calling `next()` on the generator object, you can now also call `send(foo)` with an arbitrary argument `foo`. Inside the generator, the value of the corresponding `yield` expression will be `foo`.
3. Because of reasons, you must first call either `next()` or `send(None)` on the generator object before calling `send(foo)` on it. There is more explanation of that in [PEP 342: Coroutines via Enhanced Generators](https://www.python.org/dev/peps/pep-0342/).

Here's an example of using `send()`:

    def i_am_a_generator():
        print "Before first yield"
        y1 = yield
        print "After first yield, y1 = %s" % y1
        y2 = yield
        print "After second yield, y2 = %s" % y2

    >>> import generators
    >>> g = generators.i_am_a_generator()
    >>> g.send("Hi!")
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: can't send non-None value to a just-started generator
    >>> g.send(None) # calling g.next() would be equavalent
    Before first yield
    >>> g.send("Yo!")
    After first yield, y1 = Yo!
    >>> g.send(42)
    After second yield, y2 = 42
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    StopIteration

https://docs.python.org/2/reference/expressions.html#generator-iterator-methods

## Practical Python knowledge (continued)

### Where to look for Python information

Most often, when I need to know how to use a bit of the standard library, I'm already in the `python` interpreter. Happily, it makes available a handy on-line help system, that you can invoke just by typing `help()`. With no arguments, it takes you to an interactive system to find the topic you're looking for. If you know what you want information about, you can pass `help()` an object and it will give you the documentation for that object. You'll need to already have `import`ed it if it's part of a moule. In general though, you don't even need to know what the object is to find the help. For example:

    a = list()
    help(a)
  
Will give you the docs for the `list` class. This is a bit contrived, but it's handy if you're in the middle of debugging to just use a variable that you want to know more about what you can do with it. You can also type `help([])` or `help(list)`. One set of docs that is not so obvous how to access is the help for all the built-in functions (e.g., `len()`, `sorted()`, `enumerate()`). You can access that with the rather confusing `help(__builtins__)`. You can also pass entire module names:

    import random
    help(random)

That is something I type often because I can never remember the name of `random.randrange()`. When you invoke `help()`, it will take you to a viewer powered by the `less` command. Type `h` for help on navigating within `less`. The most useful thing to know is that you can search by typing `/foo` to find occurrences of "foo" and then type `n` to go to the next one. If you happen to know what method you're interested in, you can type that directly. I often type `help(random.sample)` because I can never remember the order of the arguments.

For more detailed information, [docs.python.org](https://docs.python.org) is the place to go. One thing to be careful of is what version of documentation you're looking at. At Isilon, we use 2.6.1. Your computers likely have 2.7.x installed.

Of course, don't forget your old friends Goolse and Stack Overflow for answers to specicifc questions. Python's "explicit is better" philosophy does have the benefit of making it quite googlable.

### The standard library functionality that you should be aware of

You should read over the docs for all the [built-in functions](https://docs.python.org/2/library/functions.html#help). A couple that I think are pretty neat and didn't realize existed until I saw them in code are:

* `any()`
* `all()`
* `bin()`
* `dir()`
* `enumerate()`
* `locals()`
* `repr()`
* `reversed()`
* `zip()`

Other modules in the standard library that I've found useful, but that I wouldn't have necessarily guessed existed:

* `string`: Contains handy constants that lets you do things like `x in string.letters`
* `re`: Regular expressions
* `pprint`: Pretty-printing; great for dicts and such
* `random`: Generate random numbers, select random samples, etc
* `fileinput`: A handy way to write a script that reads from file arguments or standard input
* `pickle`: Save objects to a file and read them back automagically
* `csv`: Access comma-separated value files
* `popen2`: For running shell commands

### Random handy things

#### Turn an attribute into a property

In ruby, if you have an object instance `foo` with a simple attribute called `bar`, it's easy to later make the behavior more complex by adding `def bar` and/or `def bar=` methods to your class. In Python, if you change an attribute to a method, all the places in the code that access `foo.bar` will now get a method object. Compare:

    class Foo(object):
        def __init__(self):
            self.bar = False
    >>> f = Foo()
    >>> f.bar
    False

    class Foo(object):
        def bar(self):
            return False
    >>> f = Foo()
    >>> f.bar
    <bound method Foo.bar of <__main__.Foo object at 0x1085d5310>>
    >>> f.bar()
    False

This can be a particular problem if your code interprets `f.bar` in a boolean context. However, rather than changing all the places `bar` is referenced, we can simply change it to a property:

    class Foo(object):
        @property
        def bar(self):
            return False
    >>> f = Foo()
    >>> f.bar
    False

Then we don't have to change all the callers!
