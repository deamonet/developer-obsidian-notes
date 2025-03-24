这个问题的提问者跟我一样把IF ELSE的放在最后了，错成一样了。



# [if/else in a list comprehension](https://stackoverflow.com/questions/4260280/if-else-in-a-list-comprehension)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 14 years ago

Modified [6 months ago](https://stackoverflow.com/questions/4260280/if-else-in-a-list-comprehension?lastactivity "2024-05-09 21:42:32Z")

Viewed 2.1m times

1718

[](https://stackoverflow.com/posts/4260280/timeline)

How do I convert the following `for`-loop containing an `if`/`else` into a list comprehension?

```python
results = []
for x in xs:
    results.append(f(x) if x is not None else '')
```

It should yield `''` if `x` is `None`, and otherwise `f(x)`. I tried:

```python
[f(x) for x in xs if x is not None else '']
```

but it gives a `SyntaxError`. What is the correct syntax?

---

See [Does Python have a ternary conditional operator?](https://stackoverflow.com/questions/394809/) for info on `... if ... else ...`.  
See [List comprehension with condition](https://stackoverflow.com/questions/24442091) for _omitting_ values based on a condition: `[... for x in xs if x cond]`.  
See [`elif` in list comprehension conditionals](https://stackoverflow.com/questions/9987483) for `elif`.

- [python](https://stackoverflow.com/questions/tagged/python "show questions tagged 'python'")
- [list](https://stackoverflow.com/questions/tagged/list "show questions tagged 'list'")
- [list-comprehension](https://stackoverflow.com/questions/tagged/list-comprehension "show questions tagged 'list-comprehension'")

[Share](https://stackoverflow.com/q/4260280 "Short permalink to this question")

[Improve this question](https://stackoverflow.com/posts/4260280/edit)

Follow

[edited Dec 9, 2023 at 23:34](https://stackoverflow.com/posts/4260280/revisions "show all edits to this post")

[

![Mateen Ulhaq's user avatar](https://i.sstatic.net/4cFxJ9Lj.jpg?s=64)

](https://stackoverflow.com/users/365102/mateen-ulhaq)

[Mateen Ulhaq](https://stackoverflow.com/users/365102/mateen-ulhaq)

27k2121 gold badges117117 silver badges152152 bronze badges

asked Nov 23, 2010 at 19:56

[

![AP257's user avatar](https://www.gravatar.com/avatar/8c8bbcdad511dd10864846ac30af1b06?s=64&d=identicon&r=PG)

](https://stackoverflow.com/users/267831/ap257)

[AP257](https://stackoverflow.com/users/267831/ap257)

93.5k8989 gold badges206206 silver badges263263 bronze badges

- The way the question is written, I'd argue that the correct answer would be `[f(x if x is not None else '') for x in xs]`. 
    
    – [Pixelchemist](https://stackoverflow.com/users/951423/pixelchemist "24,866 reputation")
    
     [CommentedDec 23, 2022 at 12:59](https://stackoverflow.com/questions/4260280/if-else-in-a-list-comprehension#comment132179142_4260280)
    

[Add a comment](https://stackoverflow.com/questions/4260280/if-else-in-a-list-comprehension# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 13 Answers

Sorted by:

                                              Highest score (default)                                                                   Trending (recent votes count more)                                                                   Date modified (newest first)                                                                   Date created (oldest first)                              

2875

[](https://stackoverflow.com/posts/4260304/timeline)

You can totally do that. It's just an ordering issue:

```python
[f(x) if x is not None else '' for x in xs]
```

In general,

```python
[f(x) if condition else g(x) for x in sequence]
```

And, for list comprehensions with `if` conditions only,

```python
[f(x) for x in sequence if condition]
```

Note that this actually uses a different language construct, a [conditional expression](https://docs.python.org/3/reference/expressions.html#conditional-expressions), which itself is not part of the [comprehension syntax](https://docs.python.org/3/reference/expressions.html#displays-for-lists-sets-and-dictionaries), while the `if` after the `for…in` is part of list comprehensions and used to _filter_ elements from the source iterable.

---

Conditional expressions can be used in all kinds of situations where you want to choose between two expression values based on some condition. This does the same as the [ternary operator `?:` that exists in other languages](https://docs.python.org/3/faq/programming.html#is-there-an-equivalent-of-c-s-ternary-operator). For example:

```python
value = 123
print(value, 'is', 'even' if value % 2 == 0 else 'odd')
```