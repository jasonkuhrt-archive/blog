# Writing pairs in TypeScript

2018 Jan 23

The following are some informal murmuring, simplified descriptions, stream of consciousness-esq, upon my experiences of trying to write a simple `pairs` function (transforms an object into a list of tuples) in TypeScript.

## Keyof

Before jumping into `pairs` lets briefly introduce the `keyof` operator in TypeScript. It makes it possible to extract the keys of an object into a union of string literals which can be used as is for a type or used in a type lookup. I am sure there are multiple better and more accurate descriptions out there but that's my stab for now. Example:

```ts
interface Foobar {
  a: string
  b: number
  c: {
    c1: boolean
    c2: RegExp
  }
}

const a: keyof Foobar = "a" // OK
const b: keyof Foobar = "b" // OK
const c: keyof Foobar = "c" // OK
const d: keyof Foobar = "d" // ERROR
```

Trying to assign `"d"` to `d` fails because `keyof Foobar` has told the compiler that `d` may only be `"a" | "b" | "c"`. The TypeScript error says as much:

```
error TS2322: Type '"d"' is not assignable to type '"a" | "b" | "c"'
```

Now lets consider `pairs`.

## Pairs

My implementation is as follows:

```ts
const pairs = <A, B extends keyof A>(a: A): [keyof A, A[B]][] => {
  const mapper = (k: keyof A): [keyof A, A[B]] => [k, a[k]]
  return Object.keys(a).map(mapper)
}
```

Lets break this down.

```
<A, B extends keyof A>
```

We have two type parameters. The second is inferred from the first. TypeScript will try to infer the first one from the types of the caller supplied arguments as well. In practice if the compiler needs a hint the user may well supply a type argument to `A`, but likely never need/want to for `B`.

For example in the following:

```ts
const data = { a: 1, b: 2 }
const a = pairs(data)
```

`a` is correctly inferred as type:

```ts
type a = ["a" | "b", number][]
```

But we won't talk about this return type just yet.

The second type parameter `B` starts by extracting the union of keys from `A` via `keyof` and then storing the result into the identifier `B` for later reference. Note the result here is not a value result but _type_ result [1]. I read `extends` as essentially `=`. I believe there is a lot more to it than this however. I've read in passing descriptions of `extends` as being for "sub-typing", providing "type constraints" etc. But for my immediate purpose this description seems sufficient, if minimally so.

Next:

```ts
(a: A)
```

Simply says that the function parameter is of some type `A`. So far we haven't done anything better than saying `(a: any)` however unlike `any` type parameters lets us build type relationships amongst values in our function making them much more useful and powerful than `any`.

Next:

```ts
: [keyof A, A[B]][] =>
```

We specify our return type. Lets break it down. The `[]` syntax is unfortunately heavily overloaded in meaning:

1. `[,]` - A tuple (the outer first set of `[]`)
2. `[]` - A list (the outer trailing set of `[]`)
3. `keyof A` - Typing the first element of the tuple to be a union of the keys of `A`.
4. `A[B]` - A type lookup into `A` using keys extracted from it and stored previously in `B` as we discussed above. The result here is a union of the values of `A`. Overall this is pretty magical to me. I just accept without much space to poke that a union of key names is passed into a lookup and produces a union of types of the values corresponding to said keys...sure.

Next:

```ts
const mapper = (k: keyof A): [keyof A, A[B]] => [k, a[k]]
//              ^  ^         ^                 ^
//              |  |         |                 | function body
//              |  |         |
//              |  |         | return type
//              |  |
//              |  | parameter type
//              |
//              | parameter
```

We now reach the function body of `pairs`. We define our mapper function rather than using an anonymous function on the next line within `.map(...)`. We do this for clarity because the return type makes things hard to read already let alone if the whole thing were a one-liner. The return type is needed because TypeScript cannot know that the JavaScript array being returned should be interpreted as a tuple rather than an array. If we let TypeScript infer the return type of `mapper` we get:

```ts
((keyof A) | A[keyof A])[]
```

The parens muddy things a bit but what we have here is an array of members whose type is a union of `keyof A` and `A[keyof A]`. But what we want is a tuple like so:

```ts
[keyof A, A[keyof A]]
```

I researched a bit why TypeScript inferrence doesn't pick tuple and found that this point is already treaded e.g. on Github as lately as 30 days ago and described as a `wontfix` by the core TypeScript team due to issues it would/could cause for backwards compatibility and how JavaScript is written in the wild. [Reference](https://github.com/Microsoft/TypeScript/issues/20899#issuecomment-354102474).

Next:

```ts
return Object.keys(a).map(mapper)
```

We read the own enumerable keys from `a`, map each one into a tuple, and finally return. Nothing much to say here. The return type aligns correctly with the overall function's return type, which again is what we saw in `mapper` but now contained in an array via `.map`:

```ts
[keyof A, A[B]][]
```

## Not perfect

Here is an example:

```ts
const myPairs = pairs({ a: 1, b: false, c: "a" })
```

With the return type in this case:

```ts
type MyPairs = ["a" | "b" | "c", string | number | boolean]
```

At first this may look pretty good but unfortunately its not quite right. We have lost the valid combinations of pairs! Clearly incorrect combinations can be allowed without type error:

```ts
myPairs.push(["a", "BUG"]) // This type checks!!
```

But we should not be too surprised as the above type tells us as much. The type we really want returned for the above example is:

```ts
type MyPairs = ["a", string] | ["b", false] | ["c", "a"]
```

At this point I was stuck without really knowing if I hit my limit of TypeScript knowledge or TypeScript's own limitation. I took this uncertainty to [StackOverflow](https://stackoverflow.com/questions/48435581/a-correct-pairs-function-in-typescript-or-flowtype/48437630#48437630) where I got a great answer by a nice fellow named [Tao](https://stackoverflow.com/users/2830131/tao).

## StackOverflow

It turns out TypeScript _can_ achieve my desired result:

```ts
type Pairs<T> = Array<{ [P in keyof T]: [P, T[P]] }[keyof T]>
```

Breaking this down:

1. A type alias called `Pairs` with one type parameter `T`
2. An array type of a mapped type going from `T` to `T2` where `T2` has the same keys but each key's type has been transformed into a tuple type of `[P, T[P]]`.
3. A type lookup on the mapped type just introduced thereby transforming it from an object to a union of all its value types. Yes, we have nested type lookups.

By step three we have almost accomplished our goal. To integrate `Pairs` type into `pairs` function we do this:

```ts
const pairs = <A extends Record<string, any>>(a: A): Pairs<A> => {
  const mapper = (k: keyof A): [keyof A, A[keyof A]] => [k, a[k]]
  return Object.keys(a).map(mapper)
}
```

The only difference from before is:

1. We constrain `A` to being an indexed object. This is just a general improvement not speicfic to Pairs type.
2. We remove `B` type parameter because we no longer need it with `Pairs` type.
3. We wrap our `A` in `Pairs` in the return type

And voila.

A parting thought I've had is about how much the type system in TypeScript makes sense to a developer versus say a mathematician. Indeed it seems that the `pairs` solution in TypeScript relies on "programming" your way through it. But how deep or shallow is this system really? There is a solution, but little else to say about it then "combine type mapping feature with nested use of type lookups". Serviceable! But leaves me wondering a bit about the theory.

[1] Coming from a dynamic language it takes some time to get used to the idea of effectively having a parallel language at play. Haskell for example is literally two langauges, the typed and untyped one, and has a whole set of type-level functions to play with.
