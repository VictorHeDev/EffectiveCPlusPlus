# Introduction

## Terminology
`declaration` - tells compilers about the name and type of something, but it omits certain details. Examples of declarations:
```cpp
extern int x            // object declaration
std::size_t numDigits(int number);  // function declaration
class Widget;           // class declaration
template<typename T>    // template declaration
class GraphNOde;
```
Each function's declaration reveals its __signature__, aka its parameter and return types. A function's signature is the same as its type.
`definition` - provides compilers with the details a declaration omits. For an object, the definition is where compilers set aside memory for the object. FOr a function or function template, the definition provides the code body. FOr a class or class template, the definition lists the members of the class or template:
```cpp
int x;                  // object definition
std::size_t numDigits(int number); // function definition
{                       // this function returns
  std::size_t digitsSoFar = 1;     // the number of digits in its parameter
  while ((number /= 10) != 0) ++digitsSoFar;
  return digitsSoFar;
}
class Widget {          // class definition
  public:
    Widget();
    ~Widget();
  // ...
}

template<typename T>    // template definition
class GraphNode{
  public:
    GraphNode();
    ~GraphNode();
    // ...
}

```
`Initialization` - is the process of giving an object its first value. For objects of user-defined types, initialization is performed by constructors.
```cpp
class A {
  public:
    A();    // default constructor which is called with NO arguments
};

class B {
  public:
    explicit B(int x = 0, bool b = true); // default constructor
};

class C {
  public:
    explicit C(int x);    // not a default constructor
}
```

Calling a constructor with the keyword `explicit` prevents them from being used to perform implicit type conversions, though they can still be used for explicit type conversions. Constructors declared `explicit` are usually preferable to non-explicit ones, because they prevent compilers from performing unexpected (often unintended) type conversions. Encouraged to declare `explicit` whenever possible.

`Copy constructor` is used to initialize an object with a different object of the same type. Defines how an object is passed by value. "Pass-by-value" means "call the copy constructor." It is generally a bad idea to pass user-defined types by value. Pass-by-reference-to-const is typically a better choice for user-defined types.
`Copy assignment operator` is used to copy the value from one object to another of the same type.

```cpp
class Widget {
  public:
    Widget();       // default constructor
    Widget(const Widget& rhs);  // copy constructor
    Widget& operator=(const Widget& rhs); // copy assignment operator
    // ...
};

Widget w1;       // invoke default constructor
Widget w2(w1);    // invoke copy constructor
w1 = w2;          // invoke copy assignment operator

Widget w3 = w2;   // here the "=" syntax is used to call the copy constructor ... beware!!!
```
How to distinguish `copy construction` from `copy assignment`: if a new object is being defined (like `w3` above), a constructor has to be called; it is NOT assignment. If no new object is being defined (like `w1 = w2` above), no constructor can be involved, so it is assignment.

`STL` - __Standard Template Library__, is devoted to containers, iterators, algorithms, and more.
`ctor` - constructors.
`dtor` - destructors.

## Naming Conventions
Parameters:
* `lhs` - left hand side. For member functions, the lhs argument is represented by the `this` pointer so sometimes `rhs` is used by itself.
* `rhs` - right hand side.
* `pt` - "pointer to type T"
* `mf` - member functions
