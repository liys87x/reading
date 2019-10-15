# Uber Go Style Guide

## Guidelines

**Pointers to interfaces**:

- You almost never need a pointer to an interface. You should be passing intefaces as values -- the underlying data can still be a pointer
- If you want interface methods modify the underlying data, you must use a pointer
- An inteface is two fields:
  - A pointer to some type-specific information
  - Data pointer
    - if the data stored is a pointer, it's stored directly
    - if the data stored is value, then a pointer to the value is stored

**Recievers and Interfaces**:

- Methods with value receivers can be called on pointers as well as values
  - Methods with pointer receivers can not be called on values
- An interface can be satisfied by a pointer, even if the method has a value receiver
  - An inteface can not be satisfied by a value, when the method has a pointer receiver

**Zero-value Mutexes(sync.Mutex and sync.RWMutex) are valid**:

- You almost never need a pointer to a mutex
  - If you use a struct by pointer, then the mutex can be a non-pointer field
  - use `var mu sync.Mutex`, do not use `mu := new(sync.Mutex)`
- Unexported structs that use a mutex to protect fields of the struct may embed the mutex
- For exported types, use a private fields

**Copy Slices and Maps at Boundaries**:

- Slices and maps contain pointers to the underlying data so be wary of scenarios when they need to be copied
- Keep in mind that users can modify a map or slice you received as an argument if you store a reference to it
  - copy it
- Be wary of user modifications to maps or slices exposing internal state
  - return new map or slice

**Defer to Clean Up**:

- use defer to clean up resources such as files and locks
- Defer has an extremely small overhead 
  - should be avoided only if you can prove that your function execution time is in the order of nanoseconds
- The readability win of using defers is worth the miniscule cost of using them

**Channel Size is One or None**:

- Channels should usually have a size of one or be unbuffered
  - By default, channels are unbuffered and have a size of zero
- Any other size must be subject to a high level of scrutiny

**Start Enums at One**:

- The standard way of introducing enumerations in Go is to declare a custom type and a const group with iota
- Since variables have a 0 default value, you should usually start your enums on a non-zero value
- There are cases where using the zero value makes sence
  - for example when the zero value case is the desirable default behavior

**Error Types**:

- declaring errors:
  - `errors.New` for errors with simple static strings
  - `fmt.Errorf` for formatted error string
  - Custom types that implement an `Error()` method
  - Wrapped errors using `pkg/errors.Wrap`
- return errors:
  - Is this a simple error that needs no extra information? If so, `errors.New` should suffice
    - If the client needs to detect the error, and you have created a simpole error using `errors.New`, use a var for the error
  - Do the clients need to detect and handle this error? If so, you should use a custom type, and implement the `Error()` method
    - Be careful with exporting custom error types directly since they become part of the public API of the package. It is preferable to expose matcher functions to check the error instead
  - Are you propagating an error returned by a downstream function? If so, use error wrapping
  - Otherwise, `fmt.Errorf` is okay

**Error Wrapping**:

- propagating errors options:
  - Return the original error if there is no additional context to add and you want to maintain the original error type
  - Add context using `pkg/errors.Wrap` so that the error message providers more context and `pkg/errors.Cause` can be used to extract the original error
  - Use `fmt.Errorf` if the callers do not need to detect or handle that specific error refused
- It is recommended to add context where possible
- When adding context to returned errors, keep the context succinct by avoiding phrases like "failed to"
  - For example, `new store`, not `failed to create new store`
- However once that error is sent to another system, it should be clear the message is an error

**Handle Type Assertion Failures**:

- Always use the "comma ok" idiom
  - The single return value form of a type assertion will panic on an incorrect type

**Don't Panic**:

- Code running in production must avoid panics
  - A program must panic only when something irrecoverable happens such as a nil dereference. An exception to this is program initialization
  - Even the tests, prefer `t.Fatal` or `t.FailNow` over panics to ensure that test is marked as failed
- Panics are a major source of cascading failures
- If an error occurs, the function must return an error and allow the caller to decide how to handle it
- Panic/Recover is not an error handing strategy

## Performance

> Performance-specific guidelines apply only to the hot path

- Prefer `strconv` over `fmt`: `strconv` is faster than `fmt`
- Do not create byte slices from a fixed string repeatedly. Instead, perform the conversion once and capture the result

## Style

- **Group Similar Declarations**:
  - Go supports grouping similar declarations: constants, variables, and type
  - Only group related declarations. Do not group declarations that are unrelated
  - Groups are not limited in where they can be used. For example, you can use them inside of functions
- **Import Group Ordering**:
  - There should be two import groups: Standard library and Everything else
- **Package Names**:
  - All lower-case. No capitals or underscores
  - Does not need to be renamed using named imports at most call sites
  - Short and succinct. Remember that the name is identified in full at every call site
  - Not plural. For example, `net/url`, not `net/urls`
  - Not "common", "util" "shared" or "lib". There are bad, uninformative names
- **Function Names**:
  - MixedCaps for function names
    - An exception is made for test functions, which may contain underscores for the purpose of grouping related test cases, e.g., `TestMyFunction_WhatIsBegingTested`
- **Import Aliasing**:
  - Import aliasing must be used if the package name does not match the last element of the import path
  - In all other scenarios, import aliases should be avoided unless there is a direct conflict between imports
