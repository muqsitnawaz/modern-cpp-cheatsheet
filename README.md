# Effective Modern C++ Cheatsheet

## Shorthands

1. **ref(s)**: reference(s)
2. **op(s)**: operation(s)

## Terms

1. **lvalue**: typically an expression whose address can be taken e.g a variable name (`auto x = 10;`)
2. **rvalue**: an expression whose address cannot be taken in C++ i.e before C++11 e.g literal types (`10`)
3. **lvalue-ref(erence)**: reference to an _lvalue_ type typically denoted by `&` e.g `auto& lvalue_ref = x;`
4. **rvalue-ref(erence)**: reference to an _rvalue_ type typically denoted by `&&` e.g `auto&& rvalue_ref = 10;`
5. **copy-operations**: _copy-construct_ from _lvalues_ using copy-constructor and copy-assignment operator
6. **move-operations** _move-construct_ from _rvalues_ using move-constructor and move-assignment operator
7. **arguments**: expressions passed to a function call at call site (could be either _lvalues_ or _rvalues_)
8. **parameters**: _lvalue names_ initialized by arguments passed to a function e.g `x` in `void foo(int x);`
9. **callable objects**: objects supporting member `operator()` e.g _functions_, `lambda`s, `std::function` etc
10. **declarations**: introduce names and types without details e.g `class Widget;`, `void foo(int x);`
11. **definitions**: provide implementation details e.g `class Widget { ... };`, `void foo(int x) { ... }`

## Chapter 1. Deducing Types

### Item 1: Understand `template` type deduction

* Deduced type of T doesn't always match that of the parameter (i.e ParamType) in template functions
* For _lvalue-refs/rvalue-refs_, compiler ignores the reference-ness of an _arg_ when deducing type of T
* With _universal-refs_, type deduction always distinguishes between _l-value_ and _r-value_ argument types
* With _pass-by-value_, _reference-ness_, `const` and `volatile` are ignored if present in the ParamType
* Raw arrays `[]` and function types always decay to pointer types unless they initialize references

### Item 2: Understand `auto` type deduction

* `auto` plays the role of `T` while its type specifier (i.e including `const` and/or _ref_) as ParamType
* For a braced initializer e.g `{1, 2, 3}`, `auto` _always_ deduces `std::initializer_list` as its type
* **Corner case**: `auto` as a _callable_ `return` type uses _template type deduction_, not _auto type deduction_

### Item 3: Understand `decltype`

* `decltype`, typically used in function `template`s, determines a variable or an expression's type
* `decltype(auto)`, unlike `auto`, includes _ref-ness_ when used in the `return` type of a _callable_
* **Corner case**: `decltype` on _lvalue expression_ (except _lvalue-names_) yields _lvalue-refs_ not _lvalues_

### Item 4: How to view deduced types?

* You can update your code so that it leads to a compilation failure, you will see the type in diagnostics
* `std::type_info::name` (and `typeid()`) depends upon compiler; use [Boost.TypeIndex](https://www.boost.org/doc/libs/1_66_0/doc/html/boost_typeindex/examples.html) library instead

## Chapter 2. `auto`

### Item 5: Prefer `auto` declarations

* `auto` prevents _uninitialized variables_ and _verbose declarations_ (e.g `std::unordered_map<T>::key_type`)
* Use `auto` especially when declaring _lambdas_ to directly hold _closures_ unlike `std::function`
  
### Item 6: How to fix undesired `auto` type deduction?

* Use `auto` with `static_cast` (a.k.a _explicitly typed initializer idiom_) to enforce correct types
* Never use `auto` directly with _invisible proxy classes_ such as `std::vector<bool>::reference` 

## Chapter 3. Moving to Modern C++

### Item 7: Distinguish between () and {} (aka braced/uniform initializer) when creating objects

* Braced initializer i.e `{}` prevents _narrowing conversions_ and _most vexing parse_ while `()` doesn't
* During _overload-resolution_, `std::initializer_list` version is always preferred for `{}` types
* **Corner case**: `std::vector<int> v{10, 20}` creates a vector with 10 and 20, not 10 `int`s initialized to 20.

### Item 8: Prefer `nullptr` to `0` and `NULL`

* Don't use `0` or `NULL`, use `nullptr` of type `nullptr_t` which represents pointers of all types!
  
### Item 9: Prefer alias declarations to `typedefs`

* Alias declarations (declared with `using` keyword) support templatization while `typedefs` don't
* Alias declarations avoid 1) `::type` suffix 2) `typename` prefix when referring to other typedefs

