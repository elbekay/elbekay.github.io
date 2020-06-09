---
layout: post
title: Getting Started with TabJolt
date: '2015-06-30T14:48:19+10:00'
tags: []
tumblr_url: https://lbk.io/post/122826591098/getting-started-with-tabjolt
---
Tableau recently launched TabJolt which is a “point-and-run” load and performance testing tool designed to work with Tableau 9 Server. You point TabJolt at a selection of workbooks on your server and TabJolt runs a set of views and interactions with the vizzes while capturing performance metrics.&nbsp;

By default TabJolt will capture metrics related to viz performance (load time, error rate, throughput etc) but can also capture system metrics via PerfMon and Tableau server process stats via JMX.

TabJolt is one tool to help you answer the question of how your server will behave under load and as your user base scales.

Before we get started the best place to go for assistance with using Tabjolt is our [Server Admin](http://community.tableau.com/community/forums/server-administration)forums.

The high level steps we will need to complete:

1. Configure Tableau Server environment for Tabjolt.
2. Prepare a host for Tabjolt installation. This should be a separate server to any Tableau Server or Worker.
3. Configure and Run Tabjolt

Here are the installs/tools what you need:

1. Tableau Server instance, standalone or clustered.   
2. A machine to run TabJolt on with the following:   

  1. JRE

    1. [http://www.oracle.com/technetwork/java/javase/downloads/jre7-downloads-1880261.html](http://www.oracle.com/technetwork/java/javase/downloads/jre7-downloads-1880261.html)  
  2. PostgreSQL (see setup instructions before installing)

    1. [http://www.enterprisedb.com/products-services-training/pgdownload#windows](http://www.enterprisedb.com/products-services-training/pgdownload#windows)  
  3. TabJolt unpackaged

    1. ZIP file from:&nbsp;[https://github.com/tableau/tabjolt/releases](https://github.com/tableau/tabjolt/releases)  
3. A decent text editor like Notepad++ or Sublime Text

  1. [https://notepad-plus-plus.org/](https://notepad-plus-plus.org/)  
OR
  2. [http://www.sublimetext.com/](http://www.sublimetext.com/)  

For the purposes of this blog I’m assuming you already have a Tableau Server instance running and ready to go and won’t be covering the install and config of Server.

**Tableau Server Changes**

Some minor changes to Tableau Server are required for TabJolt to run.

1. Enable readonly access to the PostgresSQL database.

Open up a command line window, and from the Tableau Server bin directory run:

    tabadmin dbpass --username readonly p@ssword

2. (Optional) Enable JMX counters to capture performance data on Tableau processes. Also from the bin directory run the following:

    tabadmin set service.jmx\_enabled true tabadmin stop tabadmin configure tabadmin start

## TabJolt Host Config

Unless specified the below is to be performed on the server you have set aside as the Tabjolt Server

Install JRE with the defaults.&nbsp;

Install PostgreSQL, when prompted for a password use&nbsp;“testresults” otherwise leave all the defaults.

Open pgAdmin and connect to localhost

<figure data-orig-width="436" data-orig-height="275" class="tmblr-full"><img src="https://66.media.tumblr.com/40125e681d48a38fc10e0e8c20cd38c2/tumblr_inline_nqac62KFD01qbqa5u_540.jpg" alt="image" data-orig-width="436" data-orig-height="275"></figure>

Select the SQL Query icon (magnifying glass in top right of below image)

<figure data-orig-width="188" data-orig-height="142"><img src="https://66.media.tumblr.com/d924dac44cc056fd78736b3ba3d60735/tumblr_inline_nqac6sSrPi1qbqa5u_540.jpg" alt="image" data-orig-width="188" data-orig-height="142"></figure>

Open&nbsp;“postgresDBSchemaPart1.sql” and run it.

<figure data-orig-width="528" data-orig-height="157" class="tmblr-full"><img src="https://66.media.tumblr.com/9b214622f874a18b2314c407dbcd1032/tumblr_inline_nqac9mw1bO1qbqa5u_540.jpg" alt="image" data-orig-width="528" data-orig-height="157"></figure>

Wait a moment and press F5 to refresh the pane until the PerfResults DB appears.

Once it appears select it and then open the query editor again, open and run&nbsp;“postgresDBSchemaPart2.sql”, its important you have PerfResults selected when you do this step.

<figure data-orig-width="375" data-orig-height="202" class="tmblr-full"><img src="https://66.media.tumblr.com/fb70f50805a7b544610a539eadd3ed2e/tumblr_inline_nqac9iFZTS1qbqa5u_540.jpg" alt="image" data-orig-width="375" data-orig-height="202"></figure>

The next steps are to configure _ServerTestConfig.yaml_, _dataretriever.config_ and _vizpool.csv_

Open each in your selected text editor.

_ **vizpool.csv** _ tells TabJolt which views to load, the syntax is straightforward, below is an example for a selection of the built in example vizzes. The number at the end should be unique (according to documentation).

    /views/Superstore/Overview/1 /views/Superstore/Product/2 /views/Superstore/Shipping/15 /views/Superstore/Performance/18 /views/Superstore/SalesForecast/21 /views/Superstore/CommissionModel/25 /views/Superstore/OrderDetails/12

_ **ServerTestConfig.yaml** _ tells TabJolt which URL to connect to your Tableau Server on as well where to find the Tableau Server PostgreSQL DB, and which user to login to Tableau Server as. You will at a minimum want to start by updating the fields highlighted below to suit your environment, don’t forget to enable the readonly user for the Tableau Server PostgreSQL DB!

    default { # tableau server host name: hostUri: [http://tab3.tabtest.com](http://tab3.tabtest.com) # HTTP User-Agent userAgent: java-client-requests # server database connection information database: connectionString: jdbc:postgresql://%s:%s serverName: tab3.tabtest.com port: 8060 databaseName: workgroup userName: readonly password: readonly driver: org.postgresql.Driver unicode: true encoding: utf-8 users: - !!com.tableausoftware.test.server.configuration.User name: password: } 

Modifying&nbsp;_ **dataretriever.config** _&nbsp;is optional but provides additional information on system and process performance. I will describe a few of the sections you might be interested in updating to capture metrics.

Impersonation settings may be needed to allow TabJolt to connect to the PerfMon service if the current RunAs account doesn’t have access.

    \<settings\> \<impersonation userName="user" password="password" domain="domainName"/\> \</settings\>

The next section defines which PerfMon counters to capture, you can leave the defaults or add others:

    \<counterGroup name="machineStatus" samplingInterval="15"\> \<counters\> \<counter category="Memory" name="% Committed Bytes In Use|Available Bytes|Committed Bytes|Page Faults/sec|Transition Faults/sec" instance=".\*"/\> \<counter category="Processor" name="% Processor Time|% Privileged Time|% User Time|% Interrupt Time|Interrupts/sec" instance="\_Total"/\> \<counter category="LogicalDisk" name="% Disk Time|Current Disk Queue Length" instance="\_Total"/\> \<counter category="Network Interface" name="Packets/sec|Packets Received Errors|Bytes Received/sec|Bytes Sent/sec|Bytes Total/sec|Current Bandwidth" instance=".\*"/\> \</counters\> \</counterGroup\>

The components section defines which Tableau processes to gather performance metrics from (uses JMX). You’ll notice I’ve commented out vizqlserver#1 and dataserver#1 as I am only running 1 instance of each on my Tableau Server and Worker, you will need to update this section as necessary to suit your environment, e.g. if you have 2x of each of these processes then don’t uncomment these lines or if you don’t want to capture the searchservice the comment out that line etc.

    \<counterGroup name="tableauProcess" samplingInterval="15"\> \<counters\> \<counter category="Process" name="% Privileged Time|% Processor Time|% User Time|Elapsed Time|Page Faults/sec|Private Bytes|Thread Count|Working Set|Working Set - Private|Working Set Peak|% Privileged Time|% Processor Time|% User Time|Elapsed Time|Page Faults/sec|Thread Count" instance="backgrounder.\*|dataserver.\*|httpd.\*|tabrepo.\*|tabspawn.\*|tabsvc.\*|vizqlserver.\*|wgserver.\*|vizportal.\*"/\> \</counters\> \</counterGroup\>

Finally under the hosts section add each host you wish to monitor in your Tableau Server instance. Simply select and copy-paste from \<host to \</host\>. At this point you can enable / disable different collection groups too, the first two are PerfMon, the remaining 3 are JMX counters.

    \<hosts\> \<host name="tab3.tabtest.com"\> \<applicableCounterGroups\> \<applicableCounterGroup\>machineStatus\</applicableCounterGroup\> \<applicableCounterGroup\>tableauProcess\</applicableCounterGroup\> \<!--enable the following section only after you jmx counter for tableau--\> \<applicableCounterGroup\>vizqlserver\</applicableCounterGroup\> \<applicableCounterGroup\>dataserver\</applicableCounterGroup\> \<applicableCounterGroup\>vizportal\</applicableCounterGroup\> \</applicableCounterGroups\> \</host\> \<host name="tab4.tabtest.com"\> \<applicableCounterGroups\> \<applicableCounterGroup\>machineStatus\</applicableCounterGroup\> \<applicableCounterGroup\>tableauProcess\</applicableCounterGroup\> \<!--enable the following section only after you jmx counter for tableau--\> \<applicableCounterGroup\>vizqlserver\</applicableCounterGroup\> \<applicableCounterGroup\>dataserver\</applicableCounterGroup\> \<applicableCounterGroup\>vizportal\</applicableCounterGroup\> \</applicableCounterGroups\> \</host\> \</hosts\>

Once&nbsp;_ **ServerTestConfig.yaml,&nbsp;**** vizpool.csv and&nbsp; **_** _dataretriever.config_** are configured we’re ready to start testing.&nbsp;

If you haven’t already done so open up a cmd prompt and switch to the tabjolt directory.

The basic syntax to use is:

    go --t=testplans\\<Test Plan Name\> --d=\<duration\> --c=\<concurency\>

Where:

> \<Test Plan Name\> is the name of the test plan to run of the two available; InteractVizLoadTest or ViewVizLoadTest. I typically run InteractVizLoadTest as it simulates a mix of interacting with (filtering, selecting, etc) and viewing.
> 
> \<Duration\> is the duration to run the test in seconds.
> 
> \<Concurrency\> is the number of concurrent samples simulate in parallel.

I typically start with the following test for 30 seconds and 1 concurrent samples, iron out any errors and then ramp up the duration to 10 minutes and 10′s or 100′s of threads.

    go&nbsp; --t=testplans\InteractVizLoadTest.jmx --d=30 --c=1

If all has gone well you should get output similar to the below while the test runs. If you receive any errors or exceptions then jump onto the Tableau [Server Admin](http://community.tableau.com/community/forums/server-administration) forums and post the error, screenshots and configs to help us help you fix it.

<figure data-orig-width="644" data-orig-height="618" class="tmblr-full"><img src="https://66.media.tumblr.com/8b08e57c1369293687bfb00ef8042b72/tumblr_inline_nqqq7x09Qb1qbqa5u_540.jpg" alt="image" data-orig-width="644" data-orig-height="618"></figure>

Here we see 1 sample and 0 errors, along with the average response time for the viz interaction. We also see a successful wrap up that the results have been stored in RUN ID 3.&nbsp;

After the tests have completed locate the results workbook which should be named **_PerformanceViz.twb_** and resides in the tabjolt folder.

There are a number of different worksheets within the viz to explore, below are a couple showing a plot of the different interaction steps taken in the first image and the second showing host CPU and memory usage.

Hopefully this post should give you a quick intro into getting started with Tabjolt. If you run into any errors or have any additional questions then drop by to our [Server Admin](http://community.tableau.com/community/forums/server-administration)forums.

<figure data-orig-width="982" data-orig-height="622" class="tmblr-full"><img src="https://66.media.tumblr.com/3a8c238d32170ee7679bf4223d58df25/tumblr_inline_nqqqitBnU91qbqa5u_540.png" alt="image" data-orig-width="982" data-orig-height="622"></figure><figure data-orig-width="975" data-orig-height="627" class="tmblr-full"><img src="https://66.media.tumblr.com/b5977afce9983e03617dfc7391b5f45e/tumblr_inline_nqqqjhW7201qbqa5u_540.png" alt="image" data-orig-width="975" data-orig-height="627"></figure>
