# Constructors, Destructors, and Assignment Operators

These fundamental operations control bringing a new object into existence and making sure it's initialized, getting rid of an object and making sure it's properly cleaned up, and giving an object a new value.

## Item 5: Know what functions C++ silently writes and calls

If you don't declare them yourself, compilers will declare their own versions of a copy constructor, a copy assignment operator, and a destructor. If you declare no constructors at all, compilers will also declare a default constructor for you.

```cpp
class Empty{}; // this is the same as below

class Empty {
  public:
    Empty() {}; // default copy constructor
    Empty(const Empty& rhs) {};   // copy construtor
    ~Empty() {};  // destructor
    Empty& operator=(const Empty& rhs) {};  // copy assignment operator
}

// invoked functions
Empty e1;   // default construtor & destructor once object lifetime ends
Empty e2(e1); // copy constructor
e2 = e1;    // copy assignment operator
```

## Item 6: Explicitly disallow the use of compiler-generated functions you do not want

By declaring a member function explicitly, you prevent compilers from generating their own version, and by making the function `private`, you keep people from calling it. Declaring member functions `private` and deliberately not implementing them is well established.

```cpp
class HomeForSale {
  public: // ...
  private:
    HomeForSale(const HomeForSale&);  // declarations only - notice the lack of parameter names which is common practice
    HomeForSale& operator=(const HomeforSale&);
}
```

Alternatively you can create a base class called `Uncopyable` and have your subclass inherit from it.

```cpp
class Uncopyable {
  protected:  // allow construction
    Uncopyable() {} // and destruction of
    ~Uncopyable() {}  // derived objects
  private:
    Uncopyable(const Uncopyable&);  // prevent copying
    Uncopyable& operator=(const Uncopyable&);
};

class HomeForSale: private Uncopyable {   // class no longer declares copy ctor nor copy assign operator
};
```

Copying operations are private in the base class so those calls will be rejected by the compiler.

## Item 7: Declare destructors virtual in polymorphic base classes

```cpp
class TimeKeeper {
  public:
    TimeKeeper();
    ~TimeKeeper();
};

class AtomicClock: public TimeKeeper {};
class WaterClock: public TimeKeeper {};
class WristWatch: public TimeKeeper {};

// clients want access to the time without worrying about the details of how it's calculated -- create
// a factory function that returns a base class pointer to a newly-created derived class object
TimeKeeper* getTimeKeeper();  // returns a pointer to a dynamically allocated object of a class derived from TimeKeeper
```

Because the objects returned by `getTimeKeeper` live on the heap, they must be properly deleted/deconstructed.

```cpp
TimeKeeper *ptk = getTimeKeeper();  // get dynamically allocated object from TimeKeeper hierarchy and use it
delete ptk;   // release it to avoid a resource leak
```

Relying on clients to perform the deletion is error-prone, so we should do it ourselves. This will properly destroy `TimeKeeper` objects, but not the sub-classes of TimeKeeper like AtomicClock, WaterClock, and WristWatch. To eliminate this problem, give the base class a virtual destructor. Then deleting a derived class object will do exactly as we thought it would--it will destroy the entire object including all its derived class parts.

```cpp
class TimeKeeper {
  public:
    TimeKeeper();
    virtual ~TimeKeeper();
};
timeKeeper *ptk = getTimeKeeper();
delete ptk;
```

Base classes like `TimeKeeper` generally contain virtual functions other than the destructor, because the purpose of virtual functions is to allow customization of derived class implementations. Any class with virtual functions should almost always have a virtual destructor. If a class does not contain virtual functions, that often indicates that it is not meant to be used as a base class.

`vptr` - "virtual table pointer" points to an array of function pointers called a `vtbl` "virtual table." When a virtual function is invoked on an object, the actual function called is determined by following the object's `vptr` to a `vtbl` and looking up the appropriate function pointer in the `vtbl`.

Declaring all destructors virtual is just as wrong as never declaring them virtual. So what can we do? The golden rule is to declare a virtual destructor in a class if and only if that class contains at least one virtual function.

Occasionally, you might want to give a class (abstract class) a pure virtual destructor. However, you MUST provide a definition for the pure virtual destructor.

```cpp
class AWOV {  // AWOV = "Abstract without Virtuals
  public:
    virtual ~AWOV() = 0;  // declare pure virtual destructor
};

AWOV:: ~AWOV() {} // definition of a pure virtual dtor
```

## Item 8: Prevent exceptions from leaving destructors

