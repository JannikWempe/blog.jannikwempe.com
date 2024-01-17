---
title: "Different Node.js Versions and Package Managers Per Project – A Solved Problem"
seoTitle: "Different Node.js Versions & Package Managers Per Project"
datePublished: Wed Jan 17 2024 19:23:41 GMT+0000 (Coordinated Universal Time)
cuid: clri6885b000308l7hyyr0cks
slug: nodejs-and-package-managers-versions
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1705519211379/6efceea6-04fe-4e65-b0a9-886d3215dfde.png
tags: nodejs, package-manager, pnpm

---

You work on different projects, maybe in different teams, or just on older and newer personal projects. Chances are you are using different Node.js versions and different package managers or package manager versions.

**You should be able to switch between projects without fiddling around with** Node.js **and your package manager.** This post shows you my setup that has solved those issues for me. I am using MacOS, so that is what I will be writing about but the solutions should be applicable for other operating systems as well.

## Dealing With Different Node Versions

The most popular solution for using different Node.js versions on your system is probably [nvm (Node Version Manager)](https://github.com/nvm-sh/nvm). It lets you easily install new Node.js versions (e.g. `nvm install 20`, `nvm install --lts` etc.), list all available versions (`nvm ls-remote`), list your installed versions (`nvm ls`), switch to specific versions (`nvm use 20`), set a default version (`nvm alias default 20`) or switch to the Node.js versions specified by the `.nvmrc` file in the current project (`nvm use`).

I am using a (mostly) NVM-compatible alternative to NVM though: [fnm (Fast Node Manager)](https://github.com/Schniz/fnm). Why? First of all, it is faster (of course it is written in Rust 😝). But it also seems to be simpler (just compare the length of the readmes). Also, one of the main functionalities that lets you **forget about dealing with Node.js versions in your day-to-day work: auto-switching NodeJS versions** based on `.nvmrc` or `.node-version` is easier to set up. You just have to add `eval "$(fnm env --use-on-cd)"` to your `.zshrc` (other shells are also supported). With that, you automatically switch Node.js versions if you switch to a different project that has the desired version specified. **You should always have a** `.nvmrc` **or** `.node-version` **file in your Node.js projects.**

There are some other differences that are less relevant to me, like `fnm` supporting fish shell which `nvm` doesn't.

<div data-node-type="callout">
<div data-node-type="callout-emoji">ℹ</div>
<div data-node-type="callout-text">I try to install everything via Homebrew to be able to update everything with a single command. That is why I also have installed <code>fnm</code> via <code>brew install fnm</code>.</div>
</div>

I have set up an alias for `fnm` to `nvm`. With that, I can execute all of the mentioned `nvm` commands via `fnm`. That is also what I do in practice. As mentioned, `fnm` tries to be mostly compatible to `nvm` and for all of the basic commands `fnm` is just a (better) drop-in replacement for `nvm`.

### What About a System-Level Node?

I have still installed `node` via Homebrew because some other formulas or casks may depend on this. If I try to uninstall it via `brew uninstall node` I get the error `Error: Refusing to uninstall /opt/homebrew/Cellar/node/21.5.0 because it is required by [...]` . That is fine, I just leave it there but otherwise don't use it.

The system-level Node.js version is also listed in `nvm ls` as `system`, but that is not my default version, so it never gets for commands I run in the terminal.

## Dealing With Different Package Managers

I have projects that are using `npm`, `yarn` and `pnpm` and with different versions of each of those. How to deal with that? **We don't want to install them globally** via `npm install -g pnpm`. Because that would mean that the Node.js (or rather `npm` in this case) version that was activated while running the global installation now has a single version of `pnpm` installed – all projects using the same Node.js version now also have the same `pnpm` version. That is not what we want.

But what is the solution? Corepack! Corepack is a tool that comes with Node.js since [version 16.9.0 (released on Sep 07, 2021)](https://nodejs.org/en/blog/release/v16.9.0#corepack). It has also been backported to [version 14.19.0 (released on Feb 01, 2022)](https://nodejs.org/en/blog/release/v14.19.0).

> Corepack is an experimental tool to help with **managing versions of your package managers**. It exposes binary proxies for each supported package manager that, when called, will identify whatever package manager is configured for the current project, transparently install it if needed, and finally run it without requiring explicit user interactions.  
> — [Node.js docs](https://nodejs.org/api/corepack.html#corepack)

With corepack you don't have to explicitly install package managers at all. Specify the package manager and its version using the `packageManager` property in `package.json` like this: `"packageManager": "pnpm@7.27.1"` . Now you can run `pnpm` commands without installing it explicitly – corepack will do that for you. This also prevents you and others from using the wrong package manager. If you try to run `yarn` in a project that has set `packageManager` to `pnpm` you will see an error: `Usage Error: This project is configured to use pnpm`. Changing the version in `packageManager` leads to installing the new version when executing the package manager the next time (without you noticing, it just takes a little longer the first time).

<div data-node-type="callout">
<div data-node-type="callout-emoji">ℹ</div>
<div data-node-type="callout-text">At the time of writing this corepack supports <code>npm</code>, <code>yarn</code>, <code>pnpm</code> and <code>bun</code>.</div>
</div>

You can also explicitly switch the package with `corepack prepare pnpm@8.1.0 --activate`. This is only useful if you can't (or don't want to?) specify the `packageManager`in `package.json`. The `packageManager` will take precedence over explicitly activating a different version.

You have to enable corepack via `corepack enable`. This has to be done once for each Node.JS version that you are using. With fnm you can add the `--corepack-enabled` flag (or setting the `FNM_COREPACK_ENABLED` environment variable – I have set this in `.zshrc`) installing a new Node.JS version to activate corepack right after the Node.js installation.

## Summary

Dealing with different Node.js versions and different package managers and its versions is a solved problem. These measurements make it a breeze:

* use `fnm` and enable its auto-switching capabilities
    
* always specify projects' Node.JS versions either via `.nvmrc` or `.node-version`
    
* set `FNM_COREPACK_ENABLED=1` globally to automatically activate corepack for each new Node.js version
    
* always specify `packageManager` in `package.json`
    

With that in place, you never have to explicitly install Node.js or package manager versions. They are configurations in your projects and get installed automatically.