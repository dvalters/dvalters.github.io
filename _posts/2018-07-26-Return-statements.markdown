---
title: Why do Python functions return None?
layout: post
tags: python
---

I was talking to a colleague recently about what would be covered in a basic 'introduction to python' tutorial. We got to talking about lists and how you would explain how to add things to lists with the `append()` method.

We had this code that turned out to be wrong because we'd forgotten that `list.append()` does not actually return a list, it merely updates the list object in place. (I.e. there is no `return` value from the function. Other programming languages might call this a `void function` or a procedure.

What we tried to do was:

```python
l = ['one', 'two', 'three']
print(l.append('four'))
```

And to our (initial) surprise, got:

```
None
```

Instead we should have called the method first (updating the list, l) and then called `print(l)` to see the updated list.

Inspecting our mistake, we investigated what the type of `l.append(item)` was:

```python
type(l.append("four")
# NoneType
```

Yep, definitely `None` it seems! But why is it `None`? If it has no return value, why does it have any type at all, why not raise an error? 

What about other functions with no return statement, we thought...

```python
def voidfunc(a, b):
  x = a + b

type(voidfunc(1, 2)
# NoneType (?!)

z = voidfunc(1, 2)
print(z)
# None
```

This last one is slightly confusing, because you might intuitively expect an error perhaps (how could you assign a value to a variable from a function with no return statement?)

In fact, it turned out from some googling that Python functions have a default return value, which is `None` if no return expression is given, or `return` is given on its own.

For example, the source code for the `append()` method of `list` is this:

In the CPython source, this is in 

#### Objects/listobject.c

```c
static PyObject *
list_append(PyListObject *self, PyObject *object)
{
    if (app1(self, object) == 0)
        Py_RETURN_NONE;
    return NULL;
}
```

It actually calls another function `app1()` which does the actual appending, which returns 0 on success. The next line `Py_RETURN_NONE` gives us a hint to what happens in the append method. 

The builtins are a little confusing, so I'll come back to them another time, but here is the code which determines what happens in functions that we write ourselves:

#### Python/compile.c

```c
/* Make sure every block that falls off the end returns None.
   XXX NEXT_BLOCK() isn't quite right, because if the last
   block ends with a jump or return b_next shouldn't set.
 */
if (!c->u->u_curblock->b_return) {
    NEXT_BLOCK(c);
    if (addNone)
        ADDOP_LOAD_CONST(c, Py_None);
    ADDOP(c, RETURN_VALUE);
}
```

Without going in to the details of this code, there is a conditional statement that makes sure to "add None" if there is no return value given. (Py_None is the Python `None` object, see https://docs.python.org/2/c-api/none.html)






