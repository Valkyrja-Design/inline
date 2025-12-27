+++
date = '2025-12-27T19:42:03+05:30'
draft = false
title = 'std::unordered_map: emplace vs try_emplace'
tags = ['C++', 'STL', 'unordered_map', 'emplace', 'try_emplace']
description = 'Comparing insertion methods in std::unordered_map'
+++

`std::unordered_map` provides 3 primary methods for inserting elements with each differing in how
the key and value type are constructed and inserted. `insert` is mostly the same as `emplace` except
when called with an object of the map's value type directly.

## [`emplace`](https://en.cppreference.com/w/cpp/container/unordered_map/emplace.html)

`emplace` constructs the key and value in place using the provided arguments. This means that
even if the key is immovable and non-copyable `emplace` still works. The downside of emplace is that
the value object is always constructed even if the key already exists in the map.

## [`try_emplace`](https://en.cppreference.com/w/cpp/container/unordered_map/try_emplace.html)

`try_emplace` constructs the value only if the key does not already exist in the map. The downside
of `try_emplace` is that the key must be copyable or movable, as it cannot construct the key
in-place like emplace can with `std::piecewise_construct`. Now `try_emplace` also uses piecewise
construction, but the subtle difference is in the arguments used for the construction.

[See demo on Godbolt](https://godbolt.org/z/dsMa99bKG)

```c++
#include <iostream>
#include <unordered_map>

struct A {
    A(int x) : x{x} { std::cout << "A ctor\n"; }
    A(const A&) { std::cout << "A copy ctor\n"; }
    A(A&&) { std::cout << "A move ctor\n"; }
    bool operator==(const A& other) const { return x == other.x; }

    int x;
};

struct B {
    B(int x) : x{x} { std::cout << "B ctor\n"; }
    B(const B&) { std::cout << "B copy ctor\n"; }
    B(B&&) { std::cout << "B move ctor\n"; }
    int x;
};

template <>
struct std::hash<A> {
    std::size_t operator()(const A& a) const noexcept {
        return std::hash<int>{}(a.x);
    }
};

int main() {
    std::unordered_map<A, B> mp;

    // This does moves
    std::cout << "emplace with key object:\n";
    mp.emplace(A{1}, B{1});

    std::cout << "emplace with constructor arguments:\n";
    // But this constructs everything in-place
    mp.emplace(2, 2);

    std::cout << "try_emplace:\n";
    // try_emplace takes the key directly as argument (not the parameters needed
    // to call the key type's constructor). So it has to perform a move/copy if
    // insertion happens. Note that the corresponding value is still constructed
    // in-place
    mp.try_emplace(A{1}, 3);
}
```

See also

- [std::map emplace without copying value](https://stackoverflow.com/questions/27960325/stdmap-emplace-without-copying-value/27960637#27960637)
- [reason to use emplace over try_emplace](https://stackoverflow.com/questions/46046828/is-there-any-reason-to-use-stdmapemplace-instead-of-try-emplace-in-c17)