### Item 10: Prefer _scoped_ `enums` to _unscoped_ `enums`

* Use `enum class` instead of `enum` to limit scope of an `enum` members to just inside the `enum`
* `enum class`es use `int` by default, prevent _implicit conversions_ and permit forward declarations

### Item 11: Prefer public-deleted functions to private-undefined versions

* Always make unwanted functions (such as _copy-operations_ for move-only types) `public` and `delete`

### Item 12: Always declare overriding functions `override`

* Declare overriding functions in derived types `override`; use `final` to prevent further inheritance

### Item 13: Always prefer `const_iterators` to `iterators`

* Prefer `const_iterators` to `iterators` for all STL containers e.g `cbegin` instead of `begin`
* For max generic code, don't assume the existence of member `cbegin`; use `std::begin` instead

### Item 14: Declare functions `noexcept` if they won't emit exceptions

* Declare functions `noexcept` when they don't emit exceptions such as functions with _wide contracts_
* Always use `noexcept` for move-operations, `swap` functions and memory allocation/deallocation
* When a `noexcept` function emits an exception: stack is _possibly_ wound and program is terminated

### Item 15: Use `constexpr` whenever possible

* `constexpr` objects are always `const` and usable in _compile-time evaluations_ e.g `template` parameters
* `constexpr` functions produce results at compile-time only if all of their args are known at compile-time
* `constexpr` objects and functions can be used in a wider context i.e _compile-time_ as well as _runtime_

### Item 16: Make `const` member functions thread-safe

* Make member functions of a type `const` as well as `thread-safe` if they do not modify its members
* For synchronization issues, consider `std::atomic` first and then move to `std::mutex` if required

### Item 17: Understand when your compiler generates special member functions

* Compiler generates a _default constructor_ only if the class type declares no _constructors_ at all
* Declaring _destructor_ and/or _copy ops_ disables the generation of _default move ops_ and vice versa
* _Copy assignment operator_ is generated if: 1) not already declared 2) no move op is declared

## Chapter 4. Smart Pointers

### Item 18: Use `std::unique_ptr` for exclusive-ownership of resource management

* `std::unique_ptr` owns what it points to, is fast as raw pointer (`*`) and supports custom deleters
* Conversion to a `std::shared_ptr` is easy, therefore _factory functions_ should always return `std::unique_ptr`
* `std::array`, `std::vector` and `std::string` are generally better choices than using raw arrays `[]`

### Item 19: Use `std::shared_ptr` for shared-ownership resource management

* `std::shared_ptr` points to an object with _shared ownership_ but doesn't actually own the object
* `std::shared_ptr` stores/updates _metadata_ on heap and can be up to 2x slower than `std::unique_ptr`
* Unless you want custom deleters, prefer `std::make_shared<T>` for creating shared pointers
* Don't create multiple `std::shared_ptr`s from a single raw pointer; it leads to _undefined behavior_
* For `std::shared_ptr` to `this`, always inherit your class type from `std::enable_shared_from_this`

### Item 20: Use `std::weak_ptr` for `std::shared_ptr`-like pointers that can dangle

