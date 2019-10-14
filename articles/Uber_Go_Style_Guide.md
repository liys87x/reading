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

**Error Wrapping**:

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

## Style

## Patterns
