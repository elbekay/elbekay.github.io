---
layout: post
title: Multiple Term Text Search in Tableau
date: '2019-10-31T20:47:10+11:00'
tags: []
tumblr_url: https://lbk.io/post/188718471333/multiple-term-text-search-in-tableau
---
Posting this small entry in the hope that it helps some people looking to add a little more flexibility into their text search in Tableau.

I’m going to address 2 scenarios OR and AND searches.

OR being examples like find text that contains “metallic OR yellow”.  
AND being examples like find text that contains “global AND black AND chair”

If you want to see how it works, or download the workbook download it here: [https://public.tableau.com/profile/elbekay#!/vizhome/TextMatching/OrNotOr](https://public.tableau.com/profile/elbekay#!/vizhome/TextMatching/OrNotOr)

Essentially we’re looking to put search boxes in that allow users to input multiple terms and match (using regex) in ways beyond what you can do with Tableau’s wildcard match.&nbsp;

<figure class="tmblr-full" data-orig-height="642" data-orig-width="518"><img src="https://66.media.tumblr.com/d7f6777fa83a339e7a183b514db224df/baae401ac314c1e7-2d/s540x810/fd672a6a6f30a60833a88c8f51cc18a380ca0bd5.png" data-orig-height="642" data-orig-width="518"></figure><figure class="tmblr-full" data-orig-height="644" data-orig-width="707"><img src="https://66.media.tumblr.com/7c11b1734883a2e939bdb180644f2ece/baae401ac314c1e7-43/s540x810/21cf272c94d470c99eb884b5defed3dd71f0f233.png" data-orig-height="644" data-orig-width="707"></figure>

The way we will create both search filters is using regex match functions in Tableau. Which takes the form of.

> REGEXP\_MATCH(  
> &nbsp; &nbsp;[Text to Search], // in the example linked above we search [Product Name]  
> &nbsp; &nbsp;[Search Pattern or Terms]  
> )

We’ll simplify the example here so we’ll use a comma (,) to separate search terms, for example Yellow,Binney to search for text containing either of those terms.

To do this you’ll need a **parameter**. To create one right click in the Parameters section in the bottom left of Tableau Desktop and select **“Create Parameter”** then select **String as the Data Type** , give it a **name like “OR Search”** and press Ok.

Next **right click on your Parameter** and select **Create \> Calculated Field**. Call this calc your **OR Pattern**

The calculation is where we set up our Search Pattern to work as an “Or”, the calculation you can copy-paste is below. Replace the parameter name with the name of the parameter that you created.

> “\b(” + REGEXP\_REPLACE([Parameters].[OR], ’,’, “|”) + “)\b”

This will create a regex pattern similar to \b(Yellow|Binney)\b and you can see how it matches at this regex tester here [https://regex101.com/r/OoE4r5/1](https://regex101.com/r/OoE4r5/1)

This will match **full words only** , if you want a **partial word match** remove the \b references in the regex calc.

If you want to make it **case insensitive** then wrap the REGEXP\_REPLACE and REGEXP\_MATCH functions in LOWER()

Finally right click on the field that you want to search and select **Create \> Calculated Field**

> REGEXP\_MATCH(  
> &nbsp; &nbsp;[Field you want to search], &nbsp;  
> &nbsp; &nbsp;[Your Or Pattern calc above]  
> )

Congratulations the OR pattern is now done. If you want to make a NOT OR type filter just put a NOT in front of REGEXP\_MATCH above.

Creating an **AND filter** is the same as the above, except your AND Pattern will be:&nbsp;

> “(?=.\*\b” + REPLACE([Parameters].[AND], ’ , ’, “\b)(?=.\*\b”) + “\b).\*”

It will generate a regex pattern like (?=.\*\bChair\b)(?=.\*\bGlobal\b) which will match text where Chair AND Global appear, it is an example of a positive lookahead in regex, see [https://www.regular-expressions.info/lookaround.html](https://www.regular-expressions.info/lookaround.html)

