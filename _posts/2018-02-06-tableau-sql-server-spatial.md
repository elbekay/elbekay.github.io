---
layout: post
title: Tableau & SQL Server Spatial
date: '2018-02-06T15:59:43+11:00'
tags: []
tumblr_url: https://lbk.io/post/170562388033/tableau-sql-server-spatial
---
This week we released a prerelease (see: [https://prerelease.tableau.com](https://prerelease.tableau.com)) version of Tableau which supports SQL Server Spatial which allows you to connect directly to spatial data stored in SQL. This starts the extension of our support which vegan in 10.2 for Shape File support.

I decided to start playing this week and thought I’d write a short blog on the topic. I’m not a GIS or SQL Spatial expert so my approach may not be perfect but hopefully it helps you get started with our pre-release as well.

Most of my reading was from the following link [https://docs.microsoft.com/en-us/sql/relational-databases/spatial/spatial-data-sql-server](https://docs.microsoft.com/en-us/sql/relational-databases/spatial/spatial-data-sql-server) and my data came from the Melbourne data portal [https://data.melbourne.vic.gov.au/](https://data.melbourne.vic.gov.au/)

I took a couple of datasets, one on the location of landmarks around Melbourne (parks, shrines, churchs, stations and the like) and a dataset of the locations for Melbourne City’s Bike Sharing stations.

The idea was to visualize parks which were within walking distance of bike stations so that someone could workout which parks were easy to visit via the bike sharing scheme. Ended up with a simple dashboard where you can adjust the distance and get a map and list of parks:

[<figure class="tmblr-full" data-orig-height="785" data-orig-width="1287" data-orig-src="https://i.imgur.com/UuG634s.png"><img src="https://66.media.tumblr.com/a1d2fb64402b0243f8aacdf06e6880df/tumblr_inline_p814kaBcZF1qbqa5u_540.png" alt="image" data-orig-height="785" data-orig-width="1287" data-orig-src="https://i.imgur.com/UuG634s.png"></figure>](https://i.imgur.com/UuG634s.png)  
Click image to expand.

The data from each is essentially the name of the landmark or station, a theme/sub-theme (e.g. park) for the landmark, and the lat/lon.

I simply loaded the data into SQL Server using SSMS’s Import Data Wizard, then converted the lat/lon to point data using the inbuilt POINT(Lat, Lon, SRID), for example:

    
    SELECT 
    [Feature Name],
    [Theme],
    POINT([Lat], [Lon], 4326) as [Location]
    FROM …

Note: 4326 is a reference to the co-ordinate system WGS84

We can then use custom SQL in Tableau to perform a spatial join. There is also already in the prerelease an Intersects type spatial join in the Tableau GUI.

    
    SELECT 
     &nbsp; &nbsp;[Theme]
     &nbsp; &nbsp;,[Sub Theme]
     &nbsp; &nbsp;,[Feature Name]
     &nbsp; &nbsp;,bs.[Featurename] AS [Bike Station Name]
     &nbsp; &nbsp;,lm.[Latitude]
     &nbsp; &nbsp;,lm.[Longitude]
     &nbsp; &nbsp;,lm.[Location] as [Landmark Location]
     &nbsp; &nbsp;,bs.[Location] as [Bike Station Location]
     &nbsp; &nbsp;,bs.Location.STBuffer(\<Parameters.Distance From\>).STIntersects(lm.Location) AS [Meets Distance]
     &nbsp; &nbsp;,bs.Location.STDistance(lm.Location) as [Distance]
    FROM 
     &nbsp; &nbsp;[melbdata].[dbo].[Landmarks] lm
     &nbsp; &nbsp;, [melbdata].[dbo].[BikeShareLocations] bs

Most of this is pretty standard, the field within the \<\> is a Parameter which lets us set the distance we’re willing to walk and returns a True/False if we’re within that distance.

Once it’s in Tableau it just becomes like any other shape file source and I’m able to double click on Landmark or Bike Station location and have it land on a map without working directly with Lat/Lons!

<figure class="tmblr-full" data-orig-height="593" data-orig-width="856" data-orig-src="https://i.imgur.com/d9DwDD4.png"><img src="https://66.media.tumblr.com/3b8efe96223a816e2c2ad14bd86a435e/tumblr_inline_p814kaAkvR1qbqa5u_540.png" alt="image" data-orig-height="593" data-orig-width="856" data-orig-src="https://i.imgur.com/d9DwDD4.png"></figure>  
  
<figure data-orig-height="539" data-orig-width="205" data-orig-src="https://i.imgur.com/E9RB7qA.png"><img src="https://66.media.tumblr.com/717d4194d82fda0fd3d4efa8e89b9f7d/tumblr_inline_p814kbEVam1qbqa5u_540.png" alt="image" data-orig-height="539" data-orig-width="205" data-orig-src="https://i.imgur.com/E9RB7qA.png"></figure>
