# Effective Modern C++ Cheatsheet

## Chapter 1. Deducing Types

### Item 1: Understand `template` type deduction

* Deduced type of T doesn't always match that of the parameter (i.e ParamType) in template fns
* With _l-ref/r-ref_ args, compiler ignores reference-ness of an arg when deducing the type of T
* With _u-refs_, type deduction always distinguishes between _l-values_ and _r-values_ args
* With _pass-by-value_, _ref-ness_, `const` and `volatile` are ignored if present in ParamType
* args that are raw array or fn types always decay to ptr types unless they initialize references

### Item 2: Understand `auto` type deduction

* `auto` plays the role of T while its type specifier (i.e including `const` or _ref_) as ParamType
* `auto` always deduces `std::initializer_list<T>` for braced initializer e.g `{1, 2, 3}`
* `auto` as function or lambda `return` type uses _template type deduction_, not _auto type deduction_

### Item 3: Understand `decltype`

* `decltype` on vars/exprs yields correct type mostly; it yields _l-value_ ref for _l-values_ exprs only
* `decltype(auto)` includes _ref-ness_ when deducing `return` type of a fn or lambda from definition

### Item 4: Know how to view deduced types

* Update your code so that it leads to a compilation failure and you will see the type in diagnostics
* `std::type_info::name` depends upon compiler implementation; use Boost.TypeIndex library

## Chapter 2. `auto`

### Item 5: Prefer `auto` decalartions

* `auto` prevents verbose declarations, uninitialized vars, and directly holds closures (lambdas)
* Use `auto` esp. in place of `std::vector<T>::size_type` and `std::unordered_map<T>::key_type`
  
### Item 6: How to fix undesired `auto` type deduction

* Use `auto` with `static_cast` (a.k.a _explicitly typed initializer idiom_) to get correct types
* Never use `auto` directly with _invisible proxy classes_ such as `std::vector<bool>::reference` 

## Chapter 3. Moving to Modern C++

### Item 7: Distinguish between () and {} when creating objects

* Braced (a.k.a uniform) initializer helps to prevent narrowing conversions and most vexing parse
* For overload-resolution, compiler always prefers `std::initializer_list` for braced initializer

### Item 8: Prefer `nullptr` to `0` and `NULL`

* Don't use `0` or `NULL`, use `nullptr` of type `nullptr_t` which represents pointers of all types!
  
### Item 9: Prefer alias declarations to `typedefs`

* Alias declarations use `using` keyword and support templatization while `typedefs` don't
* Alias declarations avoid 1) `::type` suffix 2) `typename` prefix when referring to other typedefs

### Item 10: Prefer _scoped_ `enums` to _unscoped_ `enums`

* Use `enum class` instead of `enum` to limit scope of an `enum` members to just inside the `enum`
* `enum class` uses `int` type by default, prevents _implicit convs_ and allows fwd declarations

### Item 11: Prefer deleted functions to private undefined ones

* Make unwanted functions (such as _copy-ctors_ for move-only types) `public` as well as `delete`

### Item 12: Always declare overriding functions `override`

* Declare overriding fns in derived types `override`; use `final` to prevent further inheritance

### Item 13: Always prefer `const_iterators` to `iterators`

* Prefer `const_iterators` to `iterators` for all STL containers e.g `cbegin` instead of `begin`
* For max generic code, don't assume the existence of member `cbegin`; use `std::begin` instead

### Item 14: Declare functions `noexcept` iff they won't emit exceptions

* Declare fns `noexcept` when they don't emit exceptions e.g fns that use _wide contracts_
* Always use `noexcept` for move-operations, `swap` functions and memory allocation/deallocation
* When a `noexcept` fn emits an exception: stack is _possibly_ wound and program is terminated

### Item 15: Use `constexpr` whenever possible

* `constexpr` objs are `const` that are known at compile time; not all `const`s are `constexpr` tho
* `constexpr` functions will produce results at compile time if their args are known during compile
* Declaring objs and fns `constexpr` allows you to use your types at compile as well as runtime

### Item 16: Make `const` member functions thread-safe

* Member fns that do not modify members of a type should be made `const` and then `thread-safe`
* For synchronization, consider `std::atomic` first and then move to `std::mutex` when required

### Item 17: Understand special member function generation

* _Default ctor_ is genrated iff no other _ctor_ declared; most generated fns are `public`/`inline`
* Declaring _dtor_ and/or _copy ops_ disables generation of default _move ops_ and vice versa
* _Copy assignment operator_ is generated if: 1) not already declared 2) no move op is declared

