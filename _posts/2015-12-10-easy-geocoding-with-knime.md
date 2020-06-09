---
layout: post
title: Easy geocoding with KNIME
date: '2015-12-10T14:15:46+11:00'
tags:
- tableau geocoding KNIME
tumblr_url: https://lbk.io/post/134895946868/easy-geocoding-with-knime
---
Sometimes you have a list of addresses that you’d like to visualise in Tableau (or another tool) but don’t have the requisite latitude and longitude co-ordinates.

If you’re addresses are in a standard format, for example like&nbsp;_85 High Street Northcote 3070_ you can look at using something like KNIME to help you generate lat/lon co-ords very quickly and easily. KNIME is an open source data integration, transformation, extraction, analytics, mining etc. platform where processes are designed with visual workflows.

What you’ll need:

1. KNIME - download it here ([https://www.knime.org/downloads/overview](https://www.knime.org/downloads/overview)), the standard installer (without all extensions) is fine.  
2. A list of addresses in a fairly standard format, think of the type of format you’d expect to be able to enter into Google maps and get a result. This can come from a DB or text file. You can find a sample dataset at the following link if you want to follow along:&nbsp;[http://pastebin.com/1ZMaht9S](http://pastebin.com/1ZMaht9S)  

## Step 1:

Run the KNIME installer.

Open KNIME.

## Step 2:

Install the Palladian extensions, we’ll use these for geocoding.

File \> Install KNIME Extensions \> Other \> Palladian for KNIME

<figure data-orig-width="945" data-orig-height="344" class="tmblr-full"><img src="https://66.media.tumblr.com/36daaa1895565a275db47d1744fb2a9a/tumblr_inline_nz4gupCEm01qbqa5u_540.jpg" alt="image" data-orig-width="945" data-orig-height="344"></figure>
## Step 3:

Build a workflow to read your database or file containing the addresses, process (geocode) the addresses and output the results.

In this example we will build a basic workflow where we read a file, generate some co-ordinates, manipulate some resulting columns, and create a file. There are plenty of articles you can find through Google to learn how to read/write from your DB with KNIME.

<figure data-orig-width="835" data-orig-height="196" class="tmblr-full"><img src="https://66.media.tumblr.com/63935d83822320c62aec417f5f52aba6/tumblr_inline_nz4gv7BmRH1qbqa5u_540.jpg" alt="image" data-orig-width="835" data-orig-height="196"></figure>

Create a new Workflow (CTRL-N) and give it a name.

<figure data-orig-width="695" data-orig-height="217" class="tmblr-full"><img src="https://66.media.tumblr.com/446302c154600e182f3aad3c4a6aaa60/tumblr_inline_nz4gvyU74k1qbqa5u_540.jpg" alt="image" data-orig-width="695" data-orig-height="217"></figure>

Bring in the File Reader node (drag it into the blank workspace), and configure (double click the node) it to read in your CSV file (if using a database, then this node will be a database reader node).

<figure data-orig-width="443" data-orig-height="110" class="tmblr-full"><img src="https://66.media.tumblr.com/5dd5d9a09a2e37fe1328d21bdd32a14c/tumblr_inline_nz4gvkpmYY1qbqa5u_540.jpg" alt="image" data-orig-width="443" data-orig-height="110"></figure><figure data-orig-width="591" data-orig-height="308" class="tmblr-full"><img src="https://66.media.tumblr.com/d1a548f9162d02b0aad7dc922eb488f8/tumblr_inline_nz4gwiFvTx1qbqa5u_540.jpg" alt="image" data-orig-width="591" data-orig-height="308"></figure>

Add a GoogleAddressGeocoder node. Configure it to use your column containing the address. Join the nodes by dragging between the small triangles on the edges of each node, these represent the input/output&nbsp;‘ports’ for each node.

<figure data-orig-width="416" data-orig-height="138" class="tmblr-full"><img src="https://66.media.tumblr.com/f2d5614a00cd6b65e5900e778561bc40/tumblr_inline_nz4gxfdPmS1qbqa5u_540.jpg" alt="image" data-orig-width="416" data-orig-height="138"></figure>

(Optional) Add a Cell Splitter Node. We will use a _space_ as our delimiter, this will split the lat/lon from one column into two, this is optional and could easily just be done in Tableau (with split).

<figure data-orig-width="548" data-orig-height="238" class="tmblr-full"><img src="https://66.media.tumblr.com/d06639c28fc4ec9b54591865d62df977/tumblr_inline_nz4gxnQmjo1qbqa5u_540.jpg" alt="image" data-orig-width="548" data-orig-height="238"></figure>

(Optional) Add a Column Rename node, and rename the two columns that were split from the CellSplitterNode, again this is optional and you can rename the fields in Tableau instead if you like.

<figure data-orig-width="615" data-orig-height="210" class="tmblr-full"><img src="https://66.media.tumblr.com/ef521927046980a82c29414b8206115a/tumblr_inline_nz4h18zLRU1qbqa5u_540.jpg" alt="image" data-orig-width="615" data-orig-height="210"></figure>

(Optional) Add a Column Filter node and remove the coordinate column, this is also optional and is just me cleaning the data.

<figure data-orig-width="767" data-orig-height="327" class="tmblr-full"><img src="https://66.media.tumblr.com/c0d8d7a34485ba666a8c65de4f35d14a/tumblr_inline_nz4gxwK04T1qbqa5u_540.jpg" alt="image" data-orig-width="767" data-orig-height="327"></figure>

Add a CSV Writer node (or other DB / file writer node) and select and output location and options.

## Step 4:

Run the workflow with Shift-F7, KNIME will read your files/DB addresses, pass them to the geocoder and write an output file/DB for you.

## Step 5:

Bring your newly created file/DB connection to Tableau and visualise the results.

<figure data-orig-width="668" data-orig-height="401" class="tmblr-full"><img src="https://66.media.tumblr.com/e16f75dd95a8e014d1eb984e3fe6d400/tumblr_inline_nz4gubdSaP1qbqa5u_540.jpg" alt="image" data-orig-width="668" data-orig-height="401"></figure>
