---
title: "How To Use Different Git Configs"
datePublished: Tue Sep 19 2023 07:00:12 GMT+0000 (Coordinated Universal Time)
cuid: clmpytv44000509ie0bkk8sr1
slug: git-config-includes
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1694873897921/51063ecb-2e70-4e0f-b467-ef9f76470c6a.png
tags: programming-blogs, github, programming, git

---

A lot of us are coding for an employee and privately or as a freelancer for multiple companies. You might want to associate commits for the different projects with a different user (e.g. email) or sign them differently (I think you should).

While you could [configure Git per repository](https://docs.github.com/en/get-started/getting-started-with-git/setting-your-username-in-git#setting-your-git-username-for-a-single-repository) it becomes pretty cumbersome. Instead, configuring Git for a subset of all of your local repositories is much more handy. This is what I will be going to show you in this post.

## Create Your Default, Global `.gitconfig`

First of all, we create our default, global `.gitconfig` by typing `git config --global -e` into your terminal. Let us keep this example simple and only config the `name` and `email` for our git `user`:

```plaintext
[user]
	name = Jannik Wempe
	email = jannik@wempe.dev
```

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">The default path of the global <code>.gitconfig</code> on MacOS <code>~/.gitconfig</code> (in you home folder).</div>
</div>

With that config, every one of your commits will be associated with the shown `name` and `email` – no matter where your repository is located on your machine.

## Overriding Git Configuration For Some Repositories

In order to be able to override your Git configuration for multiple repositories at once we have to group them in a folder. I use a `~/code` folder that looks like this:

```plaintext
.
├── work
└── private
```

I want to associate my commits in repositories that are located in `~/code/work` with a different email address. Let us create a separate Git configuration for that: `touch ~/.gitconfig-work`. This is what that file looks like:

```plaintext
[user]
	email = jannik@other.com
```

We now have `~/.gitconfig` and `~/.gitconfig-work` in our home directory. But how to tell Git to use a different config for all repositories within `~/code/work`?

By changing your `~/.gitconfig` to this:

```plaintext
[user]
	name = Jannik Wempe
	email = jannik@wempe.dev

[includeIf "gitdir/i:~/code/work/"]
	path = ~/.gitconfig-work
```

`gitdir/i` is a condition telling Git to include the file at the given `path` if the Git repository is located in any sub-directory of `~/code/work/` (ending the path with `/` automatically adds `/**` and thus matching all sub-directories). The `/i` part is configures the matching to be case-insensitive.

> The `include` and `includeIf` sections allow you to include config directives from another source. These sections behave identically to each other with the exception that `includeIf` sections may be ignored if their condition does not evaluate to true; see "Conditional includes" below.  
> — [Git - git-config Documentation](https://git-scm.com/docs/git-config#_includes)

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">There are other conditions like <code>onbranch</code>. Check out the <a target="_blank" rel="noopener noreferrer nofollow" href="https://git-scm.com/docs/git-config#_conditional_includes" style="pointer-events: none">documentation</a> for details.</div>
</div>

The content of `~/.gitconfig-work` **will be inserted at the exact position where it is mentioned.** That is why it will override previous configurations with the same (and why the includes should be at the end of the file). This way you could effectively append any configuration to your default `.gitconfig` – not just the `user` property.

With this config, any new commit in any repository located within `~/code/work` will be associated with the email configured in `.gitconfig-work`. Et voilá, we have overridden the Git configuration for multiple Git repositories at once.

## Conclusion

You can stitch together Git configuration files by leveraging `Include` and `IncludeIf`. The conditional `IncludeIf` allows us to conditionally overwrite certain configurations.