- **Function Grouping and Ordering**:
  - Functions should be sorted in rough call order
  - Functions in a file should be grouped by receiver
  - Exported functions should appear first in a file, after `struct`, `const`, `var` definitions
  - A `newXYZ()`/`NewXYZ()` may appear after the type is defined, but before the rest of the methods on the reciever
  - Since fucntions are grouped by receiver, plain utility functions should appear towards the end of the file
- **Reduce Nesting**:
  - Code should reduce nesting where possible by handling error cases/special conditions first and returning early or continuing the loop
  - Reduce the amount of code that is nested multiple levels
- **Unnecessary Else**:
  - If a variable is set in both branches of an if, it can be replaced with a single if.
- **Top-level Variable Declarations**:
  - At the top level, use the standard `var` keyword. Do not specify the type, unless it is not the same type as the expression
- **Prefix Unexported Globals with _**:
  - Prefix unexported top-level `var S` and `const S` with `_` to make it clear when they are used that they are global symbols
  - Exception: Unexported error values, which should be prefixed with `err`
  - Rationale: Top-level variables and constants have a package scope. Using a generic name makes it easy to accidentally use the wrong value in a different file
- **Embedding in Structs**: Embedded types(such as mutexes) should be at the top of the field list of a struct, and there must be an empty line separating embedded fields from regular fields
- **Use Field Names to initialize Structs**: 
  - You should almost always specify field names when initializing structs
  - Exception: Field names may be omitted in test tables when there are 3 or fewer fields
- **Local Variable Declarations**:
  - Short variable declarations(`:=`) should be used if a variable is being set to some value explicitly
  - Declaring Empty Slices should use `var`: `var filtered []int`
- **nil is a valid slice**:
  - You should not return a slice of length zero explicitly. Return `nil` instead
  - To check if a slice is empty, always use `len(s) == 0`. Do not check for `nil`
  - The zero value(a slice declared with `var`) is usable immediately without `make()`
- **Reduce Scope of Variable**:
  - Where possible, reduce scope of variables. Do not reduce the scope if it conflicts with `Reduce Nesting`
  - If you need a result of a function call outside of the if, then you should not try to reduce the scope
- **Avoid Naked Parameters**:
  - Naked parameters in function calls can hurt readability. Add C-style comments(`/* ... */`) for parameter names when their meaning is not obvious
  - Better yet, replace naked `bool` type with custom types for more readable and type-safe code. This allows more than just two states(ture/false) for that parameter in the future
- **Use Raw String Literals to Avoid Escaping**:
  - Go supports `raw string literals`, which can span multiple lines and include quotes. Use these to avoid hand-escaped strings which are much harder to read
- **Initializing Struct References**:
  - Use `&T{}` instead of `new(T)` when initializing struct references so that it is consistent with the struct initialization
- **Format Strings outside Printf**:
  - If you declare format strings for `Printf-style` functions outside a string literal. make them `const` values. This helps `go vet` perform static analysis of the format string
  - e.g. `fmt.Printf(msg, 1, 2)`, use `const msg = "unexpected values %v, %v\n"` define `msg`, not `msg := "unexpected values %v, %v\n"`
- **Naming Printf-style Functions**:
  - When you declare a `Printf-style` function, make sure that `go vet` can detect it and check the format string.
  - This means that you should use pre-defined `Printf-style` function names if possible. `go vet` will check these by default
  - If using the pre-defined names is not an option, end the name you choose with `f`: `Wrapf`, not `Wrap`

## Patterns

**Test Tables**:

- Test tables make it easier to add context to error message, reduce duplicate logic, and add new test cases
- Encourage explicating the input and output values for each test case with `give` and `want` prefixes

**Functional Options**:

Functional options is a pattern in which you declare an opaque `Option` type that records information in some internal struct. You accept a variadic number of these options and act upon the full information recorded by the option on the internal struct

Use this pattern for optional arguments in constructors and other public APIs that you foresee needing to expand, especially if you already have three or more arguments on those functions

```go
// BAD

// package db
func Connect(
    addr string,
    timeout time.Duration,
    caching bool,
) (*Connection, erro) {// ...}

// Timeout and caching must always be provided, even if the user wants to use the default
db.Connect(addr, db.DefaultTimeout, db.DefaultCaching)
db.Connect(addr, newTimeout, db.DefaultCaching)
db.Connect(addr, db.DefultTimeout, false /* caching */)
db.Connect(addr, newTimeout, false /* caching */)


// GOOD
type options struct {
    timeout time.Duration
    caching bool
}

type Option interface {
    apply(*options)
}

type optionFunc func(*options)

func (f optionFunc) apply(o *options) {
    f(o)
}

func WithTimeout(t time.Duration) Option {
    return optionFunc(func(o *options) {
        o.timeout = t
    })
}

func WithCaching(cache bool) Option {
    return optionFunc(func(o *options) {
        o.caching = cache
    })
}

func Connect(
    addr string,
    opts ...Option,
) (*Connection, error) {
    options := options{
        timeout: defaultTimeout,
        caching: defaultCaching,
    }

    for _, o := range opts {
        o.apply(&options)
    }
}

// Options must be provided only if needed
db.Connect(addr)
db.Connect(addr, db.WithTimeout(newTimeout))
db.Connect(addr, db.WithCaching(false))
db.Connect(
    addr,
    db.WithCaching(false),
    db.WithTimeout(newTimeout),
)
```
