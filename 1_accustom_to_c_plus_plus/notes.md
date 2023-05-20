# Accustoming Yourself to C++

## Item 1: View C++ as a federation of languages

1. C
2. Object-Oriented C++
3. Template C++
4. STL

Prepare to code-switch and change strategies when switching from one sublanguage to another. For example, pass-by-value is generally more efficient than pass-by-reference for built-in types. But when you move from the C part of C++ to Object-Oriented C++, the existence of user-defined constructors and destructors means that pass-by-reference-to-const is usually better. This is especially the case when working in Template C++ because often times you don't even know the type of object you're dealing with. In STL, iterators and function objects are modeled on pointers in C, so the old C pass-by-value rule (more efficient) applies again.

## Item 2: Prefer `const`s, `enum`s, and `inline`s to `#define`s

"Prefer the compiler to the preprocessor," because `#define` might be removed before compile time. When replacing `#define`s with constants, 2 special cases are worth mentioning.

1. Defining constant pointers - because constant definitions are typically put into header files (where many source files can include and use them), it's important that the **pointer** be declared **const**, usually in addition to what the pointer points to. Generally string objects are generally preferable to their char\* progenitors, so the bottom example is more preferred.

```cpp
const char* const authorName="Scott Meyers";
const std::string authorName("Scott Meyers"); // preferred
```

2. Class-specific constants - to limit the scop eof a constant to a class, you must make it a member. To ensure there's at most one copy of the constant, make it a **static** member. The enum hack is worth knowing because the enum hack is a fundamental technique of template programming.

```cpp
class GamePlayer {
  private:
    static const int NumTurns = 5;    // constant declaration (not definition)
    int scores[NumTurns];             // use of constant
  // ...
};

// an older version of static constant below
class CostEstimate {
  private:
    static const double FudgeFactor;    // declaration of static class constant; goes in header file
};

const double CostEstimate::FudgeFactor = 1.35; // definition of a static class constant; goes in the impl. file

// below also works and is called the "enum hack" for taking advantage of the fact that values of enumerated type can be used where ints are expected
class GamePlayer {
  private:
    enum { NumTurns = 5 };    // "enum hack" makes NumTurns a symbolic name for 5
    int scores[NumTurns];             // this is fien
  // ...
};

```

## Item 3: Use `const` whenever possible

Declaring something `const` will enlist the compiler's help to make sure the constraint is not violated. OUtside of classes, you can use it for constants at global, or namespace scope, objects declared `static` at file, function, or block scope. Inside classes, you can use it for both static and non-static data members. For pointers, you can specify whether the pointer itself is `const`, if the data it points to is `const`, both, or neither.

If the word `const` appears to the left of the asterisk, what it's **pointed to** is const; if the word `const` appears tot he right of the asterisk, the **pointer** itself is const; if `const` appears on both sides, both are constant.

```cpp
void f1(const Widget *pw); // f1 takes a pointer to a constant Widget object
void f2(Widget const *pw); // so does f2
```

Within a function declaration, `const` can refer to the function's return value, to individual parameters, and for member functions, to the function as a whole.

### `const` Member functions

1. They make the interface of a class easier to understand
2. They make it possible to work with `const` objects

One of the fundamental ways to improve a C++ program's performance is to pass objects by reference-to-const. What does it mean for a member function to be `const`? There are 2 prevailing notions: physical correctness vs. logical correctness.

- physical constness - believes a member function is const if and only if it doesn't modify any of the object's data members (excluding those that are static) aka if it doesn't modify any of the bits inside the object. It's easy to detect violations; the compilers just look for assignments to data members.

## Item 4: Make sure that objects are initialized before they're used

Reading uninitialized values yields undefined behavior. The best way to deal with this is to **always** initialize your objects before you use them. For non-member objects of built-in types, you'll need to do this manually.

```cpp
int x = 0;    // manual initialization of an int
const char* text = "A C-style string";  // manual initialization of a pointer
double d;     // "initialization" by reading from an input stream
std::cin >> d;
```

For almost everything else, the responsibility for initialization falls on the constructors. Make sure all constructors initialize everything in the object. An important distinction is that assignment != initialization.

```cpp
class PhoneNumber {
  // ...
};

class ABEntry { // ABEntry = "Address Book Entry"
  public:
    ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>&phones);
  private:
    std::string theName;
    std::string theAddress;
    std::list<PhoneNumber> thePhones;
    int numTimesConsulted;
};

ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones) {
  theName = name;   // these are all assignments, not initializations
  theAddress = address;
  thePhones = phones;
  numTimesConsulted = 0;
}

ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones)
:
  theName(name),   // these are all initializations
  theAddress(address),
  thePhones(phones),
  numTimesConsulted(0)
{
  // ctor (constructor) body is now empty (good!)
}
```

Ordering for initializing members in the initialization list MATTERS.

#### Order of initialization of non-local static objects defined in different translation units

`static object` - one that exists from the time it's constructed until the end of hte program. Static objects inside functions are known as **local static objects** because they're local to a function. The other kids of static objects are known as **non-local static objects**. Static objects are automatically destroyed when the program exits, i.e., their destructors are automatically called when `main` finishes executing.
`translation unit` - is the source code giving rise to a single object file. It's basically a single source file plus all of its **#include** files

The relative order of initialization of non-local static objects defined in different translations is undefined. How can we figure out the correct order? Well, it's almost impossible, but we can move each non-local static object into its own function, where it's declared `static` instead. These functions return references to the objects they contain, and Clients can call the functions instead of referring to the global objects. Looks like a Singleton pattern.

```cpp
class FileSystem { ... }; // as before
FileSystem& tfs() {   // this replaces the tfs (the file system) object; it could be static in the FileSystem class
  return fs;          // return a reference to it
}

class Directory { ... };  // as before
Directory::Directory(params) {  // as before, except references to tfs are now to tfs()
  std::size_t disks = tsf().numDisks();
}

Directory& tempDir() {  // this replaces the tempDir object; it could be static in the Directory class
  static Directory td;  // define/initialize local static object
  return td;            // return a reference to it
}
```

The scheme is simple: define and initialize a local static object on line 1, return it on line 2.
