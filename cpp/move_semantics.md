# move semantics

## lvalue & rvalue

- Every C++ expression has a data type and a value category.
- In C++ every expression is either an lvalue or an rvalue.

- Value categories have two properties of expressions:

1. has identity
2. can be moved from

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

1. lvalue

- An lvalue refers to a memory location and allows us to take the address of that memory location via the & operator.
- Variable names, including const variables, array elements, function calls that return an lvalue reference, unions, class members, and string literal

2. rvalue

- An rvalue is an expression that is not an lvalue and has no address.
- Literals, function calls that return a non-referenced type, and temporary objects

3. xvalue

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