C++ does not like destructors that emit exceptions (especially when 2 exceptions are thrown at the same time!) There are 2 ways to avoid trouble:

1. Destructor could terminate the program. If `.close()` throws, then typically call `abort`
2. Swallow the exception arising from the call to close. This will suppress important information, but you can still continue with the program.

In general, swallowing exceptions in the destructor is a bad idea because it suppresses important information. Neither of the approaches above are compelling. A better strategy is to design the `DBConn`'s interface so that its clients have an opportunity to react to problems that may arise from the thrown exception. It can also keep track of whether or not the `DBConnection` had been closed to prevent the connection from leaking.

```cpp
class DBConn {
  public:
    void close() {
      db.close();
      closed = true; // new function for client use
    }

    ~DBConn() {
      if (!closed) {
        try {   // close the connection if the client didn't
          db.close;
        } catch {   // if closing fails
          // make log entry that call to close failed
          // terminate or swallow
        }
      }
    }

  private:
    DBConnection db;
    bool closed;
}
```

If an operation may fail by throwing an exception and there may be a need to handle that exception, the exception has to come from some non-destructor function. Destructors that emit exceptions are dangerous and always run the risk of premature program termination or undefined behavior.

## Item 9: Never call virtual functions during construction or destruction

The virtual calls won't do what you think. During base class construction or destruction, virtual functions never go down into derived classes. Instead, the object behaves as if it were of the base type. Because base class constructors execute before derived class constructors, derived class data members have not been initialized when base class constructors run. Instead do this:

```cpp
class Transaction {
  public:
    explicit Transaction(const std::string& logInfo);
    void logTransaction(const std::string& logInfo) const;  // now a non-virtual function
};

Transaction::Transaction(const std::string& logInfo) {
  logTransaction(logInfo);  // now a non-virtual call
}

class BuyTransaction: public Transaction {
  public:
    BuyTransaction(parameters)
    : Transaction(createLogString(parameters)) // pass log info
    {}  // to base class constructor

  private:
    static std::string createLogString(parameters);
}
```

Since you can't use virtual functions to call down from base classes during construction, you can compensate by having derived classes pass necessary construction information up to base class constructors instead.

## Item 10: Have assignment operators return a reference to `*this`

Assignment is right-associative. Assignment returns a reference to its left-hand argument. Follow this pattern when implementing assignment operators for classes:

```cpp
class Widget {
  public:
    Widget& operator=(const Widget& rhs) {  // return type is a reference to the current class
      // ...
      return *this;   // return the left-hand object
    }

    Widget& operator+=(const Widget& rhs) {  // convention applies to +=, -=, *=, etc.
      // ...
      return *this;
    }

    Widget& operator=(const Widget& rhs) {  // it applies even if the operator's parameter type is unconventional
      // ...
      return *this;
    }
};
```

## Item 11: Handle assignment for self in `operator=.`

An assignment to self occurs when an object is assigned to itself. What does that mean?

```cpp
class Widget {};
Widget w;
w = w;  // assignment to self
a[i] = a[j];
*px = *py; // potential assignment if px and py happen to point at the same thing
```

The traditional way to prevent this error is to check for assignment to self via an **identity test** at the top of `operator=`. A newer and alternative way to handle this is to manually order statements in `operator=` to make sure the implementation is both exception and self-assignment safe is to use a technique known as "copy and swap."

```cpp
class Widget {
  void swap(Widget& rhs); // exchange *this's and rhs's data;
};

Widget& Widget::operator=(const Widget& rhs) {
  Widget temp(rhs);   // make a copy of rhs' data
  swap(temp);         // swap *this' data with the copy's
  return *this;
}
```

If a class' copy assignment operator is declared to take its argument by value, then passing something by value creates a **copy** of it. We can leverage that to swap without creating the copy ourselves.

```cpp
class Widget {
  void swap(Widget& rhs); // exchange *this's and rhs's data;
};

Widget& Widget::operator=(const Widget rhs) { // rhs is a copy of the object and is passed by value
  swap(temp);         // swap *this' data with the copy's
  return *this;
}
```

## Item 12: Copy all parts of an object
Copying functions include the `copy constructor` and `copy assignment operator`. If you add a data member to a class, you need to make sure that you update the copying functions too. Any time you write copying functions for a derived class, you must also copy the base class parts.
When you're writing a copying function be sure to:
1. Copy all local data members
2. Invoke the appropriate copying function in all base classes

To avoid code duplication in copy constructors and copy assignment operators, you can create a third member function that calls both. This special function is typically called `init` and is a private function. 