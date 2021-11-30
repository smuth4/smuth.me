+++
tags = ["gluster"]
date = "2020-08-02"
title = "What are those zero byte files on my glusterfs bricks?"
+++

While troubleshooting some issues on a distributed GlusterFS cluster, I came across the presence of a bunch of odd zero-byte files with empty permissions on the bricks.

```
132504 1 ---------T 2 1000 1000 0 Nov 28 02:09 /data/gluster/tank/brick-xxxxxxxx/brick/example/file.txt
```

I had known they were there previously, but since the bricks were composed of ZFS zpools, there was very little danger of running out of inodes or anything, so I had let them be. This time however, they seemed to be related to the issue at hand, and after some digging, I found they were being [used to maintain hard link information](https://gluster-users.gluster.narkive.com/FNNOr7Ru/files-losing-permissions#post5).

The mailing list has strict instructions to not meddle with them, but in my particular case they ended up being the root of the problem. At some point in the past, it seems the cluster was in a partially available state, and some files had gotten into a state where they existing on multiple bricks (which never happens under normal cirumstances). During a later cleanup effort, they had been replaced with hardlinks to identical files. This caused the very unusual symptom of gluster reporting client-side the existence of multiple files with the same exacts paths, one of which was the actual file, the others being the zero-byte files at hand.

Interestingly, most programs dealt with this reasonably well. Since the files still had different inodes, anything that scanned the effected parent directories discarded the bad files as unreadable and continued on. It was only when programs attempted to open the files directly by the path that things went haywire. Generally they'd get a handle to the good path, but under certain circumanstances they'd get the bad one. It was possible to coax the client cache into recognizing the good path again, but ultimately I ended up deleting the bad files directly from the bricks in order to resolve the problem.
