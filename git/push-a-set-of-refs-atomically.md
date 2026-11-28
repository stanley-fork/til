# Push A Set Of Refs Atomically

Let's say I've created both a commit on `main` with some release details and an
annotated tag for the version of the release. Those are two distinct refs -- the
commit and the annotated tag. I now want to release a new version of my package.
I have a GitHub Actions CI workflow that triggers the release process whenever
an annotated tag lands. So, most of the time, a simple `git push origin main
vX.Y.Z` would push both refs, doing the trick.

There is a potentially problem though. What if conflicting changes have landed
on `main` since I last pulled. Maybe I'm working with someone else and we
haven't coordinated well. Or perhaps I merged what I thought was an unrelated PR
and forgot it would conflict.

Now when I run `git push origin main vX.Y.Z`, the tag is successfully pushed
while the latest commit to `main` is rejected. The release process kicks off
because of the new tag despite my missing release commit. Now I have a bad
release.

There are a number of procedural fixes the help with what I've described above.
One thing that will immediately address the issue at the seam of the `git push`
is to ensure the refs are pushed _atomically_.

```bash
git push --atomic origin main vX.Y.Z
```

The `--atomic` flag makes the push all or nothing.

> Use an atomic transaction on the remote side if available. Either all refs are
> updated, or on error, no refs are updated. If the server does not support
> atomic pushes the push will fail.

GitHub, GitLab, and probably most modern Git server hosts support atomic pushes.

See `man git-push` for more details.