* `std::weak_ptr` operates with the possibility that the object it points to might have been destroyed
* `std::weak_ptr::lock()` returns a `std::shared_ptr`, but a `nullptr` for _destroyed objects_ only
* `std::weak_ptr` is typically used for _caching_, _observer lists_ and prevention of _shared pointers cycles_

### Item 21: Prefer make functions (i.e `std::make_unique` and `std::make_shared`) to direct use of new

* Use _make functions_ to remove source code duplication, improve exception safety and performance
* When using `new` (in cases below), prevent memory leaks by immediately passing it to a _smart pointer_!
* You must use `new` when 1) specifying custom deleters 2) pointed-to object is a _braced initializer_
* Use `new` when `std::weak_ptr`s outlive their `std::shared_ptr`s to avoid _memory de-allocation delays_

### Item 22: When using Pimpl idiom, define special member functions in an implementation file

* _Pimpl idiom_ puts members of a type inside an impl type (`struct Impl`) and stores a pointer to it
* Use `std::unique_ptr<Impl>` and always implement your destructor and _copy/move ops_  in an impl file

## Chapter 5. Rvalue references, move semantics and perfect forwarding

* _Move semantics_ aim to replace expensive _copy ops_ with the cheaper _move ops_ when applicable
* _Perfect forwarding_ forwards a function's args to other functions parameters while preserving types

### Item 23: Understand `std::move` and `std::forward`

* `std::move` performs an unconditional cast on _lvalues_ to _rvalues_; you can then perform _move ops_
* `std::forward` casts its _input arg_ to an _rvalue_ only if the _arg_ is bound to an _rvalue_ name

### Item 24: Distinguish universal-refs from rvalue-refs

* Universal-refs (i.e `T&&` and `auto&&`) always _cast_ lvalues to _lvalue-refs_ and rvalues to _rvalue-refs_
* For universal-ref parameters, _auto/template type deduction_ must occur and they must be _non-`const`_

### Item 25: Understand when to use `std::move` and `std::forward`

* Universal references are usually a better choice than overloading functions for _lvalues_ and _rvalues_
* Apply `std::move` on _rvalue refs_ and `std::forward` on _universal-refs_ last time each is used
* Similarly, also apply `std::move` or `std::forward` accordingly when returning by value from functions
* Never return _local objects_ from functions with `std::move`! It can prevent _return value optimization (RVO)_
  
### Item 26: Avoid overloading on universal-references

* _Universal-refs_ should be used when client's code could pass either _lvalue refs_ or _rvalue refs_
* Functions overloaded on _universal-refs_ typically get called more often than expected - avoid them!
* Avoid _perf-forwarding constructors_ because they can hijack _copy/move ops_ for non-`const` types

### Item 27: Alternatives to overloading universal-references

* _Ref-to-const_ works but is less efficient while _pass-by-value_ works but use only for _copyable types_
* _Tag dispatching_ uses an additional parameter type called _tag_ (e.g `std::is_integral`) to aid in matching
* Templates using `std::enable_if_t` and `std::decay_t` work well for _universal-refs_ and they read nicely
* _Universal-refs_ offer efficiency advantages although they sometimes suffer from usability disadvantages

### Item 28: Understand reference collapsing

* Reference collapsing converts `& &&` to `&` (i.e lvalue ref) and `&& &&` to `&&` (i.e rvalue ref)
* Reference collapsing occurs in `template` and `auto` type deductions, alias declarations and `decltype`

### Item 29: Assume that move operations are not present, not cheap, and not used

* Generally, _moving_ objects is usually much cheaper then _copying_ them e.g _heap-based_ STL containers
* For some types e.g `std::array` and `std::string` (with SSO), _copying_ them can be just as efficient

### Item 30: Be aware of failure cases of perfect forwarding

