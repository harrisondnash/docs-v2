---
title: Types
description: A type defines the set of values and operations on those values. Types are never explicitly declared as part of the syntax. Types are always inferred from the usage of the value.
menu:
  influxdb_2_0_ref:
    parent: Flux specification
    name: Types
weight: 213
---

{{% note %}}
This document is a living document and may not represent the current implementation of Flux.
Any section that is not currently implemented is commented with a **[IMPL#XXX]** where
**XXX** is an issue number tracking discussion and progress towards implementation.
{{% /note %}}

A **type** defines the set of values and operations on those values.
Types are never explicitly declared as part of the syntax except as part of a [builtin statement](#system-built-ins).
Types are always inferred from the usage of the value.
Type inference follows a Hindley-Milner style inference system.

## Union types
A union type defines a set of types.
In the examples below, a union type is specified as follows:

```js
T = t1 | t2 | ... | tn
```

where `t1`, `t2`, ..., and `tn` are types.

In the example above a value of type `T` is either of type `t1`, type `t2`, ..., or type `tn`.

## Basic types
All Flux data types are constructed from the following types:

### Null types
The **null type** represents a missing or unknown value.
The **null type** name is `null`.
There is only one value that comprises the _null type_ and that is the _null_ value.
A type `t` is nullable if it can be expressed as follows:

```js
t = {s} | null
```

where `{s}` defines a set of values.

### Boolean types
A _boolean type_ represents a truth value, corresponding to the preassigned variables `true` and `false`.
The boolean type name is `bool`.
The boolean type is nullable and can be formally specified as follows:

```js
bool = {true, false} | null
```

### Numeric types
A _numeric type_ represents sets of integer or floating-point values.

The following numeric types exist:

```
uint    the set of all unsigned 64-bit integers | null
int     the set of all signed 64-bit integers | null
float   the set of all IEEE-754 64-bit floating-point numbers | null
```

{{% note %}}
All numeric types are nullable.
{{% /note %}}

### Time types
A _time type_ represents a single point in time with nanosecond precision.
The time type name is `time`.
The time type is nullable.

#### Timestamp format
Flux supports [RFC3339 timestamps](/v2.0/reference/glossary/#rfc3339-timestamp):

- `YYYY-MM-DD`
- `YYYY-MM-DDT00:00:00Z`
- `YYYY-MM-DDT00:00:00.000Z`

### Duration types
A _duration type_ represents a length of time with nanosecond precision.
The duration type name is `duration`.
The duration type is nullable

Durations can be added to times to produce a new time.

##### Examples of duration types
```js
1ns // 1 nanosecond
1us // 1 microsecond
1ms // 1 millisecond
1s  // 1 second
1m  // 1 minute
1h  // 1 hour
1d  // 1 day
1w  // 1 week
1mo // 1 calendar month
1y  // 1 calendar year

3d12h4m25s // 3 days, 12 hours, 4 minutes, and 25 seconds
```

### String types
A _string type_ represents a possibly empty sequence of characters.
Strings are immutable and cannot be modified once created.
The string type name is `string`.
The string type is nullable.

{{% note %}}
An empty string is **not** a _null_ value.
{{% /note %}}

The length of a string is its size in bytes, not the number of characters,
since a single character may be multiple bytes.

### Bytes types
A _bytes type_ represents a sequence of byte values.
The bytes type name is `bytes`.

## Regular expression types
A _regular expression type_ represents the set of all patterns for regular expressions.
The regular expression type name is `regexp`.
The regular expression type is **not** nullable.

## Composite types
These are types constructed from basic types.
Composite types are not nullable.

### Array types
An _array type_ represents a sequence of values of any other type.
All values in the array must be of the same type.
The length of an array is the number of elements in the array.

### Object types
An _object type_ represents a set of unordered key and value pairs.
The key must always be a string.
The value may be any other type, and need not be the same as other values within the object.

### Function types
A _function type_ represents a set of all functions with the same argument and result types.

{{% note %}}
[IMPL#249](https://github.com/influxdata/platform/issues/249) Specify type inference rules.
{{% /note %}}

### Generator types
A _generator type_ represents a value that produces an unknown number of other values.
The generated values may be of any other type, but must all be the same type.

{{% note %}}
[IMPL#658](https://github.com/influxdata/platform/query/issues/658) Implement Generators types.
{{% /note %}}

#### Polymorphism
Flux types can be polymorphic, meaning that a type may take on many different types.
Flux supports let-polymorphism and structural polymorphism.

##### Let-polymorphism
Let-polymorphism is the concept that each time an identifier is referenced, it may take on a different type.
For example:

```js
add = (a,b) => a + b
add(a:1,b:2) // 3
add(a:1.5,b:2.0) // 3.5
```

The identifiers, `a` and `b`, in the body of the `add` function are used as both `int` and `float` types.

##### Structural polymorphism
Structural polymorphism is the concept that structures (objects in Flux) can be
used by the same function even if the structures themselves are different.
For example:

```js
john = {name:"John", lastName:"Smith"}
jane = {name:"Jane", age:44}

// John and Jane are objects with different types.
// We can still define a function that can operate on both objects safely.

// name returns the name of a person
name = (person) => person.name

name(person:john) // John
name(person:jane) // Jane

device = {id: 125325, lat: 15.6163, lon: 62.6623}

name(person:device) // Type error, "device" does not have a property name.
```

Objects of differing types can be used as the same type so long as they both contain the necessary properties.
Necessary properties are determined by the use of the object.
This form of polymorphism means that checks are performed during type inference and not during runtime.
Type errors are found and reported before runtime.