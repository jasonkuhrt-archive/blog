# The bottom of Record Syntax

2015 Nov 17

“Bottom” is a term that means a computation that will never successfully complete. In mathematics Bottom is symbolically represented as ⊥. Cases of bottom include an infinite loop or some program error. Haskell, although safer than many languages, permits writing code that can contain ⊥. This is bad, because it means potentially shipping a bug to the end-users rather than catching a bug at compile time for engineers to fix. Two of the cases of where Haskell can contain ⊥ are discussed below. You can learn more about ⊥ in a Haskell context from the [Haskell Wiki](https://wiki.haskell.org/Bottom).

## The Record Construction Case

In Haskell it is permitted to partially apply Data constructors. For example given this data constructor:

```
> type Size = Integer
> data Colour = Red | Green | Blue deriving (Eq, Show)
> data Garment = Shirt Colour Size deriving (Eq, Show)
```

Shirt has the type of:

```
> :t Shirt
> Shirt :: Colour -> Size -> Garment
```

Indicating that it may be incrementally applied like so:

```
> let RedShirt = Shirt Red
```

whose result has the type of:

```
:t RedShirt
RedShirt :: Size -> Garment
```

Given the above data definition for `Shirt` we cannot use Record Syntax to construct it:

```
> Shirt { colour = Red, size = 5 }
> ‘colour’ is not a (visible) field of constructor ‘Shirt’
> ‘size’ is not a (visible) field of constructor ‘Shirt’
```

But we can if we use Record Syntax to define `Shirt` like so:

```
> data Garment = Shirt { colour :: Colour, size :: Size } deriving (Eq, Show)
```

Data defined with Record Syntax can still be constructed using regular function syntax. One difference between the Function Syntax and Record Syntax is that the former permits incrementally feeding arguments thanks to currying, while the latter must be given all arguments up front or else ⊥ is returned! Also note that while Haskell does compile (bad), at least it warns.

```hs
> let bottomValue = Shirt { colour = Red }

<interactive>:22:17: Warning:
Fields of ‘Shirt’ not initialised: size
In the expression: Shirt {colour = Red}
In an equation for ‘endOfDays’: bottomValue = Shirt {colour = Red}

> bottomValue

Shirt {colour = Red, size = \*\*\* Exception: <interactive>:22:17–38: Missing field in record construction size
```

## The Record Accessor Case

There is another case wherein ⊥ occurs with Record Syntax: Using accessors to read a component of record data wherein the Type of that Record is a Sum Type. For example (note the last line):

```
> data Foo = Bar | Qux { a :: Integer } deriving (Eq, Show)
> let qux = Qux 5

> a qux
> 5

> a Bar
> \*\*\* Exception: No match in record selector a
```

Rather than a compile error there is a runtime error. The problem is related to the fact that `Bar` creates values of type `Foo` just like `Qux`. Correct usage of the accessor `a`, particular to data constructed via `Qux`, is not enforced between the Data Constructors of a Sum Type. But the compiler will catch usage of accessor `a` on other types like we probably expect. For example:

```
> data Moar = Lah

> a Lah

> Couldn’t match expected type ‘Foo’ with actual type ‘Moar’
> In the first argument of ‘a’, namely ‘Lah’
> In the expression: a Lah
```

What is a solution/technique around this issue? A convention to avoid ⊥ suggested in Haskell Programming From First Principals is never define a Record in a Sum Type. For example, below is a safe version of `Qux`:

```
> data Qux = Qux { a :: Integer } deriving (Eq, Show)
> data Foo = Bar | Blah Qux deriving (Eq, Show)

> a Bar

> Couldn’t match expected type ‘Qux’ with actual type ‘Foo’
> In the first argument of ‘a’, namely ‘Bar’
> In the expression: a Bar
```

Much better! We no longer have a runtime error.
