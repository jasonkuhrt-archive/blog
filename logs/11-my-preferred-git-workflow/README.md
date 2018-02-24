# A workflow that optimizes for simplicity

2018 Feb 19

<!-- TOC START min:2 max:3 link:true update:true -->

* [Principals](#principals)
* [Flow Diagram](#flow-diagram)
* [Breakdown](#breakdown)
  * [Add a feature](#add-a-feature)
  * [Make a release](#make-a-release)
* [Rationale](#rationale)
* [Future improvements](#future-improvements)
* [References](#references)

<!-- TOC END -->

The following is my preferred workflow on software projects. It likely isn't a universal fit for all shapes and sizes but in my experience this or variants of it have worked quite well for me. I strive for the optimal benefit-to-simplicity ratio.

## Principals

1. **Remove steps**
   Keep thing simple. Complicated or even semi-complicated workflows attempting to "fix" perceived risks may just end up complicating matters by making the surface area for erring even larger. Also the more complicated the process, the greater the risk of dampened collaboration.

2. **Automate steps**
   Do not waste time policing that rules are remembered and followed correctly. We're human. What can go wrong will go wrong. [Murphy told us that](https://en.wikipedia.org/wiki/Murphy%27s_law). Use tooling to stop thinking about this.

3. **Standardize along the cow path**
   Use a simple approach that covers vast majority of cases. Edge cases ought to live in case-by-case procedures, rather than diluting the central pathway.

## Flow Diagram

![diagram](./diagram2.png)

## Breakdown

### Add a feature

### Make a release

## Rationale

1. **Follows a battle-tested pattern popularized by Github ([ref](https://guides.github.com/introduction/flow/))**

   * Seems to align with a wide cross section of industry practice avoiding not-invented-here syndrome.
   * Avoids unruly surprises for new engineers to your organization with contemporary expectations while at the same time equipping juniors to quickly redeploy in-house knowledge to the field.

2. **Master as base branch**

   * Aligns with tooling defaults (for example `hub pull-request`).
   * Makes it practically impossible to not know which brach to pull-request too for the vast majority of cases.
   * Makes it so there is only one branch to [protect](https://help.github.com/articles/about-protected-branches/) (excepting the odd and few version branches).

3. **Squash-merge pull-requests ([ref](https://help.github.com/articles/about-pull-request-merges/#squash-and-merge-your-pull-request-commits))**

   * Enables a clean git history where it counts most.
   * Reverting bad features becomes trivial.
   * Is [enforceable via tooling](https://help.github.com/articles/configuring-commit-squashing-for-pull-requests/) therefore automatically guaranteeing that mainline history will never become dirty (not-withstanding bad data entry at PR UI when merging)
   * Frees developers to work quickly in their branch without wasted brain cycles about how inconsequential detail will affect mainline history. Nice commits still matter and they should be honed to make PRs easier to understand. _But these two points don't need to be mutually exclusive_.
   * Encourages PRs to be small, focused, releasable units of change which they need to be for a scalable and successful PR workflow anyways. This is behaviour the workflow should reinforce wherever possible.
   * Individual commit history is still maintained on the PR itself for future auditing.

4. **One level of branching (feature branches)**

   * Simple.
   * Aligns with continuous delivery workflows where work should constantly be going to production on `master`.
   * Makes it possible to use repo settings to automatically enforce repo settings. I have seen cases where the PR merge style used to mainline versus another branch type like releases had different policies. There is no way to enforce that on Github however so you end up with more complex branches with less guarantee.

5. **Can be used for repos both internal and open-source repos**

6. **Simplicity**

   * Aligns with the trend toward smaller teams working on smaller repos in microservice architectures.

   * Makes developers happy.

   * Encourages small quality changes since they're easy to make. If change is laborious expect to see less of it and/or in larger batches. A poorly written documentation paragraph, a promiscuous function signature, a missing test case, a dependency upgrade, a refactor, ... these sorts of things could be dissuaded by complexity and/or may more often be batched into unrelated pull-requests.

7. **Automate flow with natural queues**

   * Close issues automatically when PR merged [ref](hthttps://github.com/blog/1506-closing-issues-via-pull-requests)
   * Automatically keep references to PR in commit message. PR contains the various feature branch commits unsquashed
   * Describe the semver effect in commit message to automate changelog later

8. Automate Style enforcement

   * [Use prettier](https://prettier.io/)
   * Use githook to run prettier prior to commit ([ref](https://prettier.io/docs/en/precommit.html)). The Husky + `pretty-quick` approach has worked great for me so far.

## Future improvements

* Prune local branches with `git sync` once https://github.com/github/hub/issues/1304

## References

* [hub](https://github.com/github/hub)
* [release](https://github.com/zeit/release)
* [squash-merge](https://help.github.com/articles/about-pull-request-merges/#squash-and-merge-your-pull-request-commits)
* [protected branches](https://help.github.com/articles/about-protected-branches/)
* [on the topic of low bars to collaboration](https://rfc.zeromq.org/spec:22/C4/)
