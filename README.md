# Adding a new keyword `proceed` to Python
What does this keyword do?
It's exactly the same as the `pass` keyword.

## Why?
Just to get our feet wet with modifying the CPython source code.

Just go to the `Grammar/python.gram` file

Find the `small_stmt` section, and add in the `proceed` keyword right
next to the `pass` keyword.
```
small_stmt[expr_ty]
| ('pass' | 'proceed') { _Py_Pass(EXTRA) }
```

* Run `make regen-pegen` to regenerate the parser.
* Then compile Cpython again.

Now you can do something like this:
```python
def do_nothing():
    proceed
```