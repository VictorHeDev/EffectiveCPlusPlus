# Resource Management

Regardless of the resource, once you are done using it then it needs to be returned to the system.

## Item 13: Use objects to manage resources

To make sure that the resource returned by a factory function is always released, we need to put that resource inside an object whose destructor will automatically release that resource when control leaves the function. By putting resources inside objects, we can rely on C++'s automatic destructor invocation to make sure that the resources are released. Many resources are dynamically allocated on the heap, are used only within a single block or function, and should be released when control leaves that block or function. The standard library's `auto_ptr` is tailor-made for this kind of situation. `auto_ptr` is a pointer-like object (smart pointer) whose destructor automatically calls `delete` on what it points to.

```cpp
void f() {
  std::auto_ptr<Investment>pInv(createInvestment());  // call factory function and use pInv as before. Automatically delete pInv via auto_ptr's dtor
}
```

2 critical aspects of using objects to manage resources:

1. Resources are acquired and immediately turned over to resource-managing objects. Using objects to manage resources is often called **Resource Acquisition Is Initialization** (RAII) because it's so common to acquire a resource and initialize a resource-managing object in the same statement.
2. Resource-managing objects use their destructors to ensure that resources are released.

An alternative to `auto_ptr` is a **reference-counting smart pointer** (RCSP), which is a smart pointer that keeps track of how many objects point to a particular resource and automatically deletes the resource when nobody is pointing to it any longer. Because `auto_ptr` and `tr1::shared_ptr` both use `delete` in their destructors, not `delete []`, using them with dynamically allocated arrays is a bad idea.

To summarize, to prevent resource leaks, use RAII objects that acquire resources in their constructors and release them in their destructors. Two commonly useful RAII classes are `tr1::shared_ptr` and `auto_ptr.tr1::shared_ptr` is usually the better choice, because its behavior when copied is intuitive. Copying an `auto_ptr` sets it to null.

## Item 14: Think carefully about copying behavior in resource-managing classes

What happens when an RAII object is copied? Most of the time we want to choose one of the following possibilities:

- Prohibit copying (we can use the Uncopyable class we created)
- Reference-count the underlying resource. Sometimes it's desirable to hold onto a resource until the last object using it has been destroyed. When that's the case, copying an RAII object should increment the count of the number of objects referring to the resource.
- Copy the underlying resource. Copying a RAII object should perform a "deep copy."
- Transfer ownership of the underlying resource.

## Item 15: Provide access to raw resources in resource-managing classes

APIs often require access to raw resources, so each RAII class should offer a way to get at the resource it manages. Access may be via explicit conversion or implicit conversion. In general, explicit conversion is safer, but implicit conversion is more convenient for clients.

## Item 16: Use the same form in corresponding uses of `new` and `delete`

When you employ a `new` expression, 2 things happen.

1. Memory is allocated
2. > = 1 ctrs are called for that memory

When you employ a `delete` expression, 2 other things happen

1. > = 1 destructors are called for the memory, then the memory is deallocated

The number of objects residing in the memory being deleted will determine how many destructors must be called. If you use `[]` in a `new` expression, you must use `[]` in the corresponding `delete` expression.

## Item 17: Store `new`ed objects in smart pointers in standalone statements

Failure to do this can lead to subtle resource leaks when exceptions are thrown.
