# React Popover Layout Algorithm

This article is an overview of the layout algorithms used for react-popover. If you are curious about what react-popover is or why it exists then you may want to read a recent article detailing its back-story.

----

## Component Anatomy

![anatomy-layout.png]()

There are three core and independent components in our system: Target, Popover, Frame. There is also an auxiliary component within Popover called Tip.

Our system knows virtually nothing about the core components except their position and bounding box dimensions. Tip requires a superset of this knowledge, adding just one additional point which is that at zero-rotation it is in its "upward pointing state" (this places a very minor constraint on how the contents of Tip should be laid out).

Target is the aim of Popover. Popover is the thing we are trying to automatically position relative to Target. Popover's position can also be influenced by Frame. Frame is the bounding box that Popover's own bounding box must be or should be within (which semantic to use is configurable). Tip is a visual aid hinting how Popover is linked to Target. It is positioned relative to Popover but like Popover can be influenced by an additional element: Target.



## Layout Anatomy

The layout system is based upon two ideas: the x and y orientations in a 2D plain and the four sides of a box. These two ideas map to two parts of our layout system termed "relative axes" and "edges". If you are familiar with the Flexbox specification then our system may seem vaguely familiar to you.

![ordinal-sides.png]()

### Relative Axes

The idea of relative axes is an abstracted notion of orientation. Rather than thinking in terms of "vertical" and "horizontal" we think about "main" and "cross" By delaying which orientation we're referring to in communication (and data) we can specify our layout rules more generally thus simplifying our specification and having a more flexible implementation.

"Relative" here means that which orientation the axis actually is depends on external conditions. In our system the conditions are regarding Popover position relative to Target. Consequently its perfectly normal for the "main" axis of one Popover to be the opposite orientation of the "main" axis of another Popover.  

The Main axis is whichever orientation in which Popover is adjacent to Target. So if the Popover is horizontally adjacent to the Target then the main axis is the horizontal one. Conversely if the Popover is vertically adjacent to the Target then the main axis is the vertical one.

The Cross Axis is simply whatever orientation Main Axis is not.

Relative Axes is a simple idea but perhaps better expressed visually than in words. Study the diagram if in doubt and again if you are familiar with the Flexbox specification then this should feel familiar to you.

### Ordinal Sides

If we abstract away concrete orientation then how do we continue thinking about the four sides of a box: top, right, bottom, left? The solution is to to remove their implied orientation and think about order. By prefixing relative axes to disambiguate we are freed to generalize the four sides into two simply two positions: "before" and "after".

The ordinal side "before" concretely refers to either top or left; conversely "after" refers to either bottom or right. The choice of mapping "before" to vertical orientation "top" (as opposed to "bottom") reflects the coordinate system on the web where 0,0 is located top-left rather than bottom-left. To people familiar with Math, Adobe Flash, or other settings, the latter is likely more natural, but alas I took the expedient approach by staying consistent with the web.

Note that unlike Relative Axes there's nothing relative about Ordinal Sides. Each axis always has a "before" and "after" side.



## Zone Determination

![zone-determination.png]()

Popover will be positioned in the zone deemed best amongst all four possibilities. The algorithm requires two passes. First calculate the fit of each zone and then rank those fits to find the optimal zone.

### Calculating Fits

For each zone we subtract Popover's height and width from it. When doing this we also have to factor in the Tip's main-axis length. Appendix A goes into detail about how and why this is done.

### Calculating Ranks

To rank zones based on their fit we take the following steps:

1. Group zones into first or second class. First class zones are those whose calculated fit has a positive difference on both dimensions. Second class zones are those whose calculated fit has a negative difference on one or both dimensions.
2. Pick the first class zone with the largest area.
3. If there are no first class zones then pick the second class zone with the least percentage of its area exceeding Frame bounds.

![zone-scenarios.png]()


TODO
Revise jitter section to encompass the following
flush out the following stub ...:
* threshold-based changes between first class zones to prevent jitter
* upgrading (change from second to first class) will not be subject to thresholds
* threshold 0 means single-pixel-difference sensitivity
* threshold 0.2 means other zone must have 20% more area to trigger change
* threshold Infinity means disable changing between first class zones altogether. Only move to a zone upon initial layout or when upgrading



## Main Axis Centre Match

With the zone selected we can now calculate the best position for Popover therein.

Our algorithm looks for the position of Popover that would see its main axis matched to that of Target. Depending on what the user has decided, this positioning may behave in three distinct ways: sensitive to the Frame bounds, ignore the frame bounds, or be a hybrid of both wherein only up to a certain threshold are Frame bounds honored, after which they are ignored. Respectively these modes are called "bounded", "unbounded", "semi-bounded".

