---
title: String Literal Template Parameters in C++20
date: 2020-02-12 16:45:04 -0500
categories: [Programming Languages]
tags: [cpp, cpp20]
seo:
  description: Using class literal non-type template parameters to implement string
    literal template parameters in C++20.
  date_modified: 2020-02-12 16:46:00 -0500
---

In a [recent post](/posts/cpp20-class-as-non-type-template-param/), I described a C++20 feature that allows literal classes to be used as non-type template parameters. The original feature proposal mentions how this could enable constant expression strings to be passed as template parameters as well (see [P0732R2, section 3.2](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0732r2.pdf)).

In summary, this can be achieved by wrapping a constant expression string inside of a class literal type.

After [a discussion](https://www.reddit.com/r/cpp/comments/f2s4ut/literal_classes_as_nontype_template_parameters_in/fhf3jyt/) with Reddit user [theICEBear_dk](https://www.reddit.com/user/theICEBear_dk), I was motivated to come up with an implementation.

The following sample demonstrates this, making use of an implicit conversion to create the illusion that a string literal can be pass directly as a template parameter.

```c++
#include <iostream>
#include <algorithm>

/**
 * Literal class type that wraps a constant expression string.
 *
 * Uses implicit conversion to allow templates to *seemingly* accept constant strings.
 */
template<size_t N>
struct StringLiteral {
    constexpr StringLiteral(const char (&str)[N]) {
        std::copy_n(str, N, raw);
    }
    
    // Retrieves the wrapped string as a constant :)
    constexpr auto& value() const { return raw; }
    
    char raw[N];
};

template<StringLiteral lit>
void Print() {
    // Get the size of the string as a constant expression.
    constexpr auto size = sizeof(lit.value());
    
    std::cout << "Size: " << size << ", Contents: " << lit.value() << std::endl;
}

int main()
{
    Print<"literal string">(); // Prints "Size: 15, Contents: literal string"
}
```
