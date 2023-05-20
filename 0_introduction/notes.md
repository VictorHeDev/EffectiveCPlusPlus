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