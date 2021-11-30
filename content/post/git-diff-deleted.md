+++
tags = ["terraform", "powerdns"]
date = "2019-08-05"
title = "Finding the log for deleted files in git"
+++

Recently I came across a blog post explaining how to get the history of a deleted file: https://dzone.com/articles/git-getting-history-deleted, which pointed to yet another blog with the simple solution https://feeding.cloud.geek.nz/posts/querying-deleted-content-in-git/:

```
git log -- deleted_file.txt
```

However, neither explained exactly why this works, and I got interested. `--` is normally the convention that means "pass everything after this into the program as a literal string. For example, if for some crazy reason you had a file named `-h` (which is perfectly legal), `rm -h` would just bring up the man page, while `rm -- -h` would work as expected (as would `rm ./-h`). However, this is just a convention and git might be free to ignore it.

As the man page actually points out, git does indeed treat these differently:

>       [--] <path>...
>           Show only commits that are enough to explain how the files that match the specified paths came to be. See
>           History Simplification below for details and other simplification modes.
>
>           Paths may need to be prefixed with -- to separate them from options or the revision range, when confusion
>           arises.

The short story, according to the [History Simplification section](https://www.git-scm.com/docs/git-log#_history_simplification), appears to be that the default behavior is able to accurately track changes across moves/renames, but they also needed a way to say "track this exact path only".
