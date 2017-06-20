## Built-in types

### Integers

Ante supports 5 signed and unsigned integer primitives:
 * 4 are platform-independent, sized at
8, 16, 32, and 64 bits respectively, of the form `[iu](8|16|32|64)`, where `i` indicates signed
and `u` indicates unsigned. 
 * The last is sized to the platform bit-size i.e. 32 bits on a 32-bit 
platform, and so on: these are of the form `[iu]sz`.

A numeric literal can be followed by an optional underscore an an integer type name to explicitly
type it.

### Characters

There are two character primitives: `c8`, for a byte character, and `c32` for a four-byte character.

### Floats

There are three float primitives: `f16`, `f32`, and `f64`.

### Booleans

Ante supports booleans with the `bool` type.

### Strings

Strings are supported through the `Str` type.

### Tuples

Tuples are groups of values, typed by their size and contents. A tuple can be created by placing values in
brackets

`(<val1>, <val2>, ...)`

where all the values can be of different types. Each element of the tuple can then be accessed by its index,
using the `#` operator. The type of a tuple can be written in a similar way:

`(<type1>, <type2>, ...)`

## User-constructed types

### Basic types

Ante lets you build basic types as combinations of variables. The basic syntax is

`type <Typename> = <variables>`

where the variables section is formatted the same way function paramaters are - that is

`(<type> <variable name>*)(, <type> <variable name>*)*` 

These variables can then be accessed on an instance of a basic type using `.<variable name>`.

#### Example

    type ThreeInts = i32 first second, i64 third

    fun get_third: ThreeInts t -> i64
        t.third

### Tagged Unions

There's also support for unions of basic types. Each part of the union is made up of a tag and an
optional list of one or more types separated by `,`, of the form

`type <Typename> = (| <tag> (<typename>(,<typename>)*)?)+`

An instance of the union type is then made up of one of the possible tags and a value for each of
the types in the corresponding type list, in order. It's only possible to extract the value from 
an instanceby pattern-matching on the tag and assigning names to each value.

    type ABC = | A u8 | B Str, bool | C

    fun printABC : ABC abc
        match abc with
        | A int -> printf "%d\n" int
        | B string b -> printf "%s if %b\n" string b
        | C -> print "Got a C!\n"

### Recursive types

If a type contains an instance of itself somewhere in its body, that instance must be a *pointer* (see
below). 

#### Example

    type IntOrString = Int i32 int
        | String Str string
 
### Type parameters

A typename can be followed a set of angle brackets inside which you can specify one or more type parameters,
which are variables preceded by a single quote mark. Type parameters can then be used in the body of the type
in place of where a type would normally appear - such as a variable, but also as a parameter in another type.

If a type has any type parameters, they need to be specified before it can be used, either by a concrete type
or by a type parameter of the current type.

#### Example

    type Box<'t> = 't contents
    type BoxedInt = Box<i32> intcontents
    type BoxedBox<'t> = Box<'t> box

### Functions on types

A special syntax exists to create functions which can be called on type instances, such as 
`instance.function(arguments)`. The function name has to start with the type, followed by a dot, followed by
the name itself. The first argument then has to be of the type.

#### Example

    fun Box<'t>.get_contents: Box<'t> b -> 't
        b.contents

## Traits

Traits are polymorphic constructs, used for specifying functions on types - they're the equivalent to classes
or interfaces in other languages.

A trait is declared as `trait <Typename>` followed by a set of function signatures, which look like functions
with no code body.

In order to use a type as a trait, you have to declare the implementation by `ext <Typename> : <Traitname>`,
followed by an implementation of each function signature in the trait, with the trait name replaced by the
type name.

Traits can also have type parameters, similar to types. If a type implements a trait with a type parameter,
it has to specify the value of the parameter, either through a type or a parameter of the type. Wherever the
trait's parameter then appears in the functions, it should be replaced appropriately.

#### Example

    trait GetInt
        fun get_value: GetInt i -> i32
    
    ext i32 : GetInt
        fun get_value: i32 i -> i32
            i
    
    trait GetType<'t>
        fun get_value: GetType<'t> g -> 't
    
    ext i32 : GetType<i32>
        fun get_value: i32 g -> i32
            g
    
    ext Box<'t> : GetType<'t>
        fun get_value: Box<'t> g -> 't
            g.contents

## Arrays

The type system in Ante includes sized arrays, where the size is passed around to functions as part of the type.
The general form of an array is `[<size> <type>]`. Arrays support an indexing operator `#<n>`, which can be used to
access and set the elements of the array. Arrays are zero-indexed.

#### Example

    fun print_all : [3 u8] array
        printf "%d" (array#0)
        printf "%d" (array#1)
        printf "%d" (array#2)

## Pointers

Ante also supports pointers of types, denoted by a type followed by a `*`. A pointer can be dereferenced
using the deref operator `@`, which can be used to get and set its value. Pointers can also be accessed at
an offset using the index operator, `#`, similarly to an array.

An instance of a pointer to an existing value can be created using `new`. 

#### Example

    let mut pointer = new 46_i32
    /* 46 */
    printf "%d" (@pointer)

    @pointer = 21_i32
    /* 21 */
    printf "%d" (@pointer)

    pointer#0 = 35
    /* 35 */
    printf "%d" (@pointer)

## Function types

It's possible to treat functions as values as well, and pass them to other functions. The type of a function
is written as a list of types, separated by commas, followed by the return type - if the function returns nothing,
the explicit `void` type is used.

#### Examples

    type Maybe = | Some u8 | None
    
    fun filter: Maybe m, u8->bool filter -> Maybe
        match m with
        | Some value -> if (filter value) then (Some value) else (None)
        | None -> None

    fun is_even : u8 value -> bool
        value % 2 == 0

    /* None */
    filter (Some 3_u8) (is_even)
    /* Some 6 */
    filter (Some 6_u8) (is_even)

