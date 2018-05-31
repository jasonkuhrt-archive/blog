# NPM Package & Lock File Basics

This is a TL;DR from a day in the life of npm dependency shenanigans. Throughout this article I will use `git status` and `git diff` to make state explicit. You should be able to reproduce exactly what you see here. Lines prefixed with `>` are what I entered into the terminal. Anything else is output I chose to show herein but often I will not copy the terminal output to this article because it would not be interesting.

## Setup

Lets start with an empty node project.

First our node and npm versions:

```
> npm --version
6.1.0
> node --version
v8.11.1
```

Lets initialize a project:

```
> mkdir foobar && cd foobar
> npm init --yes
> git init .
> git status --short
?? package.jsonk
> git add -A && git commit -m 'initial commit'
```

Now lets add some dependencies:

```
> npm install io-ts fp-ts
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN foobar@1.0.0 No description
npm WARN foobar@1.0.0 No repository field.

+ io-ts@1.1.3
+ fp-ts@1.6.1
added 2 packages from 1 contributor in 2.291s
```

Note how like `yarn` we now do not need to specify `--save`:

```
> git status --short
 M package.json
?? node_modules/
?? package-lock.json
> git diff package.json
```

```diff
diff --git a/package.json b/package.json
index 2ed804c..8752155 100644
--- a/package.json
+++ b/package.json
@@ -8,5 +8,9 @@
   },
   "keywords": [],
   "author": "",
-  "license": "ISC"
+  "license": "ISC",
+  "dependencies": {
+    "fp-ts": "^1.6.1",
+    "io-ts": "^1.1.3"
+  }
 }
```

And our lock file has been created:

```
> cat package-lock.json
{
  "name": "foobar",
  "version": "1.0.0",
  "lockfileVersion": 1,
  "requires": true,
  "dependencies": {
    "fp-ts": {
      "version": "1.6.1",
      "resolved": "https://registry.npmjs.org/fp-ts/-/fp-ts-1.6.1.tgz",
      "integrity": "sha512-Er2h61dY/1lgZLzJY9CyITvLehF/KnqbvAZqeJ9xVnxT0zpAN9+QCxZRrAiFmzkFv4gBVcUM9ohScbRfx4hceQ=="
    },
    "io-ts": {
      "version": "1.1.3",
      "resolved": "https://registry.npmjs.org/io-ts/-/io-ts-1.1.3.tgz",
      "integrity": "sha512-FsktDpz3rYeYU/cQBZrGYvsl1gv4M/xLJKkfOiWe5xQNvz6zAihpykay4OEPk265JgEJYJN9DALZOUbWWSqmGA==",
      "requires": {
        "fp-ts": "^1.0.0"
      }
    }
  }
}
```

Lets do a checkpoint:

```
> echo "node_modules" > .gitignore && \
  rm -rf node_modules && \
  git add -A && \
  git commit -m 'add first deps'
```

## Incompatible Downgrade

What will happen if we manually downgrade deps inside package.json?

```
> vim package.json
> cat package.json | grep --after-context 2 dep
  "dependencies": {
    "fp-ts": "1.6.0",
    "io-ts": "1.1.0"
> npm install
> cat node_modules/fp-ts/package.json | grep version\":
  "version": "1.6.0"
> cat node_modules/io-ts/package.json | grep version\":
  "version": "1.1.0"
> git diff package-lock.json
```

