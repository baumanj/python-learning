# Thursday's Python learning schedule

* Before 10am: unstructured time/Q&A
* 10–10:15: Lecture: Practical Python knowledge
* 10:15–11:15: A brief coding project
* 11:15–11:30: Coffee break/Question time
* 11:30–12: A longer coding project: [FarMarFinder]2: The Revenge
* 12–1: Lunch
* 1-1:15: Reading some real-life python code
* 1:15–2: Project time
* 2–2:15: Break
* 2:15–3: Project time
* 3–5: Students pick

[FarMarFinder]: https://github.com/Ada-Developers-Academy/far_mar_finder

## Practical Python knowledge

I've written Python here and there since 2003, but it's never been my primary language for my full-time gig. As such, I have focused mostly on the features that I've needed to get the work done. Hopefully these same things will help you get on your feet as quickly as possible in your internships. Here are the things that I've found most useful to know:

* How to read/debug Python
* Where to look for Python information
* The standard library functionality that you should be aware of

### How to read/debug Python

There's nothing magic about reading Python; you read it like any other programming language. The trickiest part is knowing where to start. For Python, once you've chosen a file to read, it's a question of finding your entry point. If it's a script that's run directly (rather than imported), look for the `if __name__ == '__main__'` bit and follow that. If it's a unit test suite (as we'll look at later today), first look at the classes that derive from `unittest.TestCase`, as that will be the broad organization of the test and then look at the methods whose names start with `test` for one to dig in to. Try to mentally read through the flow of the code and get a general sense for it. After you've done that, it's often a good idea to run the code in a debugger.

There are lots of options for python debuggers and a [good summary here](http://blog.ionelmc.ro/2013/06/05/python-debugging-tools/) of various tools, but I usually just end up using pdb. It's part of the standard library, so it's always available and it's the first hit when you google "python debugging", so it's easy to remember. There are two ways to get into pdb. If the place you want to stop and check out is deep in the guts of the program and you can edit the code, it's very simple to add the line:

    import pdb; pdb.set_trace()
 
right where you want it to stop. Otherwise, if you want to see the flow from the beginning, or you can't edit the code easily, you can just execute your script and add a `-m pdb` argument to the python interpreter like so:

    python -m pdb myscript.py
 
Then, once you are in pdb you'll get a happy little `(pdb)` prompt from which you can look at the code line by line and examine the values of variables (or even change them). 95% of these 6 commands:

1. l [number]: **l**ist the code around line _number_ or continue from last list command
2. n: go to the **n**ext line
3. s: **s**tep into the function if the next line is a function call
4. b [number or name]: **b**reak (i.e., stop) at line _number_ or function _name_
5. p _expression_: **p**rint the value of the expression
6. c: **c**ontinue until the next breakpoint or the program stops naturally

Basically I work like this:
1. **L**ist the source to find the line number I'm curious about.
2. Set a **b**reakpoint there and **c**ontinue until I hit it.
3. **P**rint out some variables to see what the state of the program is.
4. **S**tep through some code (or **n**ext over function calls I don't care about) and see what the flow looks like.
5. Repeat until I've figured out whatever it is I want to know.

### Where to look for Python information

Most often, when I need to know how to use a bit of the standard library, I'm already in the `python` interpreter. Happily, it makes available a handy on-line help system, that you can invoke just by typing `help()`. With no arguments, it takes you to an interactive system to find the topic you're looking for. If you know what you want information about, you can pass help an object and it will give you the documentation for that object. You'll need to already have `import`ed it if it's part of a moule. In general though, you don't even need to know what the object is to find the help. For example:

    a = list()
    help(a)
  
Will give you the docs for the `list` class. This is a bit contrived, but it's handy if you're in the middle of debugging to just use a variable that you want to know more about what you can do with it. You can also type `help([])` or `help(list)`. One set of docs that is not so obvous how to access is the help for all the built-in functions (e.g., `len()`, `sorted()`, `enumerate()`). You can access that with the rather confusing `help(__builtins__)`. You can also pass entire module names:

    import random
    help(random)

That is something I type often because I can never remember the name of random.randrange. When you invoke `help()`, it will take you to a viewer powered by the `less` command. Type `h` for help on navigating within `less`. The most useful thing to know is that you can search by typing `/foo` to find occurrences of "foo" and then type `n` to go to the next one. If I happen to know what method I'm interested in, I can type that directly. I often type `help(random)`

For more detailed information docs.python.org is the place to go. One thing to be careful of is what version of documentation you're looking at. At isilon, we use 2.6.1. Your computers likely have 2.7.x installed.

Of course, don't forget your old friend Stack Overflow for answers to specicifc questions.

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

* `string`: Contains handy constants that lets you do things like `x in string.letters`.
* `re`: Regular expressions
* `pprint`: Pretty-printing; great for dicts and such.
* `random`: Generate random numbers, select random samples, etc.
* `fileinput`: A handy way to write a script that reads from file arguments or standard input.
* `pickle`: Save objects to a file and read them back automagically.
* `csv`: Access comma-separated value files.
* `popen2`: For running shell commands.

### Random handy things

#### Reload your code changes without restarting Python

When I'm writing Python code, I'm usually playing around in the interactive Python interpreter. A really handy way to try out your code is to launch `python` in the same directory as your `myscript.py` file, then type:

    import myscript

Then try out some of your code interactively by running functions like `myscript.foo`.

Then make some edits and you can just run:

    reload(myscript)

#### Turn an attribute into a property

In ruby, if you have an object instance `foo` with a simple attribute called `bar`, it's easy to later make the behavior more complex by adding `def bar` and or `def bar=` methods to your class. In python, if you change an attribute to a method, all the places in the code that access `foo.bar` will now get a method object. Compare:

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
  