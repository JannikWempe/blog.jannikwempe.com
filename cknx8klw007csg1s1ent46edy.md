# Why Svelte is different - and awesome!

*Just to get this out of the way in the beginning:*

*This is not meant to be bashing other frameworks like React, Vue, or Angular. I used all of them and React (using NextJS) is mostly still my go-to.*

# What is Svelte?

> Svelte is a radical new approach to building user interfaces. Whereas traditional frameworks like React and Vue do the bulk of their work in the browser, Svelte shifts that work into a **compile step** that happens when you build your app.

TLDR;
it's like React or Vue, but with the main difference that it is [a compiler](https://svelte.dev/blog/frameworks-without-the-framework).

There is a quote in the blog article linked above:
>Wait, this new framework has a runtime? Ugh. Thanks, I'll pass.

> – front end developers in 2018

Even though this didn't happen in 2018, I think we'll get to that mindset at some point.

## What does "Svelte is a compiler" mean?

It essentially means that Svelte-specific code gets compiled (think about transformed) to JavaScript, which is executable by the browser.

Another compiler you might know is the TypeScript compiler (`tsc`), which compiles TypeScript to JavaScript. It's the same concept.

So what is the deal? You can also write React code as `.js` and ship it. That is true, BUT that JavaScript code does not work in the browser without shipping the React runtime system as well.

> A runtime system refers to the collection of software and hardware resources that enable a software program to be executed on a computer system.

*Note: Even though I a lot of people are talking about "(no) runtime", it should more accurately be "(no) runtime **system**".*

Read the great [React as a UI Runtime](https://overreacted.io/react-as-a-ui-runtime/) blog post from [Dan Abramov](https://mobile.twitter.com/dan_abramov). It explains React being a runtime (system) in depth.

There is also another benefit besides not require a runtime. Svelte could extend and change the JavaScript syntax because the Compiler compiles the JavaScript in the end. Therefore Svelte could get rid of some limitations provided by JavaScript syntax.

This could also be a downside because if Svelte would strongly deviate from JavaScript syntax, it would essentially become another language to learn. No worries, Svelte tries to stick to the JavaScript Syntax.

# Benefits resulting from Svelte being a compiler
Since Svelte is a compiler and therefore does not require a runtime system to be loaded into the client, there are several advantages. These are what make Svelte special. The most important advantages that came to my mind are shown in the next sections.

## Performance
This one should be obvious: No runtime to load for the client results in faster load times.

The following image shows an excerpt of a JS framework benchmark (see [this GitHub repo](https://github.com/krausest/js-framework-benchmark)). It's based on a large table with randomized entries and measures the time for various operations including rendering duration.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1619355158508/Olvdi5zOk.png)

The app using Svelte ships the lowest amount of code. *(Somehow Svelte seems to require less code than vanilla JS, I have no clue how that could happen 😅)*

But it not only ships less code to the client but also executes faster:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1619355565050/accrTZHyr.png)

One of the reasons for this is that Svelte does not use a virtual DOM (vDOM). Svelte does not rely on the diff between vDOM and DOM to update the DOM. The other mentioned frameworks like React, Vue ~~and Angular~~ *(EDIT: Angular uses Incremental DOM)* do use the vDOM. You can read details about this in Svelte's blog post [Virtual DOM is pure overhead](https://svelte.dev/blog/virtual-dom-is-pure-overhead).

A quick quote from that post:
> Svelte is a compiler that knows **at build time** how things could change in your app, rather than waiting to do the work at run time.

## Svelte in a Micro-Frontend Architecture
Micro-Frontends (MFEs) is a topic in itself (read about it in [this article](https://martinfowler.com/articles/micro-frontends.html) proved by [Martin Fowler](https://twitter.com/martinfowler)). But the concept is basically that different teams can separately develop distinct parts of the frontend. Teams can also choose the tech stack they want to use. Therefore the client could end up loading different versions of Angular, Vue, React, etc.:

> Some micro frontend implementations can lead to duplication of dependencies, *increasing the number of bytes our users must download*. 
*(from Martin Fowler article linked above)*

But what about Svelte? Svelte (also using different versions of it) does not come with the downside of increasing the KBs a client has to load. 

Svelte is an awesome choice for MFE Architecture.

# Other benefits
These benefits do not result from Svelte being a compiler, but they make Svelte stand out anyways.

## REPL
Svelte has an awesome REPL. You can start playing around and try out things without any effort. This is awesome! [Try it out](https://svelte.dev/repl/hello-world?version=3.37.0).

You can also see the compiled JS and outputted CSS (it can be written in the same `.svelte` file) by clicking on "JS Output" or "CSS Output" respectively.

Is this evidence enough that Svelte is a compiler? 😉

The REPL is used in their awesome tutorial as well. You can learn Svelte hands-on: [Svelte tutorial](https://svelte.dev/tutorial/basics).

## Builtin features
Svelte has some features built-in that you need in almost any app anyways (at least larger ones), like transitions, animations, and a store. No need for an additional dependency or the decision between various choices in the first place.

> A store is simply an object with a subscribe method that allows interested parties to be notified whenever the store value changes. 

```javascript
import { writable } from 'svelte/store';

export const count = writable(0);

export const increment = () => {
	count.update(n => n + 1);
}
```

That's it. You can import `count` and `increment` across your app. Simple!

[Try out Svelte store tutorial](https://svelte.dev/tutorial/writable-stores)

Animations and transitions in Svelte are easy to use. Can you guess what the following code is doing?

```javascript
{#if visible}
	<p in:fly="{{ y: 200, duration: 2000 }}" out:fade>
		Flies in, fades out
	</p>
{/if}
```

[Try out Svelte transitions tutorial](https://svelte.dev/tutorial/in-and-out)

But they can also be used for more complex things, like the following:

![animations.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1619357232242/hS_6eOg5V.gif)

Have fun building this in React 🤪

[Try out Svelte animations tutorial](https://svelte.dev/tutorial/animate)

# SvelteKit
[SvelteKit](https://kit.svelte.dev/) is a topic of its own. But it is one of the main reasons why I am so excited. Think of SvelteKit being for Svelte what NextJS is for React.

But why is it awesome?

>SvelteKit **fully embraces the serverless paradigm**, and will launch with support for all the major serverless providers, with an 'adapter' API for targeting any platforms that we don't officially cater to.

Read about it in [What's the deal with SvelteKit?](https://svelte.dev/blog/whats-the-deal-with-sveltekit)

As I am writing this SvelteKit is currently in beta. Can't wait for the release!

# Conclusion
I could go on and on and on (have I mentioned Svelte is written in TypeScript?). But this wraps it up. You can see that I am excited, right? Svelte is something I would place my bet on. Learning Svelte and the differences to runtime system-based frameworks is definitely not a waste of time.

I am looking forward to Sveltes evolution in the future. I hope that it will soon be more widely used and I can start client projects using Svelte 😉