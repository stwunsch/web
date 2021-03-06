---
title: "Python interface: PyROOT"
layout: single
sidebar:
  nav: "manual"
toc: true
toc_sticky: true
---

ROOT can be used from Python thanks to its Python-C++ bindings, called **PyROOT**. PyROOT is HEP's entrance to all C++ from Python, e.g. for frameworks and their steering code. The PyROOT bindings are *automatic* and *dynamic*: no pre-generation of Python wrappers is necessary.

With PyROOT, all the ROOT functionality can be accessed from Python while, at the same time, benefiting from the performance of the ROOT C++ libraries.

PyROOT is compatible with both Python2 (>= 2.7) and Python3.

> Using PyROOT requires working knowledge of Python. Detailed information about the Python language can be found in the [Python Language Reference](https://docs.python.org/3/reference/){:target="_blank"}.

Together with ROOT 6.22, a major revision of PyROOT has been released. This **new PyROOT** has extensive support for *modern C++* (it operates on top of [cppyy](https://cppyy.readthedocs.io/)), is more *pythonic* and is able to *interoperate* with widely-used Python data-science libraries (e.g. [NumPy](https://numpy.org/), [pandas](https://pandas.pydata.org/), [Numba](https://numba.pydata.org/)). Therefore, we strongly recommend to move to the new PyROOT!

## Building and installing
PyROOT is enabled by default when building and installing ROOT. Please refer to [Building ROOT from source]({{ '/install/build_from_source' | relative_url }}) for specific information on how PyROOT can be built, and to [Installing ROOT]({{ '/install' | relative_url }}) to learn about the ways to install ROOT.

## Getting started

Once ROOT has been installed, PyROOT can be used both from the Python prompt and from a Python script. The entry point to PyROOT is the `ROOT` module, which needs to be imported first: 

```python
import ROOT
```

After that, all the ROOT C++ functionality (classes, functions, etc.) can be accessed via the `ROOT` module. In the example below, we show how to create a histogram (an instance of the `TH1F` class) and randomly fill it with a gaussian distribution.

```python
h = ROOT.TH1F("myHist", "myTitle", 64, -4, 4)
h.FillRandom("gaus")
```

### Tutorials

There are a number of tutorials that show how to use the various ROOT features from Python. They can be found in the link below. 

{% include tutorials name="PyROOT" url="pyroot" %}

## Interactive graphics

Just like from C++, it is also possible to create interactive ROOT graphics from Python thanks to PyROOT. As an example, let's consider the following code, which creates a one-dimensional function and draws it:

```python
>>> import ROOT
>>> f = ROOT.TF1("f1", "sin(x)/x", 0., 10.)
>>> f.Draw()
```

As a result of running the code above from the Python prompt, a new canvas will open up to show the drawn function:

{% include figure_image
img="tf1_draw.png"
caption="Example of graphics generated with PyROOT."
%}

> **Note that** the code above can also be written in a script file and executed with Python. In that case, the script will run to completion and the Python process will terminate, making the created canvas disappear. If you want to keep the Python process alive and thus be able to inspect your canvas, run the script with `python -i script_name.py`.

## Interface

*Content is coming soon!*

## Loading user libraries & jitting

*Content is coming soon!*

## IPython: running Python from C++

ROOT also allows to run Python code from C++ via the `TPython` class.

The example below shows how to use `Exec` to run a Python statement, `Eval` to evaluate a Python expression and get its result back in C++ and `Prompt` to start an interactive Python session. 

```cpp
root [0] TPython::Exec( "print(1 + 1)" )
2
root [1] auto b = (TBrowser*)TPython::Eval( "ROOT.TBrowser()" )
(TBrowser *) @0x7ffec81094f8
root [2] TPython::Prompt()
>>> i = 2
>>> print(i)
2
```

For a more complete description of the `TPython` interface, please check the reference guide for {% include ref class="TPython" %}.

## JupyROOT: PyROOT for Jupyter notebooks

*Content is coming soon!*

## New PyROOT: Backwards-Incompatible Changes

The new PyROOT has some backwards-incompatible changes with respect to its predecessor, which are listed next:
- Instantiation of function templates must be done using square brackets instead of parentheses. For example, if we consider the following
  code snippet:

```python
>>> import ROOT

>>> ROOT.gInterpreter.Declare("""
template<typename T> T foo(T arg) { return arg; }
""")

>>> ROOT.foo['int']  # instantiation
cppyy template proxy (internal)

>>> ROOT.foo('int')  # call (auto-instantiation from arg type)
'int'
```

Note that the above does not affect class templates, which can be instantiated either with parenthesis or square brackets:

```python
>>> ROOT.std.vector['int'] # instantiation
<class cppyy.gbl.std.vector<int> at 0x5528378>

>>> ROOT.std.vector('int') # instantiation
<class cppyy.gbl.std.vector<int> at 0x5528378>
```

- Overload resolution in new cppyy has been significantly rewritten, which sometimes can lead to a different overload choice
(but still a compatible one!). For example, for the following overloads of `std::string`:

```cpp
string (const char* s, size_t n)                           (1)
string (const string& str, size_t pos, size_t len = npos)  (2)
```

when invoking `ROOT.std.string(s, len(s))`, where `s` is a Python string, the new PyROOT will pick (2) whereas the old
would pick (1).

- The conversion between `None` and C++ pointer types is not allowed anymore. Instead, `ROOT.nullptr` should be used:

```python
>>> ROOT.gInterpreter.Declare("""
class A {};
void foo(A* a) {}
""")

>>> ROOT.foo(ROOT.nullptr)  # ok

>>> ROOT.foo(None)          # fails
TypeError: void ::foo(A* a) =>
TypeError: could not convert argument 1
```

- Old PyROOT has `ROOT.Long` and `ROOT.Double` to pass integer and floating point numbers by reference. In the new
PyROOT, `ctypes` must be used instead.

```python
>>> ROOT.gInterpreter.Declare("""
void foo(int& i) { ++i; }
void foo(double& d) { ++d; }
""")

>>> import ctypes

>>> i = ctypes.c_int(1)
>>> d = ctypes.c_double(1.)

>>> ROOT.foo(i); i
c_int(2)

>>> ROOT.foo(d); d
c_double(2.0)
```

- When a character array is converted to a Python string, the new PyROOT only considers the characters before the
end-of-string character:

```python
>>> ROOT.gInterpreter.Declare('char MyWord[] = "Hello";')

>>> mw = ROOT.MyWord

>>> type(mw)
<class 'str'>

>>> mw  # '\x00' is not part of the string
'Hello'
```

- Any Python class derived from a base C++ class now requires the base class to define a virtual destructor:

```python
>>> ROOT.gInterpreter.Declare("class CppBase {};")
 True
>>> class PyDerived(ROOT.CppBase): pass
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
TypeError: CppBase not an acceptable base: no virtual destructor
```

- There are several name changes for what concerns cppyy APIs and proxy object attributes, including:

| Old PyROOT/cppyy                          | New PyROOT/cppyy                |
|-------------------------------------------|---------------------------------|
| cppyy.gbl.MakeNullPointer(klass)          | cppyy.bind\_object(0, klass)    |
| cppyy.gbl.BindObject / cppyy.bind\_object | cppyy.bind\_object              |
| cppyy.AsCObject                           | libcppyy.as\_cobject            |
| cppyy.add\_pythonization                  | cppyy.py.add\_pythonization     |
| cppyy.compose\_method                     | cppyy.py.compose\_method        |
| cppyy.make\_interface                     | cppyy.py.pin\_type              |
| cppyy.gbl.nullptr                         | cppyy.nullptr                   |
| cppyy.gbl.PyROOT.TPyException             | cppyy.gbl.CPyCppyy.TPyException |
| buffer.SetSize(N)                         | buffer.reshape((N,))            |
| obj.\_\_cppname\_\_                       | obj.\_\_cpp\_name\_\_           |
| obj.\_get\_smart\_ptr                     | obj.\_\_smartptr\_\_            |
| callable.\_creates                        | callable.\_\_creates\_\_        |
| callable.\_mempolicy                      | callable.\_\_mempolicy\_\_      |
| callable.\_threaded                       | callable.\_\_release\_gil\_\_   |

- New PyROOT does not parse command line arguments by default anymore. For example, when running
`python my_script.py -b`, the `-b` argument will not be parsed by PyROOT, and therefore the
batch mode will not be activated. If the user wants to enable the PyROOT argument parsing again,
they can do so by starting their Python script with:

```python
import ROOT
ROOT.PyConfig.IgnoreCommandLineOptions = False
```

- In new PyROOT, `addressof` should be used to retrieve the address of fields in a `struct`,
for example:

```python
>>> ROOT.gInterpreter.Declare("""
struct MyStruct {
  int a;
  float b;
};
""")

>>> s = ROOT.MyStruct()

>>> ROOT.addressof(s, 'a')
94015521402096L

>>> ROOT.addressof(s, 'b')
94015521402100L

>>> ROOT.addressof(s)
94015521402096L
```

In old PyROOT, `AddressOf` could be used for that purpose too, but its behaviour was inconsistent.
`AddressOf(o)` returned a buffer whose first position contained the address of object `o`, but
`Address(o, 'field')` returned a buffer whose address was the address of the field, instead
of such address being contained in the first position of the buffer.
Note that, in the new PyROOT, `AddressOf(o)` can still be invoked, and it still returns a buffer
whose first position contains the address of object `o`.

- In old PyROOT, there were two C++ classes called `TPyMultiGenFunction` and `TPyMultiGradFunction`
which inherited from `ROOT::Math::IMultiGenFunction` and `ROOT::Math::IMultiGradFunction`, respectively.
The purpose of these classes was to serve as a base class for Python classes that wanted to inherit
from the ROOT::Math classes. This allowed to define Python functions that could be used for fitting
in Fit::Fitter.
In the new PyROOT, `TPyMultiGenFunction` and `TPyMultiGradFunction` do not exist anymore, since their
functionality is automatically provided by new cppyy: when a Python class inherits from a C++ class,
a wrapper C++ class is automatically generated. That wrapper class will redirect any call from C++
to the methods implemented by the Python class. Therefore, the user can make their Python function
classes inherit directly from the ROOT::Math C++ classes, for example:

```python
import ROOT

from array import array

class MyMultiGenFCN( ROOT.Math.IMultiGenFunction ):
    def NDim( self ):
        return 1

    def DoEval( self, x ):
        return (x[0] - 42) * (x[0] - 42)

    def Clone( self ):
        x = MyMultiGenFCN()
        ROOT.SetOwnership(x, False)
        return x

def main():
    fitter = ROOT.Fit.Fitter()
    myMultiGenFCN = MyMultiGenFCN()
    params = array('d', [1.])
    fitter.FitFCN(myMultiGenFCN, params)
    fitter.Result().Print(ROOT.std.cout, True)

if __name__ == '__main__':
    main()
```

- Inheritance of Python classes from C++ classes is not working in some cases. This is described
in [ROOT-10789](https://sft.its.cern.ch/jira/browse/ROOT-10789) and
[ROOT-10582](https://sft.its.cern.ch/jira/browse/ROOT-10582).
This affects the creation of GUIs from Python, e.g. in the
[Python GUI tutorial](https://root.cern/doc/master/gui__ex_8py.html), where the inheritance
from `TGMainFrame` is not working at the moment. Future releases of ROOT will fix these
issues and provide a way to program GUIs from Python, including a replacement for TPyDispatcher,
which is no longer provided.

- When iterating over an `std::vector<std::string>` from Python, the elements returned by
the iterator are no longer of type Python `str`, but `cppyy.gbl.std.string`. This is an
optimization to make the iteration faster (copies are avoided) and it allows to call
modifier methods on the `std::string` objects.

```python
>>> import cppyy

>>> cppyy.cppdef('std::vector<std::string> foo() { return std::vector<std::string>{"foo","bar"};}')

>>> v = cppyy.gbl.foo()

>>> type(v)
<class cppyy.gbl.std.vector<string> at 0x4ad8220>

>>> for s in v:
...   print(type(s))  # s is no longer a Python string, but an std::string
...
<class cppyy.gbl.std.string at 0x4cd41b0>
<class cppyy.gbl.std.string at 0x4cd41b0>
```
