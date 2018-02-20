# My optimal workflow

2018 Feb 19

The following is my preferred take on software workflow topics as it relates to git, github, and changelogs. It surely is not a universal fit for all types of projects but in my experience would seem to cover the vast majority. That said I have no novel ideas to submit. Just cherry-picks from my observations of great work done by others in the field. My curatorial sensibilities bias toward the point of optimal benefit-to-simplicity ratio.

## The Flow

### Principals

It tries to adhere to the following principals:

1. **Low & inclusive barrier to collaboration**
   The more complicated the process, the more people feel blocked from participating.

2. **Tooling over conventions**
   Avoid manual human policing/maintenance. Make things foolproof. We're human so what can go wrong will go wrong. [Murphy told us that](https://en.wikipedia.org/wiki/Murphy%27s_law).

3. **Simplicity for safety**
   Complicated or even semi-complicated workflows attempting to "fix" perceived risks may just end up complicating matters by making the surface area for screwing up even larger. The worst-case scenario is non-simple workflows based on convention.

4. **Standardize along the cow path**
   Use a simple approach that covers vast majority of cases, leaving the genuinely weird cases to live in their own ad-hoc quarantine.

### Diagram

![diagram](./diagram2.png)

### Breakdown

1. **Follows a battle-tested pattern popularized by Github ([ref](https://guides.github.com/introduction/flow/))**

   * Seems to align with a wide cross section of industry practice avoiding not-invented-here syndrome.
   * Avoids unruly surprises for new engineers to your organization with contemporary expectations while at the same time equipping juniors to quickly redeploy in-house knowledge to the field.

2. **Master as base branch**

   * Aligns with tooling defaults (for example `hub pull-request`)
   * Makes it practically impossible to not know which brach to pull-request too for the vast majority of cases
   * Makes it so there is only one branch to [protect](https://help.github.com/articles/about-protected-branches/) (excepting the odd and few version branches)
   * TBD makes it possible to leverage repo settings to automatically guarantee a single PR merging style (if there were multiple kinds of PR flows into bases with varying standards as prescribed by some git flows then repo settings wouldn't be able to enforce for a single style thereby passing the buck to humans to manually police)

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

5. **Can be used for repos both internal and open-source repos**

6. **Simplicity**

   * Aligns with the trend toward smaller teams working on smaller repos in microservice architectures.

   * Makes developers happy.

   * Encourages small quality changes since they're easy to make. If change is laborious expect to see less of it and/or in larger batches. A poorly written documentation paragraph, a promiscuous function signature, a missing test case, a dependency upgrade, a refactor, ... these sorts of things could be dissuaded by complexity and/or may more often be batched into unrelated pull-requests.

7. **Commit messages further automate the flow**
   * Keep references to PR for commit history
   * Describe the semver effect to automate changelog later
   * [Optionally close issues automatically](https://help.github.com/articles/closing-issues-using-keywords/) (but PR description is often the better place to do this)

### References

* [hub](https://github.com/github/hub)
* [release](https://github.com/zeit/release)
* [squash-merge](https://help.github.com/articles/about-pull-request-merges/#squash-and-merge-your-pull-request-commits)
* [protected branches](https://help.github.com/articles/about-protected-branches/)
* [on the topic of low bars to collaboration](https://rfc.zeromq.org/spec:22/C4/)
