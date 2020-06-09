---
layout: post
title: Tableau REST API & PowerShell Quickstart
date: '2016-04-03T09:08:58+10:00'
tags:
- tableau
- server
- restapi
- rest api
tumblr_url: https://lbk.io/post/142144850713/tableau-rest-api-powershell-quickstart
---
&nbsp;This is a quick intro into using PowerShell to invoke the Tableau Server REST API. The REST API allows you to programmatically query and manage Tableau Server assets like users, groups, workbooks, projects, sites, data sources, etc. The trickiest parts I see for users new to the REST API are what do I need to sign-in and how do I get and use the Auth Token, both of which are covered here.

As this is designed to be a quick intro I’m omitting things like error handing, helper functions etc. that you would have in the real world.

If you’re reading this but you’re just as happy to use Python then I recommend the much more complete resource and Python library implementation of the REST API here:&nbsp;[http://tableauandbehold.com/tableau-rest-api-resources/](http://tableauandbehold.com/tableau-rest-api-resources/)

**Let’s get started and open the PowerShell ISE…**

First off we need to sign in. The URL to sign in is&nbsp;[http://my-server/api/2.0/auth/signin](https://onlinehelp.tableau.com/current/api/rest_api/en-us/help.htm#REST/rest_api_concepts_auth.htm%3FTocPath%3DConcepts%7C _____ 3) and we also need to provide a request body which includes our username, password and [site](https://onlinehelp.tableau.com/current/server/en-us/sites_intro.htm) we want to login to.

Here is an example to generate the request body, you could just hardcode the username and password if you wanted to:

> $username = “username”  
> $password = “password”
> 
> # generate body for sign in  
> $signin\_body = (’\<tsRequest\>  
> &nbsp;\<credentials name=“’ + $username + ’” password=“’+ $password + ’” \>  
> &nbsp; &nbsp;\<site contentUrl=“” /\>  
> &nbsp;\</credentials\>  
> \</tsRequest\>’)

Next we need to make a POST request to our sign in URL, we use the PowerShell Invoke-RestMethod command for this. Selecting the menu _View -\> Show Command Add-In_ will show a useful reference / helper on the available options for this (and other) commands.

> $response = Invoke-RestMethod -Uri https://mytableauserver/api/2.0/auth/signin -Body $signin\_body -Method post

If you receive an error response or exception first work out if it’s syntax related, if it’s not then review [Handling Errors](https://onlinehelp.tableau.com/current/api/rest_api/en-us/help.htm#REST/rest_api_concepts_errors.htm%3FTocPath%3DConcepts%7C _____ 10), if you continue to have issues post on the [developer community forums](https://community.tableau.com/community/developers)

Now we’ve signed in.

**Time to get the all important Auth Token**

We need to get the Auth Token associated with our sign in. This is necessary to do anything further with the REST API as the token identifies you to the Server. If you have a look at the [response body](https://onlinehelp.tableau.com/current/api/rest_api/en-us/help.htm#REST/rest_api_concepts_auth.htm%3FTocPath%3DConcepts%7C _____ 3)&nbsp;documentation it will help with understanding the below.

> # get the auth token, site id and my user id
> 
> $authToken = $response.tsResponse.credentials.token  
> $siteID = $response.tsResponse.credentials.site.id  
> $myUserID = $response.tsResponse.credentials.user.id

**Set up headers for authentication**

Subsequent requests need the auth token to be provided along with the request.

> # set up header fields with auth token  
> $headers = New-Object “System.Collections.Generic.Dictionary[[String],[String]]”
> 
> # add X-Tableau-Auth header with our auth token  
> $headers.Add(“X-Tableau-Auth”, $authToken)

Now we can start to make useful requests.

**Getting a list of user details**

The endpoint for this is a get request to /api/api-version/sites/site-id/users/

We need to provide our authentication header along with this request

> $response = Invoke-RestMethod -Uri [https://tab.databender.net/api/2.0/sites/$siteID/users](https://tab.databender.net/api/2.0/sites/%24siteID/users) -Headers $headers -Method Get

Too see individual details about users we can access the right part of our response body, for example user names:

> $response.tsResponse.users.user.name  
> aexxxxxxx  
> _{ snip… }_  
> rcxxxxxxxxxx  
> Taxxxxxxxxxxxxxx  
> tlxxxx

Or site role:

> $response.tsResponse.users.user.siteRole  
> Unlicensed  
> _{ snip… }_  
> ServerAdministrator  
> Guest  
> Interactor  
> Publisher  
> ServerAdministrator

Or unique user id’s:

> $response.tsResponse.users.user.id
> 
> _{ output omitted … }_

**Listing workbooks available to a user**

To see all the workbooks owned by a user, or to which they have read permission, the endpoint is&nbsp; /api/api-version/sites/site-id/users/user-id/workbooks

> $response = Invoke-RestMethod -Uri [https://tab.databender.net/api/2.0/sites/$siteID/users/$myUserID/workbooks](https://tab.databender.net/api/2.0/sites/%24siteID/users/%24myUserID/workbooks) -Headers $headers -Method Get

In this case you’ll notice we have to pass in a user id in addition to the site id. I’m using my own user id gleaned from when I signed in, but you could easily substitute a user id from calls to the users endpoint.

Like before we can now access the request body:

> $response.tsResponse.workbooks.workbook
> 
> id &nbsp; &nbsp; &nbsp; &nbsp; : xdxxxxxx-xxxc-xxxx-xxxx-xxxxxxxcexxx  
> name &nbsp; &nbsp; &nbsp; : World Indicators  
> contentUrl : WorldIndicators  
> showTabs &nbsp; : true  
> size &nbsp; &nbsp; &nbsp; : 1  
> createdAt &nbsp;: 2014-05-09T23:54:50Z  
> updatedAt &nbsp;: 2015-06-07T07:25:30Z  
> project &nbsp; &nbsp;: project  
> owner &nbsp; &nbsp; &nbsp;: owner  
> tags &nbsp; &nbsp; &nbsp; :

Of individual parts of it:

> $response.tsResponse.workbooks.workbook.name  
> Sales  
> World Indicators  
> Advertising Dashboard  
> Variety

**And that wraps up this short intro…**

This should give you enough info to get started with the REST API using PowerShell. You can read more about the different methods available via the REST API through [this comprehensive reference](https://onlinehelp.tableau.com/current/api/rest_api/en-us/help.htm#REST/rest_api_ref.htm#Query_Workbooks_for_User%3FTocPath%3DAPI%2520Reference%7C _____ 55)

