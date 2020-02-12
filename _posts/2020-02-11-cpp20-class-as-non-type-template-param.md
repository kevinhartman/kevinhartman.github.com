---
title: Literal Classes as Non-type Template Parameters in C++20
date: 2020-02-05 00:00:00 +0800
categories: [Programming Languages]
tags: [cpp, cpp20]
seo:
  date_modified: 2020-02-12 09:13:10 -0500
  description: Using user-defined literal classes as non-type template parameters
    in C++20.
---

With the introduction of C++20, it's now possible to provide a literal class type (i.e. `constexpr` class) instance as a template parameter.

> As a refresher, a non-type template parameter is a template parameter that does not name a type, but rather, a constant ***value*** (e.g. `template<int value>`).

## Literal Class Example

```c++
#include <iostream>

struct NullOptT {} NullOpt;

/**
 * Literal class type.
 *
 * Represents an optionally provided `int`.
 */
struct OptionalInt {
    constexpr OptionalInt(NullOptT) {}
    constexpr OptionalInt(int value): has_value(true), value(value) {}
            
    const bool has_value = false;
    const uint32_t value {};
};

/**
 * Prints whether or not a value was provided for "maybe" WITHOUT branching :)
 */
template<OptionalInt maybe>
void Print() {
    if constexpr(maybe.has_value) {
        std::cout << "Value is: " << maybe.value << std::endl;
    } else {
        std::cout << "No value." << std::endl;
    }
}

// Note: implicit conversions are at play!
int main()
{
    Print<123>();     // Prints "Value is: 123"
    Print<NullOpt>(); // Prints "No value."
}
```

In above example, the `Print` function accepts a non-type template parameter of type `OptionalInt`.

Prior to C++20, non-type template parameters were restricted to:

> - lvalue reference type (to object or to function);
> - an integral type;
> - a pointer type (to object or to function);
> - a pointer to member type (to member object or to member function);
> - an enumeration type;
> - std::nullptr_t (since c++11)

Now (C++20), this has been extended with support for:
> - an floating-point type;
> - a literal class type with the following properties: 
>   - all base classes and non-static data members are public and non-mutable and
>   - the types of all bases classes and non-static data members are structural types or (possibly multi-dimensional) array thereof.
>
> From <https://en.cppreference.com/w/cpp/language/template_parameters>

A type which satisfies one of the above definitions is considered to be a ***structural*** type.

From the above example, `OptionalInt` satisfies the definition for allowable literal class types. This custom class type allows the `Print` function to accept an *optional* integer as a template parameter. Since this value is provided as a template parameter, the implementation of `Print` can use it in constant expressions (e.g. as the condition to an `if constexpr` to conditionally compile *only* the relevant branch).

Note that `std::optional` is ***not*** considered to be a structural type, and thus cannot be used with non-type template parameters. More on this below.

# Limitations Explained
As noted above, the following limitations apply to literal class types used as non-type template parameters.

## All base classes and non-static data members are **public** and **non-mutable**.
This means that these class types can *not* have private data fields. They *can* however have private functions.

Regarding the non-mutable data member requirement, I would *expect* this to mean that the presence of either a non-const data member or a non-const member function should cause a compilation failure, but this does not appear to be enforced in GCC (9.2.0). I've [filed a bug](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=93705), just in case.

```c++
class Literal {
public:
    constexpr Literal() {}

private:
    const int private_data {}; // Fails to compile!

    constexpr int PrivateFunc() const { return 123; } // This is allowed.
}
```

## The types of all bases classes and non-static data members are **structural types** or array thereof.
This means that literal class types must be comprised of data members that are also allowable under the same restrictions for non-type template parameters. That is to say they must *also* be structural types.

# Compiler Compatibility
The examples provided here have been tested using GCC 9.2.0 (support was added in GCC 9).

Clang's C++2a implementation does not yet support class types as non-type template parameters. The current implementation status of C++2a features in Clang can be found here: <https://clang.llvm.org/cxx_status.html#cxx20>.

# Relevant Proposals
This feature was originally proposed in [P0732R2](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0732r2.pdf) and was refined by [P1907R1](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1907r1.html).
