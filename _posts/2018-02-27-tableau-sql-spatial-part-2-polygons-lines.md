---
layout: post
title: 'Tableau & SQL Spatial Part 2 - Polygons, Lines & Points'
date: '2018-02-27T12:40:14+11:00'
tags: []
tumblr_url: 'https://lbk.io/post/171328516293/tableau-sql-spatial-part-2-polygons-lines'
published: true
---
In this post we’ll look at points in polygons and polygons intersecting other polygons. This is a follow up to my previous post on getting started with Tableau and Microsoft SQL Spatial. I’m not a MS SQL spatial expert by any means, if you see me doing something silly contact me.

Previous post here:&nbsp;[http://www.lbk.io/post/170562388033/tableau-sql-server-spatial](http://www.lbk.io/post/170562388033/tableau-sql-server-spatial)

Note on the datasets: I’m using Mesh Blocks from the Australia Bureau of Statistics. All statistical areas in Australia are built up from Mesh Blocks, ABS describes them as a sort of geographic building block. Other data sets (like point/line locations) are randomly generated and do not represent any individual.&nbsp;

You can read more about Mesh Blocks here - [http://www.abs.gov.au/websitedbs/d3310114.nsf/home/australian+statistical+geography+standard+%28asgs%29](http://www.abs.gov.au/websitedbs/d3310114.nsf/home/australian+statistical+geography+standard+%28asgs%29)

## Points in Polygons

From time to time I’m asked something along the lines of “I have a list of points representing customers/stores/warehouses/network towers/etc. that I have a lat/lon, I also have a set of geographical boundaries (post code, mesh bloc, business region) and I’d like to know which geographical area my points these fall into”.

In the prerelease of Tableau I’ve been using for these spatial posts there is now an spatial intersect join type that helps us figure out these sorts of questions.

In this example I have a list of Customers or Residents and I’d like to know the Statistical Area they fall within.

Customers (randomly generated locations)

<figure data-orig-width="632" data-orig-height="450" class="tmblr-full"><img src="https://66.media.tumblr.com/261f971b5abce778a7eafad8820e3a86/tumblr_inline_p4scf5kJXw1qbqa5u_540.png" alt="image" data-orig-width="632" data-orig-height="450"></figure>

Mesh Blocks

<figure data-orig-width="646" data-orig-height="514" class="tmblr-full"><img src="https://66.media.tumblr.com/f09fe8c750b59fa378aaa7a2d1372b3e/tumblr_inline_p4scfekPmg1qbqa5u_540.png" alt="image" data-orig-width="646" data-orig-height="514"></figure>

Join

<figure data-orig-width="478" data-orig-height="286" class="tmblr-full"><img src="https://66.media.tumblr.com/b2387dd27d03fd8b0d6ae6b0229e6251/tumblr_inline_p4scfvEcNU1qbqa5u_540.png" alt="image" data-orig-width="478" data-orig-height="286"></figure>

Result

<figure data-orig-width="644" data-orig-height="559" class="tmblr-full"><img src="https://66.media.tumblr.com/2bcaaeeb9bcea8612b53672848612bcd/tumblr_inline_p4scgjtki01qbqa5u_540.png" alt="image" data-orig-width="644" data-orig-height="559"></figure>

Our Resident points have now been classified by their statistical area to which they sit, as well as meshblock in this case as well for examples sake.

## Polygons/Points Intersecting Polygons

What if we were doing some utility works and knew that we expected mesh blocks within a given buffer of 250m from the site to be impacted. How could we visualize that and highlight those impacted mesh blocks in Tableau?

We can use passthrough SQL for this, in this case I’ve made up a location of works at a given lat/lon, and our pass through SQL checks for intersection or not (returning a 1 in case of intersection, 0 otherwise).

<figure data-orig-width="685" data-orig-height="440" class="tmblr-full"><img src="https://66.media.tumblr.com/9f39cd54ac7f0391d35292f1f5424da6/tumblr_inline_p4scmsfmgl1qbqa5u_540.png" alt="image" data-orig-width="685" data-orig-height="440"></figure>

The result is we can visualize the meshblocks that will be impacted.

<figure data-orig-width="534" data-orig-height="467" class="tmblr-full"><img src="https://66.media.tumblr.com/e02340628f79ce1b958aec9a4a5e7b92/tumblr_inline_p4schkPJub1qbqa5u_540.png" alt="image" data-orig-width="534" data-orig-height="467"></figure>

Additionally our parameter gives us an easy to expand or contract the works area to see the changes. Adjusting from 250m to 100m for example:

<figure data-orig-width="639" data-orig-height="405" class="tmblr-full"><img src="https://66.media.tumblr.com/ebf52526a5a57fd5ff99742323119ccc/tumblr_inline_p4schxHam21qbqa5u_540.png" alt="image" data-orig-width="639" data-orig-height="405"></figure>

In another example what if we were going to build a new road, and wanted to start planning for resident engagement within 100m works of the road. If we know the route of the road:

<figure data-orig-width="572" data-orig-height="346" class="tmblr-full"><img src="https://66.media.tumblr.com/b97263f59896a97d32da040e2c9fd6c0/tumblr_inline_p4sci5ot6T1qbqa5u_540.png" alt="image" data-orig-width="572" data-orig-height="346"></figure>

We can buffer it, and then check the mesh blocks that would be impacted:

<figure data-orig-width="472" data-orig-height="494" class="tmblr-full"><img src="https://66.media.tumblr.com/f9e80ab460615ebc56270b6e4791c9b2/tumblr_inline_p4scibg2dO1qbqa5u_540.png" alt="image" data-orig-width="472" data-orig-height="494"></figure>

Like earlier this uses the fact that Tableau can use pass through SQL to return not only basic types like int/bool/string but also spatial types which allows us to plot lines, polygon intersections and so on.

<figure data-orig-width="556" data-orig-height="272" class="tmblr-full"><img src="https://66.media.tumblr.com/9f8d70c77778fbfa5a9a5850b9cdcd3e/tumblr_inline_p4scilqXOH1qbqa5u_540.png" alt="image" data-orig-width="556" data-orig-height="272"></figure><figure data-orig-width="503" data-orig-height="237" class="tmblr-full"><img src="https://66.media.tumblr.com/040b665385f19c1362ca0893f31f18b5/tumblr_inline_p4sciqkjAN1qbqa5u_540.png" alt="image" data-orig-width="503" data-orig-height="237"></figure>