```
> diff --git a/package-lock.json b/package-lock.json
index 7ec9a88..e97f361 100644
--- a/package-lock.json
+++ b/package-lock.json
@@ -5,18 +5,17 @@
   "requires": true,
   "dependencies": {
     "fp-ts": {
-      "version": "1.6.1",
-      "resolved": "https://registry.npmjs.org/fp-ts/-/fp-ts-1.6.1.tgz",
-      "integrity": "sha512-Er2h61dY/1lgZLzJY9CyITvLehF/KnqbvAZqeJ9xVnxT0zpAN9+QCxZRrAiFmzkFv4gBVcUM9ohScbRfx4hceQ=="
+      "version": "1.6.0",
+      "resolved": "https://registry.npmjs.org/fp-ts/-/fp-ts-1.6.0.tgz",
+      "integrity": "sha512-zfr7ipmiKO/uC2MjskNPfEqV6gaubFRSTsQKZkKTdBT6nLhFN3bMFhrJyL9Tvvw7BfXLzlZtJkrr3LdENqco2g=="
     },
     "io-ts": {
-      "version": "1.1.3",
-      "resolved": "https://registry.npmjs.org/io-ts/-/io-ts-1.1.3.tgz",
-      "integrity": "sha512-FsktDpz3rYeYU/cQBZrGYvsl1gv4M/xLJKkfOiWe5xQNvz6zAihpykay4OEPk265JgEJYJN9DALZOUbWWSqmGA==",
+      "version": "1.1.0",
+      "resolved": "https://registry.npmjs.org/io-ts/-/io-ts-1.1.0.tgz",
+      "integrity": "sha512-UvnWBGF8M+1199K+5xOfk8BAjxoRavLF7yOkzJL3Gz60KQO+74UY4y69FuKdvBF+AqUUTn316Vn+2mvFKU6DQw==",
       "requires": {
         "fp-ts": "^1.0.0"
       }
     }
   }
 }
-
```

So:

1.  lock file forced to conform to our package file
2.  deps installed agree too

Lets checkpoint again:

```
> rm -rf node_modules && git commit -am 'exactly pin deps'
```

## Compatible Upgrade

What happens if we pin our deps back to `^` which will accept compatible (semver minor/patch) versions equal to or greater than than the specified version?

First update our package file:

```
> vim package.json
> cat package.json | grep -A 2 dep
  "dependencies": {
    "fp-ts": "^1.6.0",
    "io-ts": "^1.1.0"
```

Now we have specified that we will accept any `1.x.x` version of fp-ts equal to or greater than `1.6.0` and `1.x.x` of `io-ts` equal to or greater than `1.1.0`. However importantly remember that our lock says what we actually have installed to date is:

```
> cat package-lock.json | grep '      "version\":'
      "version": "1.6.0",
      "version": "1.1.0",
```

This question relates to what regularly happens in day-to-day developer life. `1.6.0` is installed today, tomorrow there is `1.6.1`, and so on. We know from before that `io-ts` and `fp-ts` have newer version than currently specified in package.json and the range we specified in package.json would allow us to install them, but the lock should not, right? Lets find out:

```
> git status --short
M  package.json
> npm install
> git status --short
M  package.json
> cat node_modules/fp-ts/package.json | grep version\":
  "version": "1.6.0"
> cat node_modules/io-ts/package.json | grep version\":
  "version": "1.1.0"
```

So:

* the lock enforces what our deps are this time
* the reason it did not before but does now is that before the lock file had a version conflict with package file and package file won, but this time around, the lock file and package file are not in conflict, but rather the lock file allows a subset of the package file.

Lets checkpoint again:

```
> rm -rf node_modules && git commit -am 'compat pin deps'
```

## Compatible Upgrade Again

Just to make the point about conflict between lock and package files clear, lets once again introduce conflict:

```
cat package.json | grep -A 2 dep
  "dependencies": {
    "fp-ts": "^1.6.1",
    "io-ts": "^1.1.3"
```

which conflicts with lock file because lock file is no longer a subset:

```
cat package-lock.json | grep '      "version\":'
      "version": "1.6.0",
      "version": "1.1.0",
```

Lets see what will happoen:

```
> git status --short
M  package.json
> npm install
> git status --short
 M package-lock.json
 M package.json
> cat node_modules/fp-ts/package.json | grep version\":
  "version": "1.6.1"
> cat node_modules/io-ts/package.json | grep version\":
  "version": "1.1.3"
> git diff package-lock.json
```