## Chapter 4. Smart Pointers

### Item 18: Use `std::unique_ptr` for exclusive-ownership of resource management

* `std::unique_ptr` owns what it points to, is fast as raw ptr (`*`) and supports custom deleters
* Always return `std::unique_ptr` from factory fns; converting it to a `std::shared_ptr` is easy
* `std::array`, `std::vector` and `std::string` are generally better choices than raw arrays `[]`

### Item 19: Use `std::shared_ptr` for shared-ownership resource management

* `std::shared_ptr` points to an object with _shared ownership_ but doesn't own the object
* `std::shared_ptr` stores/updates _metadata_ on heap and can be 2x slower than `std::unique_ptr`
* Unless you want custom deleters, prefer `std::make_shared<T>` for creating shared pointers
* Don't create multiple `std::shared_ptr`s from a single raw ptr; it leads to _undefined behavior_
* For a `std::shared_ptr` to `this`, inherit your class type from `std::enable_shared_from_this`

### Item 20: Use `std::weak_ptr` for `std::shared_ptr`-like ptrs that can dangle

* `std::weak_ptr` operates with the possibility that the obj it points to might have been destroyed
* `std::weak_ptr::lock()` returns `nullptr` for _destroyed objs_, else `std::shared_ptr` always
* `std::weak_ptr` can be used for caching, observer lists and for prevention of shared ptrs cycles

### Item 21: Prefer `std::make_unique` and `std::make_shared` to direct use of new

* `make` functions remove src code duplication, improve exception safety and are faster (sometimes)
* `make` are bad for custom deleters, large mem objs and `std::weak_ptr`s that outlive _shared ptrs_
* Do not mix _overloading_, _braced initializer_ and `std::initializer_list`; use `new` if you do
* When using `new` (in any case), prevent mem leaks by immediately passing it to a _smart ptr_ always

### Item 22: When using Pimpl idiom, define special mem fns in an implementation file

* _Pimpl idiom_ puts members of a class type inside an impl (`struct Impl`) and stores a ptr to it
* For `std::unique_ptr<Impl>`, always _implement_ your copy, move and dtor ops in an impl file
* With `std::shared_ptr<Impl>`, no need to do that; shared ptr is roughly just as efficient here

## Chapter 5. Rvalue references, move semantics and perfect forwarding

* _Move semantics_ usually replace expensive copy ops (e.g _copy ctor_) with cheaper move ops
* _Perfect forwarding_ forwards a fn's args to other fns params while strictly preserving types

### Item 23: Understand `std::move` and `std::forward`

* `std::move` performs an unconditional cast to an _rvalue_; you can then perform _move ops_
* `std::forward` casts its _input arg_ to an _rvalue_ only if the _arg_ is bound to an _rvalue_ name

### Item 24: Distinguish universal refs (u-refs) from rvalue refs

* U-refs (occur in `T&&` and `auto&&`) _cast_ lvalues to lvalue refs and rvalues to rvalue refs
* For refs to be universal, _type deduction_ and _non-`const`ness_ of the param type is a pre-req

### Item 25: Understand when to use `std::move` and `std::forward`

* Universal references are usually a better choice than overloading fns for _lvalues_ and _rvalues_
* Apply `std::move` on _rvalue refs_ and `std::forward` on _universal-refs_ last time each is used
* Similarly, also apply `std::move` or `std::forward` accordingly when returning by value from fns
* Never return local objs in fns using `std::move`; it can prevent _return value optimization (RVO)_
  
### Item 26: Avoid overloading on universal references

* _Universal-refs_ should be used when client's code could pass either _lvalue refs_ or _rvalue refs_
* Fns overloaded on _u-refs_ usually get called more often than expected so you should avoid them
* Avoid perf-fwding ctrs; they are usually better matches for non-`const` values than copy/move ops

### Item 27: Alternatives to overloading universal-references

* _Ref-to-const_ works but is less efficient while _pass-by-value_ works but only for copyable types
* Tag dispatching (e.g using `std::true_type`) takes _u-refs_ and a second arg to help in matching
* Templates using `std::enable_if_t` and `std::decay_t` also work for _u-refs_ and they reads nicely
* Generally, _u-refs_ have efficieny advantages but they sometimes suffer from usability disadvantages

### Item 28: Understand reference collapsing

* Reference collapsing converts `& &&` to `&` (i.e lvalue ref) and `&& &&` to `&&` (i.e rvalue ref)
* Reference collapsing occurs in `template` and `auto` type deductions, alias declrs and `decltype`

