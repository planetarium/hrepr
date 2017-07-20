
# hrepr

`hrepr` is a package to create an HTML representation for Python objects. It can output to Jupyter Notebooks and it can also generate standalone pages with the representation of choice objects.

## Install

`hrepr` requires at least Python 3.6

```bash
$ pip3 install hrepr
```

## Usage

```python
from hrepr import hrepr
obj = {'potatoes': [1, 2, 3], 'bananas': {'cantaloups': 8}}

# Print the HTML representation of obj
print(hrepr(obj))

# Wrap the representation in <html><body> tags and embed the default
# css style files to produce a standalone page.
print(hrepr(obj).as_page())
```

In a Jupyter Notebook, return `hrepr(obj)` directly.

## Custom representations

A custom representation for an object can be defined using the `__hrepr__` method on the object, and if necessary, an `__hrepr_resources__` method on the class. No dependency on `hrepr` is necessary:

```python
class RedGreen:
    def __init__(self, r, g):
        self.r = r
        self.g = g

    @classmethod
    def __hrepr_resources__(cls, H):
        return H.style('''
        .red { background-color: red; }
        .green { background-color: green; }
        ''')

    def __hrepr__(self, H, hrepr):
        return H.div(
            H.div['red'](hrepr(self.r)),
            H.div['green'](hrepr(self.g))
        )
```

`__hrepr__` receives two arguments:

* `H` is the HTML builder, which has the simple interface: `H.tag['klass', ...](child, ..., attr=value, ...)` to create the tag `<tag class=klass attr=value>child</tag>`. You can add more classes, attributes and children, i.e. it is legal to write `H.a['cls1'](href='blah')['cls2']('hello', attr=value)`
    - Use `H.raw(string)` to insert an unescaped string (e.g. literal HTML)
    - Use `H.inline(...)` to concatenate the children without wrapping them in a tag.
* `hrepr` is a function that can be called recursively to get the representation of the object's fields.

`__hrepr_resources__` is optional, but if it is defined, it should return one or a list of tags that should be inserted in the `<head>` section of the page in order to properly format the representation. These can be `<style>` tags, `<script>` tags, or `<link>` tags.

## Custom hrepr

You can also customize the `hrepr` function by subclassing the `HRepr` or `StdHRepr` classes (the difference between the two is that the latter defines default representations for several Python types like `list` or `dict` whereas the former does not).

Your subclass can override the following functions:

* **`global_resources`** should return one or a list of tags to insert in `<head>`.
* **`repr_<type>`** should return the representation for objects with classes named `<type>`. There are no `H` and `hrepr` parameters, but you can access them as `self.H` and `self` respectively.
* **`__call__`** is the main representation function, and will be called recursively for every object to represent.

```python
from hrepr import StdHRepr

class MyRepr(StdHRepr):
    def global_resources(self, H):
        return H.style(".my-integer { color: fuchsia; }")

    def repr_int(self, n):
        return self.H.span['my-integer'](str(n + 1))

myrepr = MyRepr()
print(myrepr(10)) # <span class="my-integer">11</span>
print(myrepr.hrepr_with_resources(10).as_page()) # This will include the style
```

