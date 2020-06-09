---
layout: post
title: Forecasting in Tableau with R
date: '2016-09-04T23:18:49+10:00'
tags: []
tumblr_url: https://lbk.io/post/149929500368/forecasting-in-tableau-with-r
---
I am relatively new to R&Tableau integration (compared to people like Bora: [https://boraberan.wordpress.com](https://boraberan.wordpress.com)) so this post is more for myself to jot down my own learning but hopefully it helps someone who is familiar with Tableau but new to R or vice-versa. I’ll update this post or add more as a I learn more.

## Install R & Rserve

Tableau recquires Rserve to interface with R. If you know you already have an Rserve instance in your environment then note down the server address / port / credentials of it for later.

If you don’t have an Rserve instance ready: Install R and (optionally) RStudio.

You can find R (for Windows) here:&nbsp;[https://cran.r-project.org/bin/windows/base/](https://cran.r-project.org/bin/windows/base/)

and RStudio here:&nbsp;[https://www.rstudio.com/products/rstudio/download3/](https://www.rstudio.com/products/rstudio/download3/)

Once installed start R or RStudio and install & start Rserve:

> install.packages(“Rserve”);
> 
> library(”Rserve”);
> 
> Rserve();

![image](http://imgur.com/mI6ssWe.png)

## Connect to Rserve

In Tableau Desktop. Select the Help -\> Settings & Performance -\>&nbsp;Manage Exernal Service

If you’ve just done a default install of Rserve on your own machine then you will want _localhost_ and _6311_ for the port, there will be no username/password by default.

![image](http://imgur.com/tj5lBKN.png)

## Interfacing with R

There are 4 functions in Tableau that allow you to interface with R, they all begin with _SCRIPT\__ with the next word the type of data Tableau expects the function to return, either bool(ean), float, int or str(ing).

![image](http://imgur.com/8KNa195.png)

Calculated fields that use R scripts are a type Table Calculation in Tableau. At a high-level this means that the calculation is based off the data points that are currently in the worksheet.

Below is a really basic example of this, in this scenario I have connected to some data in Tableau ([http://pastebin.com/svsfhi7Q](http://pastebin.com/svsfhi7Q)), and plotted Production by Month.

Using the SCRIPT\_REAL function to have R to calculate the standard deviation, you can think about this as Tableau passing the monthly data points on production currently in the view to R and receiving the standard deviation in response.

![image](http://imgur.com/9lPFNpq.png)

In this case there is a little syntax to pass our measure of interest to R which is _.arg1 - .arg1_ will essentially be replaced by_&nbsp;_the first measure after the comma; _arg2_ would be after another comma and so on.

Looking at forecasting example to leverage a forecasting model not included in Tableau here&nbsp;I am following some of [Ed Boone’s youtube intro](https://www.youtube.com/watch?v=zFo7QixEKvg)&nbsp;on ARIMA modeling in R which uses public domain data Rob Hyndman [posted](http://robjhyndman.com/tsdldata/data/beer.dat) on beer production, you can watch teh video here:&nbsp;&nbsp;and I made a pastebin of the data here as a csv&nbsp;[http://pastebin.com/svsfhi7Q](http://pastebin.com/svsfhi7Q)

The SCRIPT\_REAL function is used again, but our R script is a little more in-depth this time.&nbsp;

![image](http://imgur.com/S2MFEQn.png)

A parameter in Tableau so that I can control the number of periods to forecast.

The arguments to the ARIMA function are from the intro video above, this is something I am still learning about myself and can’t comment on the specifics of their selection.

Tableau doesn’t allow you to append rows to the dataset (with the exception of the inbuilt forecasting function which is a special case). So there are some lines of R to trim or shift the original data _n periods_ and append the forecast data to the end.&nbsp;I add very similar calculated fields for the prediction intervals.

In the ending up with a worksheet that looks like the below, with a parameter that allows me to choose the number of months to forecast; plus a helper function that filters the data so I only see 3x the number periods selected (otherwise its a little busy with almost 500 months of data).

![image](http://imgur.com/T1hU2rh.png)

