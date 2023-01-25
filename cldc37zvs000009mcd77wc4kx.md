# You Have Fucked Up! How to git revert?

You have messed up production. All hell broke loose. What to do now? Fix it as fast as possible and undo the last change that made everything fall apart to unblock further deployments.

## Fix Production Fast

First of all, it is a good idea to get back to a working version of the application as fast as possible. Many hosting platforms have some sort of rollback functionality ([like Vercel's instant rollback](https://vercel.com/docs/concepts/deployments/instant-rollback)) that you can use to get back to a working application in seconds. Or you deploy the last working commit from your local machine (be careful to not make things worse; you can leverage [git bisect](https://git-scm.com/docs/git-bisect) to find the last working commit). This is only a temporary solution. The HEAD of your main branch will still point to the broken application version. So how to fix that?

## Undo Your Fuckup – Git Revert

This section is about getting your main branch (or trunk or whatever you want to call it) back to a functional state to unblock further deployments (and to prevent yourself from accidentally deploying the broken application again using your CI/CD pipeline).

### Remove Last Commit (not `git revert`)

You could just remove your last commit using `git reset HEAD^ --hard` and `git push --force` and get rid of that commit (assuming that this has introduced the error). **This is rarely a good idea.** You don't want to rewrite the git history of a shared remote branch. (Even if you are alone in this project I'd prefer the next solution). It can introduce conflicts with your co-workers' local working version of that branch. But how to do better?

![removing commit SHA1 by using git reset and force push](https://cdn.hashnode.com/res/hashnode/image/upload/v1674669030942/108711f3-5a33-495a-b628-94803e065465.png align="center")

**Here is where** `git revert` **comes into place.**

### Git Revert (undo) the Last Commit

> `git revert` is used to record some new commits to **reverse the effect of some earlier commits** (often only a faulty one).

*(Source:* [*Git revert documentation*](https://git-scm.com/docs/git-revert)*)*

This is what we want. We create a new commit (not changing the existing commits and messing with our co-workers' sanity) that applies the changes that the commit to be reverted introduced. We can revert the changes of the last commit by finding its SHA using `git log` and run `git revert SHA`:

![reverting a single commit using git log and git revert](https://cdn.hashnode.com/res/hashnode/image/upload/v1674667816083/3809b465-a5d3-45f3-b4b0-dfe60ab666e3.png align="center")

Commit `SHA3` now applies the changes that are required to get back from `SHA2` to `SHA1`. Problem solved and we are good to go, right? Well, in this simple case, yes. But often we work with Pull Requests (PRs) and create merge commits introducing our (shitty) changes to the main branch. Trying the above will result in this error:

```bash
git revert SHA2
# error: commit SHA2 is a merge but no -m option was given.
# fatal: revert failed
```

`git revert` fails and you start to sweat. Let's explore what this error is about and how to fix it.

### Git Revert a Merge Commit

The part "but no -m option was given" of the previous error is already hinting at the solution. What is the `-m` option of `git revert`?

> `-m parent-number`
> 
> `--mainline parent-number`
> 
> Usually you cannot revert a merge because you do not know **which side of the merge should be considered the mainline**. This option specifies the parent number (starting from 1) of the mainline and allows revert to reverse the change relative to the specified parent.

(Source: [Git revert documentation](https://git-scm.com/docs/git-revert#_options))

Okay, next question: What does "which side of the merge should be considered the mainline" mean?

Let's have a look at this scenario:

![two branches and a merge commit SHA3 that is based on SHA1 (main branch) and SHA2 (branch A)](https://cdn.hashnode.com/res/hashnode/image/upload/v1674668527499/4ae01ecd-3c31-489d-88c5-ef82837f186a.png align="center")

As you can see, `SHA3` which introduced the issues has **two parent branches** (`SHA1` and `SHA2`). How should git know which one it should revert to (the mainline)? This is what you have to tell `git revert` with the `-m` option.

How to find the "parent number" that you have to pass to `git revert -m`? You can use `git catfile -p SHA3` for that or look at the commit message of the merge commit using `git log`:

![git log and git cat-file commands in action](https://cdn.hashnode.com/res/hashnode/image/upload/v1674669278777/d7c0269e-7fc1-4d44-b767-e400a0f187e7.png align="center")

`git log` tells you that `SHA3` is a merge commit of `SHA1` and `SHA2`. `git cat-file` shows you details for commit `SHA3` and tells you that `SHA3` has two parents (basically the same info as in the commit message). In both cases the first mentioned SHA is `parent-number` 1 (the second 2 etc.). In order to revert the changes introduced in commit `SHA3` and get back to `SHA1` you have to execute `git revert -m 1 SHA3`. Now you are all set and can push the branch with the new commit.

![adding a new commit SHA4 that reverts SHA3 back to the state of SHA1](https://cdn.hashnode.com/res/hashnode/image/upload/v1674668889646/27d39782-d101-4220-a766-dbaed6359247.png align="center")

Now you know to revert merge commits and understand what you are doing instead of copying the command from Stackoverflow over and over again 😜

## Conclusion

It can become stressful if you are in the situation of having to fix a production problem that you (or a co-worker) have introduced. Things get really bad if you face a "fatal" error trying to revert a merge commit. Make sure you can deal with these situations before you have to deal with them.

> The time to repair a roof is when the sun is shining.
> 
> \- OR -
> 
> The time to learn how to deal with a production issue is when production is not broken.