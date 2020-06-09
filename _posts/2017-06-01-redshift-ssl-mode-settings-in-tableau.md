---
layout: post
title: Redshift SSL Mode Settings in Tableau
date: '2017-06-01T11:40:58+10:00'
tags:
- tableau
- redshift
- aws
- amazon
tumblr_url: https://lbk.io/post/161299018338/redshift-ssl-mode-settings-in-tableau
---
Redshift has a number of different options (known as sslmode) for SSL that determine how it handles encryption. These determine whether SSL is used or not (disable, allow, prefer, require), and whether the server certificate is verified or not (verify-ca, verify-full), these options are documented in detail here [http://docs.aws.amazon.com/redshift/latest/mgmt/connecting-ssl-support.html](http://docs.aws.amazon.com/redshift/latest/mgmt/connecting-ssl-support.html)

When you configure a connection to Redshift using the ODBC admin you will see this options. When you use Tableau you will only see a checkbox for use SSL or not.

<figure data-orig-width="396" data-orig-height="361" class="tmblr-full"><img src="https://66.media.tumblr.com/dfa56cc5b46de9547dcdd59402d98a84/tumblr_inline_oquhw2WWyO1qbqa5u_540.png" data-orig-width="396" data-orig-height="361"></figure>

Selecting the Require SSL option in Tableau will put the Redshift drive into “require” mode which means the connection will only succeed if the Redshift cluster supports SSL.

But how do you configure Tableau to a stricter mode that verifies the Server certificate? Like “verify-ca” mode.

The answer lies in customizing the file that defines Tableau’s connection to Redshift, either your TDS (individual data source configurations) or TDC (global customization for a given data source type). If you right click on a data source in Tableau Desktop and select “Add to Saved Data Sources” you will be prompted to save a \*.tds file – once you have saved this file open it in a text editor.

Once you have the TDS file open you will see a section called “odbc-connect-string-extras”, within this section you will want to add a setting for the SSLMode you want e.g.: &nbsp;odbc-connect-string-extras=‘SSLMode=verify-ca’

<figure class="tmblr-full" data-orig-height="147" data-orig-width="459"><img src="https://66.media.tumblr.com/224c9dcf023197e8b9c779423a9c6ec1/tumblr_inline_oquhx9NfD61qbqa5u_540.png" data-orig-height="147" data-orig-width="459"></figure>

Once this is configured you will need to make sure you have downloaded the Redshift certificate and saved it as root.crt following the instructions form Amazon here [http://docs.aws.amazon.com/redshift/latest/mgmt/connecting-ssl-support.html](http://docs.aws.amazon.com/redshift/latest/mgmt/connecting-ssl-support.html)

If you haven’t downloaded the cert file then you will get an error when you try to connect.

<figure class="tmblr-full" data-orig-height="252" data-orig-width="556"><img src="https://66.media.tumblr.com/8c20808e09d2cca199c6b2c1d2d6e4f7/tumblr_inline_oqui04v2Ja1qbqa5u_540.png" data-orig-height="252" data-orig-width="556"></figure>

Once the cert file is in the right place you will be able to connect as expected:

<figure class="tmblr-full" data-orig-height="198" data-orig-width="624"><img src="https://66.media.tumblr.com/2333ece55fb5900673098d55726a9071/tumblr_inline_oquhxwtLeO1qbqa5u_540.png" data-orig-height="198" data-orig-width="624"></figure>
