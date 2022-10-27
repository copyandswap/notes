# move semantics

## lvalue & rvalue

- Every C++ expression has a data type and a value category.
- In C++ every expression is either an lvalue or an rvalue.

- Value categories have two properties of expressions:

  - has identity
  - can be moved from

- Expressions that:

1. have identity and cannot be moved from are called lvalue expressions
2. have identity and can be moved from are called xvalue expressions
3. do not have identity and can be moved from are called prvalue expression
4. do not have identity and cannot be moved from are not used

```text
    |                         | Can be moved from (=rvalue) | Cannot be moved from |
    --------------------------------------------------------------------------------
    | Has identity (=glvalue) | xvalue                      | lvalue               |
    | No identity             | prvalue                     | not used             |
```

**lvalue**

- An lvalue refers to a memory location and allows us to take the address of that memory location via the & operator.
- Variable names, including const variables, array elements, function calls that return an lvalue reference, unions, class members, and string literal

**rvalue**

- An rvalue is an expression that is not an lvalue and has no address.
- Literals, function calls that return a non-referenced type, and temporary objects

**xvalue**

- An xvalue is an expring value and unnnamed objects that is soon to be destroyed.
- It may be either treated as glvalues or as rvalues depending on the context.
- Function calls that return an rvalue reference or use std::move

```c++
    int i = 4; // i: lvalue, 4: rvalue
    const char *s = "hello"; // s: lvalue, string literal: lvalue

    int f1(); // this call is an rvalue
    int &f2(); // this call is an lvalue
    int &&f3(); // this call is an xvalue
    static_cast<int &&>(x); // xvalue
    std::move(x); // xvalue
```

## lvalue reference 

- It is marked with one ampersand &.
- An lvalue reference is a reference that binds to an lvalue.
- However, we can bind an rvalue to a const lvalue reference.


```c++
    int x = 5;
    int &ref = x; // ok

    int &ref = 42; // error
    const int &ref = 42; // ok
```

## rvalue reference

- It is marked with two ampersand &&.
- An rvalue reference is a reference that binds to an rvalue.
- It cannot bind to an lvalue.
- It supports the implementation of move semantics and perfect forwarding.

```c++
    int &&ref = 42; // ok

    int a = 42;
    int &&ref = a; // error
```

## move semantics

- Rvalue references support the implementation of move semantics.
- It avoids expensive copies, transfers ownership resources move from, and more efficient code.
- It allows us to optimize the copying of objects where we no longer need the value. It can used implicitly (for unnamed temporary objects or local return values) or explicitly with std::move.
- std::move means I no longer need this value here. It converts the expression into an rvalue.
- Moved-from objects are still valid but unspecified value.
- It has no effect for const objects.
- Copy semantics is used as a fallback for move semantics (if copy semantics are supported).  

- We now have three major ways of call-by-reference:

1. **non-const lvalue reference**
2. **const lvalue reference**
3. **rvalue reference**

```c++
    void foo(std::string &arg); // binds to an lvalue
    void foo(const std::string &arg); // binds either to an lvalue or an rvalue
    void foo(std::string &&arg); // binds to an rvalue such as temporary objects or non-const object marked with std::move
```

## move constructor & move assigment operator

- Similar to copy constructor and copy assigment operator, except parameter is an rvalue reference
- Transfer ownership of resources from existing object to object being constructed
- Leave the source object in a valid state but unspecified value

```c++
    struct foo {
        foo(foo &&other) noexcept;
        foo &operator=(foo &&other) noexcept;
    };
```

- Move operations should be explicitly noexcept that are implemented strong exception guarantee. Because moves are supposed to transfer resources and not allocate or acquire resources.

```c++
    struct s1 {
        int a;
        std::string b;
        s1(s1 &&other) noexcept = default;
        s1 &operator(s1 &&other) noexcept = default;
    };

    struct s2 {
        double *data;
        s2(s2 &&other) noexcept : data(std::move(other.data)) {
            other.data = nullptr;
        }

        s2 &operator(s2 &&other) noexcept {
            if (this != &other) {
                delete[] data;
                data = std::move(other.data);
                other.data = nullptr;
            }
            return *this;
        }
    };

    struct s3 {
        double *data;
        s3(s3 &&other) noexcept : data(std::exchange(other.data, nullptr)) {}

        s3 &operator(s3 &&other) noexcept {
            if (this != &other) {
                delete[] data;
                data = std::exchange(other.data, nullptr);
            }
            return *this;
        }
    };
```

**std::exchange**

- It assigns value to a object and returns the old value of that object.
- It is useful when implementing move operations.

```c++
    template<class T, class U = T>
    T exchange(T &object, U &&value);
``` 

## Rule of 0/3/5

- If default behavior is correct for all five, you let compiler do everything.
- If you must define one of the five, you should declares all of them explicitly.

**Notes**

- Destructor, =default and =delete functions count as user-defined!
- The default copy operations are generated, if no move operation is user-defined.
- The default move operations are generated, if no copy operation or destructor is user-defined.

## std::move

- It doesn't move anything. 
- It converts any expression into an rvalue so it can be bound to an rvalue reference.
- It only says "I no longer need this value".
- The C++ standard library guarantees that move-from objects are in a valid but unspecified state.

```c++
    template<typename T>
    constexpr remove_reference_t<T> &&move(T &&t) noexcept {
        return static_cast<remove_reference_t<T> &&>(t);
    }
```

- You might wonder why moved-from objects are still valid objects and are not destroyed. The reason is that there are some useful applications where it makes sense to use moved-from objects again.

```c++
    std::vector<std::string> rows;
    std::string row;
    while (std::getline(mystream, row)) {
        rows.push_back(std::move(row));
    }

    template<typename T>
    void swap(T &a, T &b) {
        T tmp = std::move(a);
        a = std::move(b);
        b = std::move(tmp);
    }
```

## noexcept

todo...

## difference between move operations and swap

todo...

## perfect forwarding

todo...