```diff
diff --git a/package-lock.json b/package-lock.json
index e97f361..966fb1e 100644
--- a/package-lock.json
+++ b/package-lock.json
@@ -5,14 +5,14 @@
   "requires": true,
   "dependencies": {
     "fp-ts": {
-      "version": "1.6.0",
-      "resolved": "https://registry.npmjs.org/fp-ts/-/fp-ts-1.6.0.tgz",
-      "integrity": "sha512-zfr7ipmiKO/uC2MjskNPfEqV6gaubFRSTsQKZkKTdBT6nLhFN3bMFhrJyL9Tvvw7BfXLzlZtJkrr3LdENqco2g=="
+      "version": "1.6.1",
+      "resolved": "https://registry.npmjs.org/fp-ts/-/fp-ts-1.6.1.tgz",
+      "integrity": "sha512-Er2h61dY/1lgZLzJY9CyITvLehF/KnqbvAZqeJ9xVnxT0zpAN9+QCxZRrAiFmzkFv4gBVcUM9ohScbRfx4hceQ=="
     },
     "io-ts": {
-      "version": "1.1.0",
-      "resolved": "https://registry.npmjs.org/io-ts/-/io-ts-1.1.0.tgz",
-      "integrity": "sha512-UvnWBGF8M+1199K+5xOfk8BAjxoRavLF7yOkzJL3Gz60KQO+74UY4y69FuKdvBF+AqUUTn316Vn+2mvFKU6DQw==",
+      "version": "1.1.3",
+      "resolved": "https://registry.npmjs.org/io-ts/-/io-ts-1.1.3.tgz",
+      "integrity": "sha512-FsktDpz3rYeYU/cQBZrGYvsl1gv4M/xLJKkfOiWe5xQNvz6zAihpykay4OEPk265JgEJYJN9DALZOUbWWSqmGA==",
       "requires": {
         "fp-ts": "^1.0.0"
       }
```

Hopeully now its clear when lock file rules and when it does not. In practice the demonstrated behavior is safe because adjustments to package.json via manual editing or commands like `npm update` are explicit conscious actions and/or (bot like greenkeeper) going through a pull-request review with CI tests green-lighting. The point is the lock should only be "losing" in developer-explicit cases.

Lets undo our chanages and take another path now: `npm update`.

Undo what we've done:

```
> rm -rf node_modules && git checkout .'
```

Now lets see what is outdated:

```
> npm outdated
Package  Current  Wanted  Latest  Location
fp-ts    MISSING   1.6.1   1.6.1  foobar
io-ts    MISSING   1.1.3   1.1.3  foobar
> npm install
> npm outdated
Package  Current  Wanted  Latest  Location
fp-ts      1.6.0   1.6.1   1.6.1  foobar
io-ts      1.1.0   1.1.3   1.1.3  foobar
```

So:

1.  `current` column is data from what is literally in node_modules
2.  `wanted` column is what the latest published version of the packge on the registry is that conforms to our range specification
3.  `latest` is simply what the latest published version of the package is. Here it is the same as `wanted` but if `io-ts` `2.0.0` were released, for example, you would see that `wanted` column remains as it is above but `latest` change to `2.0.0`.

Now lets update:

```
> npm update
> git diff
```

