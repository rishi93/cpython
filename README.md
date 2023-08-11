# Adding a pipe operator to Python

## What is the pipe operator?
It looks like this: `|>`

Suppose we had the following piece of code:
```python
def double(x):
    return 2*x

# Normally we do this
double(1)
# Or even this
double(double(1))

# With the pipe operator we can do this:
1 |> double
1 |> double |> double
```

The pipe operator really shines when you need to apply a chain of functions
to something

## How to go about it?
* We need to introduce the new token

Go to the `Grammar/Tokens` file and add this new token
```
VBARGREATER             '|>'
```

Now run this: `make regen-token`


If you've read the "Crafting Interpreters" book, you know the usual workflow of
a scripting language:

Source code -> Tokenizer -> Parser -> AST 

* We need to add a new grammer rule
First, a detour:

How would we describe the grammar rule for `1 + 2`
```
sum:
    atom + atom
    atom

atom:
    NUMBER
```
So the above grammar can match `1` or `1 + 2`. But what about `1 + 2 + 3`?

We make the `sum` rule recursive
```
sum:
    sum + atom
    atom

atom:
    NUMBER
```
Now we can match `1 + 2 + 3` and also `1 + 2 + 3 + 4` and so on...

This is pretty similar to what we want with the pipe operator as well, we want to
be able to pipe as many times as possible.

We make the following modification to the `Grammar/python.gram` file
```
shift_expr[expr_ty]:
    | a=shift_expr '<<' b=pipe { _PyAST_BinOp(a, LShift, b, EXTRA) }
    | a=shift_expr '>>' b=pipe { _PyAST_BinOp(a, RShift, b, EXTRA) }
    | pipe

# Arithmetic operators
# --------------------
pipe[expr_ty]:
    | a=pipe '|>' b=sum { _PyAST_BinOp(a, CallPipe, b, EXTRA) }
    | sum

sum[expr_ty]:
    | a=sum '+' b=term { _PyAST_BinOp(a, Add, b, EXTRA) }
    | a=sum '-' b=term { _PyAST_BinOp(a, Sub, b, EXTRA) }
    | term
```

Make the following modification to the `Parser/Python.asdl` file
```
    operator = Add | Sub | Mult | MatMult | Div | Mod | Pow | LShift
                 | RShift | BitOr | BitXor | BitAnd | FloorDiv | CallPipe
```

And then run the command: `make regen-ast`
And then run the command: `make regen-pegen`

## Execute our new operation
Go to this file: `Lib/opcode.py`

Add this snippet:
```
def_op('POP_EXCEPT', 89)

def_op('BINARY_PIPE_CALL', 90)

HAVE_ARGUMENT = 91             # real opcodes from here have an argument:
```

## Making the compiler use the new "opcode"
Go to this file: `Python/compile.c`
```c
static int compiler_visit_expr1(struct compiler *c, expr_ty e) {
    switch (e->kind) {
        case BinOp_kind:
            VISIT(c, expr, e->v.BinOp.left);
            VISIT(c, expr, e->v.BinOp.right);
            ADDOP(c, binop(e->v.BinOp.op));
            break;
    }
}
```