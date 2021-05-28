## Debunking Tailwind Counterarguments

# Same Discussions Over and Over Again

This post debunks some of the arguments against using `tailwindcss`. I've argued about them quite often in the past. Maybe you have, too? I'll plan to just reference this article from now on. This will make my (and your?) life easier 🤪

If you are in the 'against Tailwind camp', I'm interested in valid counterarguments from your perspective. At the end of this article, I will point to a con I currently think of as being somehow valid.

*NOTE: Even though this post is about Tailwind the written also applies for utility first CSS frameworks in general.*

# The Counterarguments
These are just some statements I heard over and over again over time. I'll go through them one by one after this section. They are ordered in descending order after the emotions coming up on my side 😅

> Tailwind violates the separation of concerns between style and markup.

---

> Tailwind is like inline styles and they are considered bad practice.

---

> Tailwind is huge and reduces performance. You have to set up `PurgeCSS` `PostCSS` plugin to make it smaller.

---

>  There is a lot I have to learn upfront.

---

> HTML looks like a mess and is bloated.

# Debunking the Arguments
Now to the fun part. How can be argued against these arguments?

## Violates Separation of Concerns
> Tailwind violates the separation of concerns between style and markup.

The concerns being referred to are the markup (HTML) and the styling (CSS). People pointing to this often like to refer to [CSS Zen Garden](http://www.csszengarden.com/), which showcases that the look and feel of a website can be changed entirely by only changing the CSS. Yes, that is true, it is possible. How often have you changed the look and feel of your entire site without touching HTML (not talking about dark mode 😉)?

You think with something like [BEM (Block Element Modifier)](http://getbem.com/) you have "seperated the concerns"? A card using BEM could look something like this:

```html
<div class="card card--dark">
    <img class="card__image">
    <h3 class="card__heading">Heading</h3>
    <div class="card__content">
        <p class="card__text">Lorem Ipsum...</p>
    </div>
</div>
```

I hope you don't want to tell me the concerns are separated now. The markup knows about being displayed as a card. You want something looking similar, but it isn't really a card? Maybe you are starting extending `.card` somehow or find a broader, more general name.

What benefits do you get by trying to completely separate the styling from the markup? From my perspective, it even is a strength of Tailwind to keep them together! Just imagine you (or somebody else) will remove the card in a few months or create different styling for it. Will you remember to also delete the CSS? Can you delete it? Another scenario: You have a big (React) component and want to cut it into smaller pieces. Which of the CSS can (or have to be) taken to the newly created component? `Strg + F` will probably be your friend.

**Keeping styling and markup together improves maintainability.**

Another benefit of not separating the styles from the markup: **You don't have to come up with names for things.** You want to change the card to be displayed as something else? You will probably change the name... It may seem like a minor thing to you, but for me, it is actually one thing less to worry about.

You probably have read or heard the following quote before:
> There are only two hard things in Computer Science: cache invalidation and naming things.

[Read this](https://martinfowler.com/bliki/TwoHardThings.html) if you have never heard about it.

There is an in-depth article with the title [CSS Utility Classes and "Separation of Concerns"](https://adamwathan.me/css-utility-classes-and-separation-of-concerns/) from  [Adam Wathan](https://twitter.com/adamwathan). Read it if you are not convinced yet. *It may be a little biased because Adam is the creator of Tailwind 🤪*

## Tailwind is Like Writing Inline Styles
> Tailwind is like inline styles and they are considered bad practice.

Sorry, but you didn't get the point of Tailwind. Most probably you haven't used it all. *(How are you able to argue against something you haven't really tried yourself?)*

Okay, got it, there are quite a lot of 1:1 mappings between CSS properties and Tailwind classes like `display: flex` and the class `flex`. But, first of all - just being a side note - this is not true for all cases, e.g. `space-y-1` (which is one of my favorites) translates into 
```css
/* CSS for Tailwinds .space-y-1 */
--tw-space-y-reverse: 0;
margin-top: calc(0.25rem * calc(1 - var(--tw-space-y-reverse)));
margin-bottom: calc(0.25rem * var(--tw-space-y-reverse));
```

Way more important: Tailwind is not about the translation of CSS properties into classes. It enables you to easily use (and stick to) a *design system by providing a limited set of classes* [through the theming](https://tailwindcss.com/docs/theme), which helps you to create a consistent styling more easily. `style="color: #6366F1;"` and `class="text-indigo-500"` provide the same outcome in this case, but by using inline styles you could just use a slightly off color in the next place whereas with Tailwind you would use something like `class="text-indigo-600"`. Yes, you could achieve something like this with CSS variables yourself, but Tailwind already provides that to you out of the box.

## Tailwind Is Bad For Performance; PurgeCSS Has to Be Set Up
> Tailwind is huge and reduces performance. You have to set up `PurgeCSS` `PostCSS` plugin to make it smaller.

You are at least somehow right. Tailwind is huge without using `PostCSS`s `PurgeCSS` plugin in order to remove unused classes ([about 3.5MB uncompressed](https://tailwindcss.com/docs/optimizing-for-production)). But you don't have to set up `PurgeCSS` and integrate it into your build step, pipeline, or whatever you use because since v1.4 this is built-in. Just provide the `purge` option in `tailwind.config.js` and you are good to go.

The end result will probably be even smaller (and therefore more performant) compared to your own CSS. Chances are, when writing your own CSS, that you ship at least some unused CSS (remember the argument about separation of concerns). Besides that, you have a lot of duplicated CSS in different places instead of reusing predefined classes *(I haven't measured it, but it makes sense to me. Anyone has numbers?)*.

⚠️ A valid pain point regarding purging is the fact that you have to be aware of how purging works. You have to write [purgable HTML](https://tailwindcss.com/docs/optimizing-for-production#writing-purgeable-html). You can't use a class like `text-$(color)-500`, because `PurgeCSS` just works by parsing your HTML as a string. It does not run your code or interpret it in some way.

## A Lot to Learn
>  There is a lot I have to learn upfront.

There is an upfront learning curve. But it is way flatter than you may think. The classes are very natural and consistently named. You only need to know a subset and chances are you can derive classes from your existing knowledge. There are exceptions to this: I had to look up the classes for `line-height` quite often (e.g. `.leading-loose`). 

It is way harder to remember the classes of CSS frameworks like Bootstrap and strangely enough, I've never heard that as an argument against it 🤷🏼‍♂️

## HTML is a Mess
> HTML looks like a mess and is bloated.

Okay, there are no arguments to fully debunk this statement. This is copied from one of my projects:

```html
<article className="overflow-hidden rounded-lg shadow-md cursor-pointer duration-300 ease-in-out hover:scale-102">
    <section className="h-full p-3 rounded-lg shadow-lg bg-gray-50">
        <h3 className="mt-1 text-xl font-semibold truncate">Title</h3>
        <footer className="flex items-center mt-2 space-x-2">
            <!-- ... -->
        </footer>
    </section>
</article>
```

The HTML undoubtedly is more cluttered 😕

BUT on the other side, you don't have to switch to other files to see/adjust the styling. And how much of a deal is it with syntax highlighting? You will still be able to have an overview of the HTML elements due to a different color. In the end, this is something you will get used to.

# Valid Cons
Even though you probably by now have noticed that I am somehow a fanboy, I am also a quite skeptical and analytical person in general. There are seldom things without any tradeoffs. It is always about trade-offs.

Please let me know the counterarguments I haven't mentioned.

I also have one myself: Compared to component libraries like [Chakra UI](https://chakra-ui.com/) (which I absolutely love - it has a very Tailwind-like API), you have to write things from scratch. Common building blocks like buttons, date pickers, modals, and inputs are hard to get right (especially considering a11y). So Tailwind initially has a burden to get up to speed. BUT. Firstly, you can reuse what you have written before and therefore it gets better. And secondly, this is where [Tailwind UI](https://tailwindui.com/) and [Headless UI](https://headlessui.dev/) come into play. They provide you with easily adjustable building blocks.

# Conclusion
I love Tailwind ❤️ There are reasons why it is getting very popular. You should try it out if you haven't already. And if you still have concerns after a while you are in a position to argue against it. (Again: Don't argue against things without at least having tried it out.)

**Happy to hear your feedback. Are you using Tailwind? Not? Why not?**