```diff
diff --git a/package-lock.json b/package-lock.json
index e97f361..966fb1e 100644
--- a/package-lock.json
+++ b/package-lock.json
@@ -5,14 +5,14 @@
   "requires": true,
   "dependencies": {
     "fp-ts": {
-      "version": "1.6.0",
-      "resolved": "https://registry.npmjs.org/fp-ts/-/fp-ts-1.6.0.tgz",
-      "integrity": "sha512-zfr7ipmiKO/uC2MjskNPfEqV6gaubFRSTsQKZkKTdBT6nLhFN3bMFhrJyL9Tvvw7BfXLzlZtJkrr3LdENqco2g=="
+      "version": "1.6.1",
+      "resolved": "https://registry.npmjs.org/fp-ts/-/fp-ts-1.6.1.tgz",
+      "integrity": "sha512-Er2h61dY/1lgZLzJY9CyITvLehF/KnqbvAZqeJ9xVnxT0zpAN9+QCxZRrAiFmzkFv4gBVcUM9ohScbRfx4hceQ=="
     },
     "io-ts": {
-      "version": "1.1.0",
-      "resolved": "https://registry.npmjs.org/io-ts/-/io-ts-1.1.0.tgz",
-      "integrity": "sha512-UvnWBGF8M+1199K+5xOfk8BAjxoRavLF7yOkzJL3Gz60KQO+74UY4y69FuKdvBF+AqUUTn316Vn+2mvFKU6DQw==",
+      "version": "1.1.3",
+      "resolved": "https://registry.npmjs.org/io-ts/-/io-ts-1.1.3.tgz",
+      "integrity": "sha512-FsktDpz3rYeYU/cQBZrGYvsl1gv4M/xLJKkfOiWe5xQNvz6zAihpykay4OEPk265JgEJYJN9DALZOUbWWSqmGA==",
       "requires": {
         "fp-ts": "^1.0.0"
       }
diff --git a/package.json b/package.json
index cf97ff1..8752155 100644
--- a/package.json
+++ b/package.json
@@ -10,7 +10,7 @@
   "author": "",
   "license": "ISC",
   "dependencies": {
-    "fp-ts": "^1.6.0",
-    "io-ts": "^1.1.0"
+    "fp-ts": "^1.6.1",
+    "io-ts": "^1.1.3"
   }
 }
```

As you can see we have achieved our result from before but instead of manually changing packge file and then running `npm install` to sync lock file we have done it in one fell swoop. This is a better technique because it makes it less likely for a change to package.json without corresponding lock update to be committed. Unfortunately [`npm update`](https://docs.npmjs.com/cli/update) is not very flexible and does not appear able to cope with things like updating outside the specified range in package file. Contrast with [`yarn upgrade`](https://yarnpkg.com/lang/en/docs/cli/upgrade/) which has many knobs to produce different reuslts for different needs such as `--scope` `--latest` `--pattern` and more.

## Why Lock?

One may suggest that rather than locking we can simply pint exact version into our package file but this has a few issues:

1.  It does not solve your dep tree problem, deps-of-deps, where they may not be be pinning exactly
2.  If everyone used exact pins however, would that be any better? It would make dep trees incredibly rigid preventing a flow of patches from improving the security/performance/etc. of your codebase. Modern apps have literally hundreds if not thousands of deps in their tree and the ability to scale minor/patch changes is one reason this can be practical. Of course we're still talking about scaling _explicitly_ via lock files. However lockfiles, unlike package files, allow consumers to advance the state of their whole dependency tree at their own pace rather than depending on their hundreds/thousands of OSS peers around the world to update exactly pinned deps in their libraries.
3.  It neuters your ability to leverage `npm outdated` and especially `npm update` and especially if you're not using `yarn` which has a more powerful update cli.

For [much] further reading into how the community has thought about these issues go read the conversations that took place leading to `5.x.x` lock files:

* https://github.com/npm/npm/issues/16866
* https://github.com/npm/npm/pull/17508

Lets checkpoint again:

```
> rm -rf node_modules && git commit -am 'latest compat pin deps'
```

## Oh, npm

A new issue with npm is that lock files between version 5 and 6 of npm are not stable. Behold:

```
> npm install -g npm@5.7 && npm --version
5.7.1
> npm install && git diff
diff --git a/package-lock.json b/package-lock.json
index 966fb1e..853c8ae 100644
--- a/package-lock.json
+++ b/package-lock.json
@@ -14,7 +14,7 @@
       "resolved": "https://registry.npmjs.org/io-ts/-/io-ts-1.1.3.tgz",
       "integrity": "sha512-FsktDpz3rYeYU/cQBZrGYvsl1gv4M/xLJKkfOiWe5xQNvz6zAihpykay4OEPk265JgEJYJN9DALZOUbWWSqmGA==",
       "requires": {
-        "fp-ts": "^1.0.0"
+        "fp-ts": "1.6.1"
       }
     }
   }
```

You can read more about the issue [here](https://github.com/npm/npm/issues/20434). At best this will cause confusion, make pull-requests harder to review, and potentially hide real problems as inter/intra teams start to ignore lock file churn as noise, until it isn't.
