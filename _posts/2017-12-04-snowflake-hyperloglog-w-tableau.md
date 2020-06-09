---
layout: post
title: Snowflake HyperLogLog w/ Tableau
date: '2017-12-04T15:39:39+11:00'
tags:
- analytics
- data
- tableau
- bigdata
- snowflake
tumblr_url: https://lbk.io/post/168173315923/snowflake-hyperloglog-w-tableau
---
Yes you can use Snowflake’s implementation of HyperLogLog aggregation with Tableau. All you need to do is leverage Tableau’s RAWSQL function.

In this case you’ll use a calc like so, we use RAWSQLAGG\_INT in this case as HyperLogLog returns a whole number:

> RAWSQLAGG\_INT(“HLL(%1)”, [Dimension Name])

In my example dataset (TPC) using HLL to approximately count the ticket number dimension across just under 3b rows:

<figure data-orig-width="456" data-orig-height="87" class="tmblr-full"><img src="https://66.media.tumblr.com/d43ebf825efe5b0b4ba2dbd1ac3b06d9/tumblr_inline_p0f5slppmz1qbqa5u_540.png" data-orig-width="456" data-orig-height="87"></figure>

As expected we get an approximation as a result (238m vs. 240m)&nbsp;

<figure data-orig-width="453" data-orig-height="112" class="tmblr-full"><img src="https://66.media.tumblr.com/5837a73f444a0ab5c52d5fbf4d926a05/tumblr_inline_p0f5tq3vuI1qbqa5u_540.png" data-orig-width="453" data-orig-height="112"></figure>

This is to be expected as HyperLogLog is an approximation for distinct count scenarios that aims to be faster over very large datasets, you can read more about it and the functions provided by Snowflake for HyperLogLog here&nbsp;[https://docs.snowflake.net/manuals/user-guide/querying-approximate-cardinality.html](https://docs.snowflake.net/manuals/user-guide/querying-approximate-cardinality.html)&nbsp;

And yes you can still use filters, other dimensions and LODs with RAWSQLAGG calcs! So things FIXED, INCLUDE, EXCLUDE are possible:

<figure class="tmblr-full" data-orig-height="90" data-orig-width="434"><img src="https://66.media.tumblr.com/cd26796d25ba904db80f02a50f0da255/tumblr_inline_p0f66hGZws1qbqa5u_540.png" data-orig-height="90" data-orig-width="434"></figure>

Works as expected:

<figure data-orig-height="52" data-orig-width="200"><img src="https://66.media.tumblr.com/2a75a77d5ce20d5937f8c1154e0421cd/tumblr_inline_p0f67tRmlU1qbqa5u_540.png" data-orig-height="52" data-orig-width="200"></figure>