The specifics of the modes are as follows. If positioning Popover to have its main axis exactly match Target's would cause Popover to exceed Frame bounds...

![popover-centering.png]()

### Bounded

...then position Popover up to Frame edge but not beyond it.

![popover-bounded.png]()

### Unbounded

...then no matter, do so.

![popover-unbounded.png]()

### Semi-Bounded

...then calculate if the area percentage cropped of Target (if any) would exceed the given threshold. If it would then allow Popover to break bounds (like unbounded), otherwise only allow Popover to go up to edge (like bounded).

It is conceivable that another factor for the threshold could be the _Popover's_ percentage area cropped. Its unclear to me if this would be useful or not though.

![popover-semi-bounded.png]()



## Tip

As mentioned in the introductory anatomy Tip is a sub-component of Popover. Its job is to visually hint the relationship between Popover and Target.

Our system assumes that Tip has a pointer on top and base on bottom. In other words that at rest (no rotation) Tip is pointing upward. This additional knowledge required by the algorithm could be configurable but either way the knowledge is necessary to be sure our rotation logic will be correct (see below).

We attempt to position Tip along the main axis of Popover/Target but just as the position of Popover can be affected by Frame so to can Tip's position be affected by the positioning relationship between Target and Popover. The logic is:

* Along main-axis Tip is between Popover and Target
* Along cross-axis Tip is at center between the closest before edge and the closest after edge from either Target or Popover.
* Tip pointer is adjacent (faces) Target. This implies rotation.

![tip-centering.png]()
![tip-rotation.png]()



## Edge Cases

There are two edge-cases that can lead to poor or unusable UX. These edge-cases can be overcome with careful attention.

### Appendix A: Infinite Loop

This section discusses why we must specially consider Tip dimensions when calculating a zone's fit rank.

![edge-case-infinite-loop.png]()

If Tip was not specially handled then it would be possible for an infinite loop of zone rank changes to occurs in some cases. Consequently the Popover would enter an infinite loop of repositions. This issue affects Class 2 and 3 but not 1 because of how its algorithm works.

* _given an optimal-zone change in the opposite axis_
* Tip will be moved such that it changes from extending length of Popover in one orientation to the other. For example the Tip goes from being on the right side of Popover (extends width) to below it (extends height)
* Popover dimensions, upon being positioned into the new optimal-zone, have changed, as a result of the new position assigned to Tip
* Popover dimension change triggers a new zone-ranking calculation to be made
* _given the new optimal-zone's rank difference with previous optimal-zone is smaller than Tip's main axis length_
* the zone-ranking calculation determines that what was previously the optimal-zone is once again the optimal-zone
* infinite loop

A non-general solution to this problem is to always add the Tip's main-axis length to Popover's main-axis length when calculating a zone's fit rank.

TODO integrate this
To compare zone dimensions with Popover without causing infinite layout loops (See Appendix A) we must add the Tip's main-axis length to the Popover's dimension which would fall upon the main-axis for the respective zone. For example when calculating Popover fit within Top Zone add Tip length to the height of Popover; for Right Zone add Tip length to width of Popover; etc. This is necessary to avoid infinite layout loops (to learn why see Appendix A "Infinite Loop").

### Appendix B: Jitter Magnification

![edge-case-jitter.png]()

When some change in the arrangement or size of components leads to a change in zone ranks then Popover will in turn be repositioned to reflect its new optimal placement. This is generally good but problems can arise when the ranks are very tight by which we mean that the arrangement, sizing, etc. of components is on the precipice of other zones being optimal. Analogy: imagine weights on a scale that are within sand-grains of each other.

The problem with this condition is that minor jitters are magnified into excessive layout changes, generally hurting the user-experience. For example a jittery Popover is distracting reading of something else in view or worse yet the content being read is _in said jittering popover_. A more concrete example: imagine Frame is the window and zones ranks are tight. If the user jitters the window's scroll position then so too goes Popover!

The underlying problem is that we have tightly coupled jitter from the inputs (arrangement, size, etc. of Target, Popover, Frame) to pass right through to our output (zone ranking). The solution is to use [hysteresis](https://en.wikipedia.org/wiki/Hysteresis) to filter out these minor fluctuations. Only then can we maintain a stable layout in the face of unstable inputs. The exact threshold to be used in the hysteresis should probably be configurable since not all use-cases will have the same stability requirements.
