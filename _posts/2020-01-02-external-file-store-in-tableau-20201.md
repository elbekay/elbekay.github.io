---
layout: post
title: External File Store in Tableau 2020.1
date: '2020-01-02T13:42:51+11:00'
tags:
- tableau
tumblr_url: https://lbk.io/post/190007416788/external-file-store-in-tableau-20201
---
In the beta release of Tableau Server 2020.1 there is new feature that helps to improve RTO & RPO, and reduce storage consumption. This new feature allows you to **store your extract and workbook files external to Tableau Server on a NAS (SMB/NFS)** using a feature is called External File Store.

You can download the beta and test this feature here: [https://prerelease.tableau.com/](https://prerelease.tableau.com/)&nbsp;

If you’re new to Tableau Server the [File Store](https://help.tableau.com/current/server/en-us/server_process_filestore.htm) is responsible for storing data extracts (\*.hyper files) and workbook (\*.twb/twbx files) and their revisions. Historically the File Store has stored the files locally on the server and there hasn’t been an option to store them externally.

RTO & RPO may improve using this feature as the process for backup and restore changes when this feature is in use. Instead of packaging up all of the extract and workbook content into a backup file as in a traditional Tableau Server backup in an External File Store backup Tableau relies on the ability of the NAS to take a snapshot of the File Store during the backup process which can generally be generated in seconds. When using this feature Tableau Server will backup the repository (a PostgreSQL database containing metadata/user information/activity logs/etc.) and copy it to the external File Store. During Restore you mount one of the snapshots taking during the backup process and run a restore, during which time Tableau will read and restore the repository backup.

Storage consumption can be reduced as multiple nodes all reference the same external storage instead of duplicating the File Store content on each node which is what occurs when not using this feature.

Enabling this feature is straightforward:

1. **Mount a file share** to Tableau Server
2. Make sure the tableau user and tableau group have full access to the share
3. Run the **tsm** command to **configure external storage** : _tsm topology external-services storage enable -network-share /mnt/\<network share name\>/tableau_

This feature does not need to be enabled during install, if you enable it down the track Tableau will transfer data from the local File Store to the external one and then decommission the local File Store.

You can see it enabled on my server here, it took about 10 minutes to migrate the File Store which in my case was about 25gb of data – mostly very large extracts.

<figure data-orig-width="847" data-orig-height="498" class="tmblr-full"><img src="https://66.media.tumblr.com/c294675808e4920f00ccf97ce3ed3c36/7df18b6397c66a63-7c/s540x810/09de582220053646def52cfd4b68172321dacc99.png" alt="image" data-orig-width="847" data-orig-height="498"></figure>

During this process we can see network traffic and increase (of ~25gb) of storage utilisation occur on my NAS:

<figure data-orig-width="712" data-orig-height="259" class="tmblr-full"><img src="https://66.media.tumblr.com/b720e1751929ba9ddcce66ec0156d536/7df18b6397c66a63-7e/s540x810/4b91384b118bae74ae70f8ba53e9e0ba07a2638e.png" alt="image" data-orig-width="712" data-orig-height="259"></figure><figure data-orig-width="754" data-orig-height="552" class="tmblr-full"><img src="https://66.media.tumblr.com/f96ed0fe616bcd2a54996d4aa78579a3/7df18b6397c66a63-76/s540x810/928f4449305eeec853e8b8edebfe8f88b5d16b5e.png" alt="image" data-orig-width="754" data-orig-height="552"></figure>

One enabled the status screen will show that we are using an external File Store:

<figure data-orig-width="686" data-orig-height="835" class="tmblr-full"><img src="https://66.media.tumblr.com/4ad28fe995d0935cbbbee6c1fc846de9/7df18b6397c66a63-5d/s540x810/cfffa5a92ef54cab1c58fcd1a26faa8dee1cbe6f.png" alt="image" data-orig-width="686" data-orig-height="835"></figure>

The backup process with this feature looks like this, Tableau Server is still available for use during this process.

1. **Start back up & prepare for storage snapshot:** During this phase a repository backup is taken and transferred to the NAS, and work done to ensure a consistent snapshot. Command: tsm maintenance snapshot-backup prepare
2. **Run a snapshot on your NAS:** This would ideally be scripted but can be tested manually.
3. **Complete the backup:** During this phase the now unneeded repository backup is removed from the NAS, and services resumed that were paused during preparation. Command: tsm maintenance snapshot-backup complete

During some testing in my lab environment running a normal backup of a Tableau Server which has around 25gb of extracts and workbooks took around 23 minutes:

<figure data-orig-width="1184" data-orig-height="204" class="tmblr-full"><img src="https://66.media.tumblr.com/80d641fa8b81bc90e2b8cedb18a5fe99/7df18b6397c66a63-86/s540x810/5b7d9877c2361cb5d3189834827e86cac72092a2.png" alt="image" data-orig-width="1184" data-orig-height="204"></figure>

Whereas the snapshot backup process took around 2 minutes:

<figure data-orig-width="967" data-orig-height="335" class="tmblr-full"><img src="https://66.media.tumblr.com/83494d6328406cd3f8c4e41312b53f8f/7df18b6397c66a63-eb/s540x810/5b7150fb067d55a8bf86dd2addd1cbc0824103fe.png" alt="image" data-orig-width="967" data-orig-height="335"></figure>

Between the Prepare and Complete steps above I took a NAS snapshot which completed effectively instantly, which you can see the directory structure here under the .zfs/snapshot directory:

<figure data-orig-width="612" data-orig-height="710" class="tmblr-full"><img src="https://66.media.tumblr.com/61b4063ec3ce3473cfb0afd7a31f85cf/7df18b6397c66a63-ec/s540x810/73315b1b4d8a26b823839a3a0153f4413f2bcfd0.png" alt="image" data-orig-width="612" data-orig-height="710"></figure>

Restore is a similar process, which at a high level is:

1. **Rollback** to a snapshot on the NAS **or clone** the snapshot as a new share.
2. **Mount the share** on the Tableau Server where you are restoring.
3. Run the **restore command** : _tsm maintenance snapshot-backup restore_

During restore testing I took a fresh Tableau Server install, and restored the backup taken above. It took around 3 minutes to restore the backup, after which Tableau Server can be started and used with all the extracts and workbook content available.

<figure data-orig-width="973" data-orig-height="756" class="tmblr-full"><img src="https://66.media.tumblr.com/b5b5b0e0f3789c99e7467b81f631d627/7df18b6397c66a63-57/s540x810/6ca368f40b3df552249abc236223e2bf705ca90b.png" alt="image" data-orig-width="973" data-orig-height="756"></figure><figure data-orig-width="1005" data-orig-height="163" class="tmblr-full"><img src="https://66.media.tumblr.com/ee61ff9fbc225db14a74a075e4a54d73/7df18b6397c66a63-70/s540x810/e529184d27699de8c2dc897128534c34279221ac.png" alt="image" data-orig-width="1005" data-orig-height="163"></figure>

After the restore above we can see the entire 25gb of content is available:

<figure data-orig-width="554" data-orig-height="192" class="tmblr-full"><img src="https://66.media.tumblr.com/b559e98056a26d18e29c57305b9d76d6/7df18b6397c66a63-6d/s540x810/1d0aebf7f1c963c816759aff5620c78a29dfd8b6.png" alt="image" data-orig-width="554" data-orig-height="192"></figure>

As you can see even in a lab environment we saw impressive (23 min to 2 min) improvements in backup and restore time for my test Tableau Server; additionally this feature makes it quicker than ever to spin off a clone of a backup and mount it to a new server for non-prod testing purposes like new version upgrades.

As a finishing note & off-topic from Tableau but good backup practice (e.g. the 3-2-1 rule) would also dictate that you don’t rely on only one backup copy/medium (the NAS snapshot in this case) for protection, so you should consider a backup the data in the snapshot to another backup medium and offsite as part of your strategy and depending on the importance of the data.

–

_ **Note:** &nbsp;testing above is in my personal lab testing environment using NVMe SSD storage as local storage for Tableau Server and FreeNAS backed by NVMe SSD as my NAS device. Speed ups for backup and restore will differ between different environments however simply knowing that it is no longer necessary to package up all extract and workbook data with External File Store should mean meaningful improvements in backup performance._

