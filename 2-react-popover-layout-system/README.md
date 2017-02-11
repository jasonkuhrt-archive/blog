# React Popover Layout Algorithm

This article is an overview of the layout algorithm used for react-popover. If you are curious about what react-popover is or why it exists then you may want to read a recent article detailing its back-story.

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

Popover will be positioned within one of four zones. The zone that best fits Popover will be chosen. For each zone we calculate the result of subtracting Popover's dimensions from it. Then based on these results we bucket zones into three tiers of preference (each tier is less preferable than the previous):

1. zones have positive difference on both dimensions
2. zones have positive difference on just one dimension
3. zones have negative difference on both dimensions

Tier 1 zones are sorted by the sums of each zone's result, descending. So the fist zone in this list is that which fits Popover with the most extra area overall.

Tier 2 and tier 3 zones are sorted by each one's percentage of crop, ascending. So the first zone in this list is that which crops Popover the least overall.


![zone-scenarios.png]()



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

### Jitter

![edge-case-jitter.png]()

### Split Brain

![edge-case-split-brain.png]()