### Item 29: Assume that move ops are not present, no cheap, and not used

* Generally, _moving_ objs is usually much cheaper then _copying_ them e.g heap-based STL containers
* For some types e.g `std::array` and `std::string` (with SSO) _copying_ them can be just as efficient

### Item 30: Perfect forwarding failure cases

* _Perf-forwarding_ fails when _template type deduction_ fails or deduces wrong type for the arg passed
* Careful with braced initializers, 0 or NULL for `nullptr` and declr only integral const static members
* Similarly, `template` and overloaded fn names; altho you can use `static_cast` to resolve fn overloads
* Finally, passing bit-field to perf-fwding fns is problematic because refs to bitfields don't exist

## Chapter 6. Lambda Expressions

### Item 31: Avoid default capture modes

* Avoid default `&` or `=` captures for lambdas cause they can easily lead to dangling refs
* Fail cases: `&` when they outlive the objects captured, `=` for mem types when they outlive `this` 
* Compiler captures `static` types by `ref` even though default capture-mode could be _by-value_

### Item 32: Use init-capture to move objects into (lambda) closures

* Often called _generalized lambda captures_ allow you to init objs inside a lambda capture expr

### Item 33: Use `decltype` on `auto&&` params for `std::forward`

* Use `auto&&` deduction in lambda's params for using `std::forward` to fwd to other functions

### Item 34: Prefer lambdas to `std::bind`

* No convincing use-use for `std::bind` after _init capture_ based `lambdas` were introduced

## Chapter 7. Concurrency API

### Item 35: Prefer task-based programming to thread-based

* `std::thread` acts as handle to an OS thread so _you_ need to manage scheduling and oversubscription
* Use `std::async` (aka task) with default launch policy to handle most of the corner cases for you
  
### Item 36: Specify `std::launch::async` for truly asynchronous tasks

* `std::async`'s default policy can make it run either async (new thread) or sync (i.e upon `.get()`)
* For `std::future_status::deferred` on `.wait_for()`, you need to always call `.get()`

### Item 37: Always make `std::threads` unjoinable on all paths

* You need to either call `.join()` or `.detach()` on an `std::thread` before it _destructs_
* In _dtor_, calling `.join()` leads to performance anomalies while `.detach()` _undefined behavior_

### Item 38: Be aware of varying destructor behavior of thread handle

* `std::future` blocks in _dtor_ if policy is `std::launch::async` by calling an _implicit_ join
* `std::shared_future` blocks when, additionally, the given shared future is the last copy
* `std::packaged_task` doesn't need a _dtor_ policy; underlying `std::thread` (running it) does

### Item 39: Consider void `std::future`s for one-shot communication (comm.)

* For simple comm., `std::condition_variable`, `std::mutex` and `std::lock_guard` is an overkill
* Use `std::future<void>` and `std::promise` for one-time communication between two threads

### Item 40: Use `std::atomic` for concurrency and `volatile` for special memory

* `std::atomic` makes R/W thread-safe and prevents the reordering of R/Ws on atomic types
* `volatile` tells compiler it's _special memory_ and compiler doesn't optimize redundant R/Ws
* `std::atomic` doesn't support copy or move ops but you can use `.load()` and `.store()` fns.
* Use `volatile` for _special memory_ and `std::atomic` for managing access to _shared memory_

## Chapter 8. Tweaks

### Item 41: Consider pass-by-value for params which are always copied and cheap to move

* Consider _pass-by-value_ for always-copied parameters instead of pass by `l/r/u-refs` in fns.
* Prefer `r-refs` parameters for move-only types to limit copying to exactly one move operation
* Do not _pass-by-value_ for base class parameter types cause it will lead to the _slicing problem_

### Item 42: Choose emplacement instead of insertion

* Use `.emplace` versions instead of `.push/.insert` to avoid temp when adding to STL containers
* When value being added uses assignment, `.push/.insert` work just as well as `.emplace` versions
* In a cont. of resource-managing types e.g unique_ptr, `.push/.insert` can prevent corner cases
* Whn using `.emplace` functions, be careful with _args_ cause they can invoke explicit _ctors_

## Abbreviations

1. **ref(s)**: reference(s)
2. **l-refs**: lvalue references
3. **r-refs**: rvalue references
4. **u-refs**: universal references
5. **objs**: objects
6. **fns**: functions
7. **op(s)**: operation(s)
