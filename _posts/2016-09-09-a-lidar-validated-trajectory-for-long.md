---
title: 'A Lidar-Validated Trajectory for a Long Home Run'
date: 2019-10-08T04:17:00+01:00
draft: false
---

![](https://miro.medium.com/max/1000/1*W6GED-fAIn6eIjwBc7uDRw.png "A Lidar-Validated Trajectory for a Long Home Run - MLB Technology Blog")  

Background
==========

MLB’s home run distance projections are generated by the Statcast system in real-time. This home run distance estimate was independently verified using a trajectory solution informed by a lidar measurement of its impact location. The unusual appearance of this home run makes it a good candidate to illustrate how MLB uses lidar to independently calculate a projected home run distance.

[Lidar](https://en.wikipedia.org/wiki/lidar) is a measurement technique in which a laser is bounced off surfaces and detected to generate information about an environment. MLB regularly performs lidar scans of all 30 MLB ballparks. The raw data is used to generate accurate 3D ballpark maps called “point clouds”, which allow for easy identification and measurement of specific features of the ballparks, such as the precise location of walls, seats, banners, or any other visible reference point. A relative measurement within this point cloud is accurate with ISO traceability down to 3 mm.

Home Run Distance Verification
==============================

First, we need to identify the ball’s landing point. In this case, Garcia’s home run impacted the “AL East 2008 Champions” banner at a point indicated by the green reticle in Figure 1. After watching the video of the Garcia home run, one might come away with the impression this banner hangs above and behind the upper deck in left field. It may even look like it bounces behind the upper deck, in which case the reported projection of 459 feet would seem well short of reality as speculated in the YouTube comments shown in Figure 2.

Figure 1. Avisail Garcia’s July-20 home run impacted the “AL East 2008 Champions” banner over the left field stands. The impact point is indicated by the green reticle.

Figure 2. A sample of comments from YouTube conveys uncertainty in the distance estimate.

The impact point and the plate origin can be identified in the lidar point cloud for Tropicana Field as shown in Figure 3. The straight-line distance to the impact point is approximately 389 feet from the tip of home plate as labelled in Figure 3. This point is also about 89 feet above the level plane defined by home plate (z=0). \[The MLB coordinate convention considers the origin (0,0,0) the tip of home plate with z antiparallel to gravity, y is towards the pitcher, x is positive towards first base.\]

Figure 3. The impact point on the banner is 389 ft from the tip of home plate and about 89 ft above the ground (z=0).

Tropicana Field has a lot going on under its dome, including its [infamous](https://www.mlb.com/groundrules/venue-12) [catwalks](https://www.tampabay.com/Rays-ballpark-engineers-computer-modeled-fly-balls-to-design-a-roof-that-would-stay-out-of-play_170255758/). The illusion of the banner placement is revealed in the perspective shown in Figure 4. From this perspective, it is clear that those banners hang almost directly _above_ the left field wall. They are not behind the upper deck stands, though it might have looked that way from the high-home broadcast camera’s perspective.

Figure 4. These banners at Tropicana Field are hanging above the D ring and below the C ring at Tropicana Field.

In Figure 5, a plumb line from the banner impact point shows its projection downward is only about six rows back from the outfield fence and about 74 feet down.

Figure 5. A plumb line from the impact point falls about six rows back from the OF wall.

This lidar-measured coordinate gives us the first important ingredient in generating a lidar-validated home run distance projection. In fact, we give this late-trajectory impact coordinate heavy control over the trajectory solution we will eventually find. The ingredients we need to generate a trajectory are shown in Table 1:

Table 1. List of ingredients and sources to create a lidar-validated trajectory solution.

Flight time or time from impact (bat) to impact (banner) is harvested from the broadcast video down to the closest frame. The flight time and impact coordinate together make the high-confidence late-trajectory coordinate. Next, we pull in local environmental conditions that will affect our eventual trajectory solution. In this case, these aren’t that interesting since Tropicana Field has a dome. In general, air density and wind conditions help to model the ball’s flight path more precisely. Finally, we use the initial conditions of the actual trajectory measured with high confidence: exit velocity, launch angle, and spray direction.

We’ve collected all the ingredients, now we need to use them to create this trajectory solution. Thankfully, esteemed physicist Alan Nathan has done some of this work already by [parameterizing the equations of motion for a baseball trajectory](http://baseball.physics.illinois.edu/trajectory-calculator-new.html). Our treatment closely follows [Dr. Nathan’s method](http://baseball.physics.illinois.edu/TrajectoryAnalysis.pdf), including drag and lift coefficient ranges from Robert Adair’s [The Physics of Baseball](https://www.amazon.com/Physics-Baseball-3rd-Robert-Adair/dp/0060084367).

The trajectory solution is calculated using a nonlinear-optimization or solver algorithm. In essence, we will tether the solution to parameters in which we have high confidence (launch velo, launch angle, temperature, etc) and allow the optimization algorithm to vary other parameters within physically realistic constraints. The goal of the optimizer is to minimize the sum of the squares of the differences (△x²+ △y²+ △z²) between the calculated-trajectory’s coordinates at the time of impact and the lidar-measured impact coordinates. We consider a solution convergent when the residual (△x²+ △y²+ △z²) is essentially zero.

The result of all of this is a closed-form trajectory solution which matches our high-confidence observations of what happened. You will note that this trajectory is calculated independently from in-flight tracking data and the derivative distance projections are independent of our tracking vendor’s projected distances.

The lidar-validated trajectory solution is plotted in Figure 6. The dashed lines represent the projected trajectories back to ground level that would have occurred had this HR not impacted the banner. The 459-foot projected range at the z=0 root is called out in the plot.

Figure 6. Trajectory plot of height and total distance as functions of time with projected trajectories after impact.

In other words Mr. Garcia’s home-run ball would have traveled another 69 feet in total distance (sqrt(x²+y²+z²)) by the time it descended from the 89-foot height (z) of the banner impact site, arriving at a total distance projection of 459 feet.

Conclusion
==========

This method leverages an observed and well-measured impact coordinate and produces a physics-based trajectory that matches the known initial and environmental conditions. The calculated trajectory results in a home run distance projection that can earn the confidence of careful observers and casual fans alike.

Of course, this treatment generates a trajectory solution which is an _estimate_ of the observed trajectory. As Nathan notes at the end of [his paper](http://baseball.physics.illinois.edu/TrajectoryAnalysis.pdf), the physics model contains spin, drag, and lift assumptions that can vary slightly from home run to home run. In addition, our generalizations about wind speed and direction are, at best, too coarse to precisely recreate a ball’s trajectory. That said, the trajectory estimate works well to convey how far a home-run ball _would have gone_ had it not been terminated by a collision. In fact, this lidar-based trajectory solution agrees very well Statcast’s live home run distance estimates when a ball is tracked late into the trajectory.

Prior work on home-run distance estimation includes Greg Rybarczyk’s fantastic [Hit Tracker](http://www.hittrackeronline.com), IBM’s Tale of the Tape which became the MCI Home Run Program, and former Yankee’s PR director [Red Patterson](https://www.mlb.com/video/mantles-tape-measure-home-run/c-7604605).

  
  
from Hacker News https://ift.tt/2LRTY3D