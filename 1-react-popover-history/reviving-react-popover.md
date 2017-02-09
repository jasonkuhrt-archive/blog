# Reviving React Popover

## The Origin

A few years ago while I was still toiling away at littleBits I worked on a particular mission with Colin Vernon (product), Shem Rajoon (design), Alin Cosmanescu (electrical engineering), and others, to improve the the cloudBit onboarding user experience. One of the many tasks was to redo the littleBits circuit diagrams we used for our setup instructions. We wanted them to communicate more directly via improved animation and interactive buoys highlighting key sections with additional details. A buoy, once selected, would open a popover (a [particular kind of GUI component](https://en.wikipedia.org/wiki/Popover_(GUI))).

We knew we wanted a flexible system. I set a high bar for ourselves by committing to creating a system that would adapt to arbitrary diagrams with arbitrary buoy placements in arbitrary layout contexts. I wanted a general solution that could be used in all cases without the overhead of needing to know layout details whatsoever. In other words an algorithm smart enough to automatically find the best possible position for a box relative to the position of another box inside some bounds.

We called this component `react-popover` and quickly open-sourced it.

## The Slumber

Fast forward roughly nine months, enter 2016. I had moved on from littleBits and in so doing lost my npm and repo access to `react-popover`. I also became busy with other things like pursing my interest in Haskell and welcoming my second child to Mother Earth! So throughout 2016 a tiny community grew around `react-popover` but their bug reports, feature proposals, and pull-requests languished idly for the most part (evidently littleBits was focused on other things).

By autumn financial realities had caught up and job hunt replaced Canadian paternity leave. I was not yet adept with Haskell so returned to comfortable territory: JavaScript, react, devOps, full stack, etc. Suddenly maintaining a library like react-popover had renewed relevancy to me too. But still littleBits was mute–I couldn't seem to get ownership back... Then a few weeks ago my ex-colleague from littleBits, Adrian Schaedle, added me back as an admin of `react-popover` out of the blue?! Hurray! Thanks Adrian and thanks littleBits.

## The Reboot

Jumping back into most any project after two years is probably going to be non-trivial; `react-popover` is no exception. While there are ok user-docs and a few demos the the code itself is written by a former me which as we know might as well be another person. In this case the "other person" seems to have been writing some rather suspect code... Darn you Jason :) The salient issues are:

* lack of abstraction and generally complicated
* procedural code including fidgety layout logic
* no static types
* no tests

To rebuild my confidence and command of the codebase I have decided to carefully review the library's core purpose: algorithmic layout. Specifically I want to clearly _capture_ what it aims to achieve by using visual diagrams that illustrate how rules play out in various cases [1]. Not only will this be valuable for me now to reestablish my connection to the project but I think it could:

* help users understand the library
* make it easier to implement for other UI frameworks e.g. Cycle
* ease future maintenance because a good visual reference aids comprehending what dense layout code and the various domain terms are trying to achieve

My review and accompanying diagrams will be published in the next few days.

[1] At first my diagrams will be static however I am interested in making them "live", not unlike the diagrams found in essays by Bret Victor.
