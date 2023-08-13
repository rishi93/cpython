# Add the `until` loop to Python

## What is it?
It's essentially the opposite of the while loop.
Ruby has it.

```python
i = 5

until i == 0:
	print("Hello")
	i -= 1

# It's the same as
while not i == 0:
    print("Hello")
    i -= 1
```

## How to add it?
Just follow the implementation of the `while_stmt` in the Python grammar, and
try to find a way to reverse it's effect.

Inspired by Eli Bendersky's [blog article](https://eli.thegreenplace.net/2010/06/30/python-internals-adding-a-new-statement-to-python)