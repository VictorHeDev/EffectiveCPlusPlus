# Designs and Declarations
## Item 18: Make interfaces easy to use correctly and hard to use incorrectly
Many client errors can be prevented by the introduction of new types.
```cpp
struct Day {
  explicit Day(int d)
  : val(d) {}
  int val;
};


struct Month {
  explicit Month(int d)
  : val(d) {}
  int val;
};

struct Year {
  explicit Year(int d)
  : val(d) {}
  int val;
};

class Date {
  public:
    Date(const Month& m, const Day& d, const Year& y);
}

Date d(30, 3, 1995);  // error: wrong types
Date d(Day(30), Month(3), Year(1995));  // error: wrong types
Date d(Month(3), Day(30), Year(1995));  // okay, types are correct
```
Making `Day`, `Month`, and `Year` full-fledged classes with encapsulated data would be better than the simple use of structs above. Introduction of new types can help prevent interface usage errors. We can use an enum to represent the month, but enums are not type safe as we might want.

```cpp
class Month {
  public:
    static Month Jan() { return Month(1); } // functions returning all valid Month values
    static Month Feb() { return Month(2); }
    static Month Dec() { return Month(12); }  // why use functions rather than member functions?
  private:
    explicit Month(int m);      // prevent creation of new Month values
};
Date d(Month::Mar(), Day(30), Year(1995));
```

## Item 19: Treat class design as type design
How do you design effective classes? Confront these questions:
* HOw should objects of your new type be created and destroyed?
* How should object initialization differ from object assignment?
* What does it mean for objects of your new type to be passed by value? Remember that the copy constructor defines how pass-by-value is implemented for a type.
* What are the restrictions on legal values for your new type?
* Does your new type fit into an inheritance graph? If you wish to allow other classes to inherit from your class, that affects whether the functions you declare are virtual, especially your destructor
* What kind of type conversions are allowed for your new type?
* What operators and functions make sense for the new type?
* What standard functions should be disallowed? Those are the ones you'll need to declare `private`
* Who should have access to the members of your new type? This should help determine which members are public, private, or protected. It also helps determine which classes and/or functions should be friends, as well as whether it makes sense to nest one class inside another
* What is the "undeclared interface" of your new type? The guarantees you offer in these areas will impose constraints on your class implementation
* How general is your new type? If you're defining a whole family of types, maybe you want to define a new class template instead of only a class
* Is a new type really what you need?

## Item 20: Prefer pass-by-reference-to-const to pass-by-value
