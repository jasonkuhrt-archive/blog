# Today I Learned

#### About

Also known as TIL, this stream records some of my personal learnings at a granular level. This acts a way to improve retention, share what I have been doing, motivate or help maintain my learning momentum, act as a track record of discipline (or lack thereof), and in the large contribute to a backlog of data that may help me better know thyself in the future.

At the risk of stating the obvious this format is nothing new. Surely most thinkers (great or otherwise) from ancient to modern times have maintained some sort of log, journal, notebook, or whatever. In my contemporary setting I have had direct influence from others practicing this kind of TIL e.g. [jbranchaud](https://github.com/jbranchaud/til/commits/master), [thoughtbot](https://github.com/thoughtbot/til), [milooy](https://github.com/milooy/TIL).

## 2018 Mon Mar 12

##### Carefully `alter`

Had some kind of issue with a MySQL table today. Possibly because of a bad `ALTER`. Attempted a column alteration from `VARCHAR` to `INT` with length `11` but my db client application (MacOS, Sequel Pro) hung at this point. I force quit but since then I think the table has been unloadable. Attempting to access says loading indefinetely. Will investigate solution tomorrow but lesson today. Next time doing alter:

1. copy table structure to new temp table
2. make alterations
3. copy data
4. delete old table
5. rename new table to old

##### Haskell `let` v `where`

Haskell has two ways of binding variables `let` and `where`. `let` is an expression and `where` is a declaration. The difference is `let` can go wherever an expression can go. For example:

```haskell
Prelude
𝝺 a = let b = 2 in b

Prelude
𝝺 a
2

Prelude
𝝺 b

<interactive>:5:1: error: Variable not in scope: b
```

1. We assign an expression to `a` which itself contains a let binding assigned `2` to `b`. We just return `b` there.
2. We prove that `a` has been assigned the bound value to `b`
3. We prove that the binding `b` was local to the expression within assignment to `a`.

On the other hand since `where` is bound to syntactic structure it can provide some unique affordances to the user. For example it introduces variables across function pattern matches:

```haskell
Prelude
𝝺 :{
Prelude| foobar 1 = 1 + a
Prelude| foobar 2 = 2 + a
Prelude| foobar _ = a
Prelude|   where a = 10
Prelude| :}

Prelude
𝝺 foobar 1
3

Prelude
𝝺 foobar 2
4

Prelude
𝝺 foobar 3
10
```

Note how the `a = 10` binding was in scope for all three pattern-matched definitons of `foobar`.

Regarding this topic there is considerably more detail on the [Haskell Wiki](https://wiki.haskell.org/Let_vs._Where). It seems that the main advantage of `where` is its readability. Even the ability to share `where` across multiple function pattern matches can be achieved somewhat using `case-of`.

##### Lambda Lifting

Turn `Free Variables` (those not found in the function's parameters) into arguments. For example:

```haskell
foo a =
  a + b
  where
    b = a + 1
```

```haskell
foo a =
  a + b a
  where
    b a = a + 1
```

## 2018 Thu Mar 8

* Got scolded today for hyping instead of doing. Good reminder. Show, don't tell, and be your own primary source rather than citing opinions of others.

##### Forto + Rollup

Refactoring some util functions in `forto`. Realized I could be getting functions like `mapObject` from a library like `ramda` without bloating `forto` with their deps. Question was, how to make the libarary calls be inlined?

```ts
import rollupTypesript from "rollup-plugin-typescript"
import typescript from "typescript"
import resolve from "rollup-plugin-node-resolve"
import * as F from "ramda"

const pkg = require("./package.json") // [1]
const external = F.keys(F.omit(["ramda"], pkg.dependencies)) // [2]

export default {
  input: "source/Main.ts",
  plugins: [
    rollupTypesript({
      typescript,
    }),
    resolve(),
  ],
  external, // [3]
  output: [
    {
      file: pkg.main,
      format: "umd",
      name: pkg.name,
      sourcemap: true,
    },
    {
      file: pkg.module,
      format: "es",
      sourcemap: true,
    },
  ],
}
```

1. Read in `package.json` to get access to all the dependency names
2. Read said deps, but omit ramda
3. Expose this list of dep names to rollup. What Rollup does in turn is not try to tree shake imports coming from these packages. However since we did not mark `ramda` as external, all `ramda` imports will be tree shaken which is exactly what we want for that dep.

Still, there are hundrends upon hundreds of lines of code added. For a lean library this still seems unacceptable. I also tried to see if I would fare better with lodash but instead the situation is actually [more complex](https://medium.com/@martin_hotell/tree-shake-lodash-with-webpack-jest-and-typescript-2734fa13b5cd).

## 2018 Mon Mar 5

##### hub issue

* Finally played with `git issue` command via `hub`
* Up to now had only even played with `gh is`
* Nothing against `gh` but nice to have `issue` available in `hub` so that I don't _need_ node etc. deps for this functionality
* Also seems fastesr than I remember `gh is` being though unclear if its IO related e.g. Github API got faster

![git-issue command output](./assets/git-issue.png)

## 2018 Sun Mar 4

* on my mind lately:
  * Just Do It
  * Graph theory and graph implementations for brand adjacency feature
  * How does an organization govern its technical evolution (technique, tools, ...)
  * Use questionaries to drive own personal profile page
  * Use aspects of profile I might want to explain or share as basis for blog posts
  * Make a PR to immer to convert to TypeScript [as per request](https://github.com/mweststrate/immer/issues/67)
  * tree shake ramda functions at library level rather than reinventing wheel
  * Polymath Inc
  * Solo blog to group blog to freelance to consultancy to service company to startup to freedom
  * Use my repeatedly validated visualization skills to explore and teach technical topics

## Wed Nov 15

* Have been playing with Most.js again recently. A new engine published as [`@most/core`](http://mostcore.readthedocs.io/en/latest/) recently hit 1.0 and will be the basis for `most@2.0`.

* Most is a functional reactive programming library with a few concepts: events, event streams, sinks, schedulers. Events are conceptually tuples of time and value. Event streams are time ordered sequences of events. Sinks are internal to the implementation of a stream and are where events go into in order for a type of stream to get at and process the event. A stream is a chain of different operations each with a sink. When the whole stream is run the run command works up the chain to the source, the source begins producing events which it feeds to the sink of the next operator which in turn the next operator does as well and so on such that there is an internal chain of sinks. Schedulers provide time facilities so that a custom scheduler can alter the entire timing of the stream. Some additional concepts supporting schedulers include Timer Timeline Clock Task and ScheduledTask.

## Sat Oct 28

* Occam's Razor is a principal that when problem solving no more assumptions should be made than necessary. It states that given a set of hypotheses the one which makes the fewest assumptions should be selected. In practice, in science, it is used as a heuristic guide in the development of theoretical models, rather than a rigorous rule forcing selection between models.

## Thur Oct 26

* Defining the terms of lambda calculus is highly inconsistent in the material I have been reading. In http://haskellbook.com/[Haskell book] the basic lambda terms are: expressions, abstractions, variables. In [this article](https://plato.stanford.edu/entries/lambda-calculus/) and [wikipedia](https://en.wikipedia.org/wiki/Lambda_calculus) they are: variables, abstractions, applications. In [this article](http://www.inf.fu-berlin.de/lehre/WS03/alpi/lambda.pdf) they are: expressions, abstractions, applications.

## Sat Oct 7

* In TypeScript using an interface versus using a type alias changes how the type is represented in IDE type tooltips. With interfaces, tooltips will show the name of the interface. For example `interface Foo {}` will show as `Foo`. However with type aliases the tooltips will contain the contents of the alias. For example `type Foo = { a: string }` will show as `{ a: string }`. I prefer the way interfaces are presented because its much more readable than having the guts of many fields splayed into a tooltip. More details can be found in [this SO thread](https://stackoverflow.com/questions/37233735/typescript-interfaces-vs-types). Noted in this thread are two additional benefits of interfaces in TypeScript:

** An interface can be named in an extends or implements clause, but a type alias for an object type literal cannot.
** An interface can have multiple merged declarations, but a type alias for an object type literal cannot.

However at least one advantage of type aliases in TypeScript is that they can alias anything (primitives, unions, etc.), not just objects.

* [AsciiDoctor has support for list continuations of an ancestor list](http://asciidoctor.org/docs/asciidoc-writers-guide/#attaching-to-an-ancestor-list).

* [mermaidjs](https://mermaidjs.github.io) seems like an awesome text-based diagraming tool

* regarding [`interface`](https://flow.org/en/docs/types/interfaces/) versus `type` in Flowtype it appears that the differences are not nearly as significant or different as in TypeScript, https://stackoverflow.com/questions/43023941/flow-interfaces-versus-types[link]. It seems they are or will become identical. I like this because of the simplicity factor. More power-to-weight ratio here than in TypeScript.

## Sun Oct 1

* To upgrade `yarn` dependencies to latest simply run `yarn upgrade`. The lock file will be updated but the semver ranges in package.json will remain unchanged.

## Tue Sep 19

* In Go command line flags are processed such that `-a=x` `--a=x` `-a x` `--a=x` all mean the same thing. The only exception is that booleans `true`/`false` cannot be used with the form `-a x` or `--a x`.

* It turns out that in k8s deployment configs `args` field (inside container spec) given flags with spaces between name and value does not work (neither would if we wrote this in `["...", ...]` style too):

  ```yaml
  args:
    - --group_conf /dgraph-config/group-mappings
    - --groups "0,1"
    - --idx 1
    - --bindall=true
    - --memory_mb 2048
  ```

* It turns out in dgraph the `--groups` flag doesn't require quotes around the arguments so `--groups="0,1"` can be entered as just `--groups=0,1`. This point matters because in the k8s deployment spec the former with quotes leads to a parse error in dgraph! This wasted several hours of my time...

  ```yaml
  apiVersion: apps/v1beta1
  kind: Deployment
  metadata:
  name: dgraph
  labels:
    app: dgraph
  spec: # ReplicaSet
  replicas: 1
  template: # Pod
    metadata:
      labels:
        app: dgraph
    spec:
      volumes:
        - name: group-mappings
          configMap:
            name: dgraph-group-mappings.conf
        - name: dgraph
          persistentVolumeClaim:
            claimName: mypvc
      containers:
        - name: dgraph
          image: dgraph/dgraph:latest
          ports:
            - name: ui
              containerPort: 8080
            - name: client
              containerPort: 9080
          command:
            - dgraph
          args:
            - --group_conf=/dgraph-config/group-mappings
            - --groups="0,1"
            - --idx=1
            - --bindall=true
            - --memory_mb=2048
          volumeMounts:
            - name: dgraph
              mountPath: /dgraph
            - name: group-mappings
              mountPath: /dgraph-config
  ```

  ```
  ❯ kc logs dgraph-3594762630-2bfm3

  Dgraph version : v0.8.1
  Commit SHA-1 : 8ea4b0a5
  Commit timestamp : 2017-09-18 10:56:37 +1000
  Branch : HEAD

  2017/09/20 01:39:09 Unable to parse 'groups' configuration
  ```

## Mon Sep 18

##### running a dgraph cluster

* multiple dgraph instances can run together in a cluster to scale read/write performance as well as provide high availability of the data
* additional configuration steps are necessary to setup a cluster
* each instance must be instructed to take on certain `group`s of predicates `--groups`
* a predicate-to-group mapping spec must be given to each instance `--group_conf`
* The mapping spec has some basic features like edge-name-prefix matching and `fp` variable for the so-called fingerprint of an edge

##### k8s

* in Kubernetes a ConfigMap resource type allows mounting file contents at file names given by the resource spec key names. for example:

  ```yaml
  spec:
    <filename_1_here>: |
      file 1 contents here!
    <filename_2_here>: |
      file 2 contents here!
  ```

## Sun Sep 17

##### In Kubernetes it is possible to maintain persistent data

* The core concepts are:

  1. `volume classes` (`vc`)
  2. `persistent volumes` (`pv`) based on `vc`
  3. persistent volume claims`(`pvc`) based on`pvc`.`pod`specs that specify volumes based on`pvc`
  4. `container`specs (within pod specs) that specify`volumeMounts`based on` pod``volumes `

* So each of these is based upon the former until `vc` hits the raw layer of whatever the kubernetes is hosted upon.

* Since Kubernetes 1.6 it has been possible to create `pvc`s directly without needing to first create `pv`s. This is referred to as dynamic provisioning. [link](http://blog.kubernetes.io/2017/03/dynamic-provisioning-and-storage-classes-kubernetes.html)

* More about the concept can be read on the [storage docs](https://kubernetes.io/docs/concepts/storage/volumes/). The third sub-section just links to a blog post, so incomplete?

* A blog post going over the topic in an end-to-end example can be found [here](http://blog.bigbinary.com/2017/04/12/using-kubernetes-persistent-volume-for-persistent-data-storage.html)

* Another example is [one section on the k8s task-oriented [docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

* Example:

  ```
  ❯ kc get pv
  NAME CAPACITY ACCESSMODES RECLAIMPOLICY STATUS CLAIM STORAGECLASS REASON AGE
  pvc-46c2c0df-9c19-11e7-a0d1-0800271d32bc 1Gi RWO Delete Bound default/mypvc standard 47m

  ❯ kc get pvc
  NAME STATUS VOLUME CAPACITY ACCESSMODES STORAGECLASS AGE
  mypvc Bound pvc-46c2c0df-9c19-11e7-a0d1-0800271d32bc 1Gi RWO standard 47m

  ❯ cat ./deployment.yaml
  apiVersion: apps/v1beta1
  kind: Deployment
  metadata:
  name: dgraph
  labels:
  app: dgraph
  spec: # ReplicaSet
  replicas: 1
  template: # Pod
  metadata:
  labels:
  app: dgraph
  spec:
  volumes: - name: dgraph
  persistentVolumeClaim:
  claimName: mypvc
  containers: - name: dgraph
  image: dgraph/dgraph:latest
  ports: - name: ui
  containerPort: 8080 - name: client
  containerPort: 9080
  command: - dgraph
  args: - -bindall=true - -memory_mb=2048
  volumeMounts: - name: dgraph
  mountPath: /dgraph
  ```

## Fri Sep 15

* installing dgraph https://docs.dgraph.io/get-started/#from-install-scripts[via simple bash script] makes not just `dgraph` available on the command line but also `dgraphloader`.
* data can be imported and exported out of dgraph using a file format called RDF. RDF stands for https://en.wikipedia.org/wiki/Resource_Description_Framework["resource description framework"]. It is actually a family of specifications maintained by the https://en.wikipedia.org/wiki/World_Wide_Web_Consortium[W3C]. N-Tripples are one of the common serialization formats for RDF data, and not coincidentally as I noted a few days ago tripples are also a W3C specification. The main enlightenment here was that I realized dgraph `set` syntax (`mutate { set { ... }}`) isn't its own design but rather just RDF. In fact the contents of an RDF file can be copy-pasted into this `set` block! It is not clear if the reverse is true. In otherwise RDF may just be a subset of what dgraph `set` can do.
* In dgraph there are no properties on nodes, just named edges to types of data
* In dgraph up until today it was only possible to have multiple outgoing node edges of the same name to other _nodes_, but not to other _values_. So for example if you had a product node it was not possible to attach multiple `image` edges to URL values. Each attachment would just override the previous one. On the other hand a person node could have multipe `friend` edges to other person nodes. However today a feature landed in `master` that allows multiple same-named edges to values just like nodes! https://dgraph.slack.com/archives/C13LH03RR/p1505509178000026[link]
* dgraph has an interface for making queries and visualizing their results

![dgraph-ui](./assets/dgraph-ui.png)

* a dgraph schema is a non-nested map of edge names to types. The types are the type of value pointed _to_ by that edge. There are no namespaces. when we add `@index` to the typing we're making _any_ node with an _outgoing_ edge of the respective name available as an entry point (e.g. `foobar(func: allofterms(some_edge_here, "some value here"))`) or for filtering (e.g. `friend @filter(allofterms(some_edge_here, "some value here")) { ... }`).
* dgraph `@filter` and entrypoint are two syntaxes for doing the same thing it seems e.g. they each accept the same functions `allofterms` `anyofterms` `eq` ...
* When specifying a field in the schema design `@reverse` makes it possible to use `~field_name_here { ... }` in queries which will follow the edge back to where its pointing _from_. `~` is the special part that signifies to travel the edge in reverse. For example given a `product` node and `category` node and a `category` _edge_ from product to `category` it would be possible to do `~category { ...product fields here... }` within a category context in a query to get the product that points to it.
* given the lack of namespacing in dgraph schemas a convention has emerged to name edges with a prefix of the node type. For example in a movies database to differentiate directors from actors the schema used edge names `director.film` and `actor.film`. Its not clear how far this pattern should go. It seems like a case-by-case decision.

## Sun Sep 10

* found out that asciidoc does not support strikethough in a way that supports Github (or viceversa) [link](https://github.com/asciidoctor/asciidoctor/issues/1030) [link](https://github.com/christiangalsterer/bitbucket-asciidoc-plugin/issues/15). This prevented me from being able to format a log title in the way I wanted.

##### Amazon Alexa is a kind of voice-based interface not unlike Apple Siri.

* Amazon Echo is a hardware product line that makes Alexa convenient to use
* Developers can "teach Alexa skills" which is analogus to e.g. writing iOS apps. teach -> write, skill -> app
* Alexa skills are configured with an amazon developer account, then implemented. The skill's interaction model is defined in this configuration layer, e.g. what utterances can be used.
* `Invocation Name` is the name given to enter your skill from alexa. For example `essence` will enter the `ssense` skill
* Each skill has multiple `intents`. These are like functions or endpoints in your skill. You defined them as a developer.
* Each intent has multiple `utterances`. These are ways the user can speak to execute the intent.
* There is another concept called `slots` which are for parameters in intents. But I have not actually played with these yet.
* There are different APIs available for developers to use to build skills. For highly custom skills there is a Custom API which can POST intents to any host running an HTTPS server.
* links: [Alexa Skills entry point for developers](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/getting-started-guide), [Amazon Echo Show entry point for developers](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/build-skills-for-echo-show#display-and-interaction-features-on-echo-show), [Custom API](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/overviews/understanding-custom-skills), [JSON Interface Reference for Custom Skills](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/alexa-skills-kit-interface-reference), [Display Interface Reference](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/display-interface-reference)

* DGraph's [go client](https://godoc.org/github.com/dgraph-io/dgraph/client) is their most feature complete one. DGraph's [`dgraphloader`](https://github.com/dgraph-io/dgraph/tree/master/cmd/dgraphloader) is built on top of it.

## Wed Sep 6

* learnt about the following `dgraph` `mutation` today.

```graphql
mutation {
  set {
    _:cat <name> "Millhouse" .
    _:cat <color> "Black" .
    _:cat <age> "0.7"^^<xs:float> .

    _:human <name> "Kaley" .
    _:human <age> "22"^^<xs:float> .
    _:human <favorite_food> "chocolate" .

    _:human <owns> _:cat .
  }

  schema {
    name: string @index .
  }
}
```

** `mutation` is for changing data in the graph or changing the graph schema
** `set` is for mutations that insert triples into the graph
\*\* the strange syntax `^^<xs:float>` is apparently how a value is typed as a float...

* about `dgraph` triples
  ** triples are specified according the W3C standard https://www.w3.org/TR/n-quads/[RDF N-Quad format]
  ** their format is `<subject> <predicate> <object> .` `subject` is always a node. `object` is either a `node` or a `value` (also know as literal). `predicate` is a directed edge from `subject` to `object`, the value here is the edge name. A given edge must always point to a consistent type (in effect the edge type). A `.` is present because of the spec apparently less because of need on dgraph side https://dgraph.slack.com/archives/C13LH03RR/p1504754827000129[link]

* `blank node` is written `_:identifier` in a mutation. Used to identify a node within a mutation. Outside a particular mutation the identifiers have no existance. `_` will be replaced by dgraph with an automatically generated 64bit unique ID. These IDs are available in the mutation return result:

```json
{
  "data": {
    "code": "Success",
    "message": "Done",
    "uids": {
      "foo": "0x2712",
      "qux": "0x2713",
      "bar": "0x2714"
    }
  }
}
```

* links: https://docs.dgraph.io/query-language/#mutations[mutation docs], https://docs.dgraph.io/master/guides/#adding-data-to-dgraph[guide/intro to mutations]

* in `dgraph` schema types are defined globally without any ability to nest into records. https://dgraph.slack.com/archives/C13LH03RR/p1504755357000113[link]. For example this would fail:

```graphql
mutation {
  schema {
    foo {
      bar: string .
    }
  }
}
```

but this would work:

```graphql
mutation {
  schema {
    bar: string .
  }
}
```

* `dgraph` supports pagination which can be used as the basis for doing batch work across an entire graph. [slack link](https://dgraph.slack.com/archives/C13LH03RR/p1504745800000004), [pagination docs link](https://docs.dgraph.io/master/query-language/#pagination)

## Tue Sep 5

* [dgraph](https://dgraph.io) has enough power in its query language to apply both collaborative-based and content-based filtering strategies [link](https://blog.dgraph.io/post/recommendation) [link](https://blog.dgraph.io/post/recommendation2/).

* _collaborative-based filtering_ is a broad strategy for recommending things based upon matching like-users and then recommending to one based on another(s).

* _cold-start_ problem refers to being unable to integrate a new user into collaborative-based filtering for lack of data with that user.

* _content-based filtering_ is a broad strategy for recommending things based on their similarity to another given thing.
