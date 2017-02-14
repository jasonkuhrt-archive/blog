# React Popover Layout Algorithm

This article is an overview of the layout algorithms used for react-popover. If you are curious about what react-popover is or why it exists then you may want to read a recent article detailing its back-story.

----

## Component Anatomy

![anatomy-layout.png]()

There are three core and independent components in our system: Target, Popover, Frame. There is also an auxiliary component within Popover called Tip.

Our system knows virtually nothing about the core components except their position and bounding box dimensions. Ditto for Tip except we additionally must know at which degree it is in a "upward pointing state".

Target is the aim of Popover. Popover is the thing we are trying to automatically position relative to Target. Popover's position can also be influenced by Frame. Frame is the bounding box that Popover's own bounding box must be/should be within (which semantic to use is configurable as we will see). Tip is a visual aid hinting that Popover refers to Target. Tip is positioned relative to Popover, but also Target as we will see.



## Layout Anatomy

The layout system is based upon x/y axes and the four sides of a box. It borrows a concept or two from Flexbox but is generally very much its own idea.

![ordinal-sides.png]()

#### Relative Axes

In our system relative axes are "main axis" and "cross axis" whose concrete orientation depends upon Popover position relative to Target. The main axis is whichever one whose orientation contains Popover adjacent to Target. For example if Popover is horizontally adjacent to Target then the main axis is horizontal. The cross axis is simply whatever orientation main axis is not.

#### Ordinal Sides

If we abstract away concrete orientation then how do we continue thinking about the four sides of a box: top, right, bottom, left? The solution is to to remove their implied orientation and think about order. By prefixing the names of relative axes to disambiguate we are freed to generalize the four sides into two: before/after. The former refers to either top or left while the latter botttom or right [1].

## Calculate the optimal zone

![zone-determination.png]()

Popover will be positioned in the zone deemed best amongst all four possibilities. The algorithm requires two passes. First calculate the fit of each zone and then rank those fits to find the optimal zone.

#### Phase 1: Fits

For each zone we subtract Popover's height and width from it. When doing this we also have to factor in the Tip's main-axis length (see appendix A for how/why). The result is knowing how much spare space each zone would have on either of its dimensions after fitting Popover.

#### Phase 2: Ranks

We group zones into first or second class. First class zones are those whose fit is positive on both dimensions. Second class zones are those whose fit is negative on one or both dimensions.

Next, pick the first class zone with the largest area. If there are no first class zones then pick the second class zone with least percentage area exceeding Frame bounds. This is your optimal zone.

Finally, check if the optimal zone just calculated beats previous optimal zone by the given threshold (see appendix B for how/why we support this). If it does then update Popover's position, otherwise do nothing.

![zone-scenarios.png]()



## Calculate Popover position


![popover-centering.png]()

With the optimal zone found we can now calculate the best position for Popover within it.

Our algorithm looks for the position of Popover that would see its main axis matched to that of Target. Depending on what the user has decided, this positioning may behave in three distinct ways: sensitive to Frame bounds, insensitive to frame bounds, sensitive to Frame bounds up to a given threshold. Respectively these modes are called "bounded", "unbounded", "semi-bounded".

If positioning Popover to have its main axis exactly match Target's would cause Popover to exceed Frame bounds, then...

* **Bounded**
...position Popover up to Frame edge but not beyond it
* **Unbounded**
...go ahead
* **Semi-Bounded**
...calculate if the area percentage cropped of Target would exceed given threshold. If yes, then be unbounded, else be bounded.

  It is conceivable that another factor for the threshold could be the _Popover's_ percentage area cropped. Its unclear to me if this would be useful or not though.

![popover-bounded.png]()
![popover-unbounded.png]()
![popover-semi-bounded.png]()



## Calculate Tip position

As mentioned in the introductory anatomy Tip is a sub-component of Popover, its job to visually hint Popover's reference of Target. Our system assumes that Tip has a pointer on top and base on bottom. In other words that at rest (no rotation) Tip is pointing upward. Its layout rules are:

* Along main-axis: between Popover and Target
* Along cross-axis: centered between nearest before-side and after-side amongst Target and Popover
* Faces Target

![tip-centering.png]()
![tip-rotation.png]()



## Appendices

These appendices cover deep details that underpin reliable layout.

### Appendix A: Infinite Loop

![edge-case-infinite-loop.png]()

When calculating a zone's fit the Tip's contribution to Popover dimensions must be specially handled. If it were not then an infinite loop of zone rank changes could occur in cases involving only second-class options.

Tip length affects either height or width of Popover depending upon the zone side. So two zones of opposite orientation are going to manifest slightly different Popover dimensions. Consequently this could affect Popover crop percentage in second-class zones leading to always another zone appearing better than the current one. The diagram helps illustrate such a case.

A non-general solution to this problem is to always add the Tip's main-axis length to Popover's main-axis length when calculating a zone's fit rank. For example for top zone add Tip length to the Popover height; for right zone add Tip length to Popover width; etc.

### Appendix B: Minimum-improvement Thresholds to prevent layout Jitter

![edge-case-jitter.png]()

Thresholds are needed to prevent layout jitter (bad for user-experience) caused by zones with tight ranking flipping around the precipice. The diagrams show examples of how minor jitters can be magnified into excessive layout changes.

The underlying problem thresholds solve is that without them we have tightly coupled jitter from the inputs (arrangement, size, etc. of Target, Popover, Frame) to pass right through to our output (zone ranking). Thresholds allow us to define and filter out insignificant zones rank changes, controlling the balance between optimal positioning and layout stability.

Some threshold examples:

* threshold 0.2 means balance stability and positioning: other zones need 20% greater area for change
* threshold 0 means prioritize optimal position: other zones need 1px greater area for change
* threshold Infinity means prioritize stability: other zones are never changed to unless it would mean upgrading from second class to first

It may be useful to let users decide if they want to opt-in/out of zone class upgrades thereby limiting criteria for zone changes strictly to their differences in area.







[1] The choice of mapping "before" to "top" as opposed to "bottom" reflects the coordinate system on the web where 0,0 is top-left. To people familiar with Math, Adobe Flash, or other environments, this is unnatural but alas I took the expedient approach by staying consistent with the web.