* _Perf-forwarding_ fails when _template type deduction_ fails or deduces wrong type for the arg passed
* Fail cases: _braced initializers_ and passing `0` or `NULL` (instead of `nullptr`) for _null pointers_
* For integral `static const` data members, _perfect-forwarding_ will fail if you're missing their definitions
* For _overloaded_ or `template` functions, avoid _fail cases_ using `static_cast` to your desired type
* Don't pass _bitfields_ directly to perfect-forwarding functions; use  `static_cast` to an _lvalue_ first

## Chapter 6. Lambda Expressions

### Item 31: Avoid default capture modes

* Avoid default `&` or `=` captures for _lambdas_ because they can easily lead to _dangling references_
* Fail cases: `&` when they outlive the objects captured, `=` for member types when they outlive `this` 
* `static` types are always captured _by-reference_ even though default capture mode could be _by-value_

### Item 32: Use init-capture (aka generalized lambda captures) to move objects into (lambda) closures

* _Init-capture_ allows you to initialize types (e.g variables) inside a lambda capture expression

### Item 33: Use `decltype` on `auto&&` parameters for `std::forward`

* Use `decltype` on `auto&&` parameters when using `std::forward` for forwarding them to other functions
* This case will typically occur when you are implementing _perfect-forwarding_ using _auto type deduction_

### Item 34: Prefer _lambdas_ to `std::bind`

* Always prefer _init capture_ based `lambdas` (aka generalized lambdas) instead of using `std::bind`

## Chapter 7. Concurrency API

### Item 35: Prefer `std::async` (i.e task-based programming) to `std::thread` (i.e thread-based)

* When using `std::thread`s, you almost always need to handle scheduling and oversubscription issues
* Using `std::async` (aka task) with _default launch policy_ handles most of the corner cases for you
  
### Item 36: Specify `std::launch::async` for truly asynchronous tasks

* `std::async`'s default launch policy can run either async (in new thread) or sync (upon `.get()` call)
* If you get `std::future_status::deferred` on `.wait_for()`, call `.get()` to run the given task

### Item 37: Always make `std::thread`s unjoinable on all paths

* Avoid _program termination_ by calling `.join()` or `.detach()` on an `std::thread` before it _destructs_!
* Calling `.join()` can lead to performance anomalies while `.detach()` leads to _undefined behavior_

### Item 38: Be aware of varying destructor behavior of thread handle

* `std::future` blocks in _destructor_ if policy is `std::launch::async` by calling an _implicit_ join
* `std::shared_future` blocks when, additionally, the given shared future is the last copy in scope
* `std::packaged_task` doesn't need a _destructor_ policy but the underlying `std::thread` (running it) does

### Item 39: Consider `std::future`s of void type for one-shot communication (comm.)

* For simple comm., `std::condition_variable`, `std::mutex` and `std::lock_guard` is an overkill
* Use `std::future<void>` and `std::promise` for one-time communication between two threads

### Item 40: Use `std::atomic` for concurrency and `volatile` for special memory

* Use `std::atomic` guarantees _thread-safety_ for _shared memory_ while `volatile` specifies _special memory_
* `std::atomic` prevents reordering of reads/write operations but permits elimination of _redundant reads/writes_
* `volatile` specifies _special memory_ (e.g for _memory mapped variables_) which permits _redundant reads/writes_

## Chapter 8. Tweaks

### Item 41: When to use pass-by-value for functions parameters

* Consider _pass-by-value_ for parameters if and only if they are always copied and are cheap to move
* Prefer `rvalue-ref` parameters for move-only types to limit copying to exactly one move operation
* Never use _pass-by-value_ for base class parameter types because it leads to the _slicing problem_

### Item 42: Choose emplacement instead of insertion

* Use `.emplace` versions instead of `.push/.insert` to avoid temp copies when adding to STL containers
* When value being added uses assignment, `.push/.insert` work just as well as the `.emplace` versions
* For containers of _resource-managing types_ e.g smart pointers, `.push/.insert` can prevent memory leaks
* Be careful when using `.emplace` functions because the _args_ passed can invoke explicit _constructors_
