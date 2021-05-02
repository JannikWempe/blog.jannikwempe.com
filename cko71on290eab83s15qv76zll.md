## React Pre-rendering and Potential Hydration Issue

# Why I did my research
I am not writing this article because the subject just came to me. It all started with an issue I had. While developing an eCommerce website for my little brother using Next.js this popped up in Google Chrome dev console:

```
Warning: Expected server HTML to contain a matching <div> in <a>.
react-dom.development.js?61bb:67
```

I scratched my had and did what a developer does: ask Google.

This is why I write this article about React, SSR and Hydration. It should help others understanding (and my future self) the issue.

# The issue and first step to find the cause

This is what I saw when refreshing the page with items in cart:

![issue-showcase.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1619949454209/QPOXcH1YS.gif)

The stack trace points to `CartNavigationIcon.tsx`. Here it is *(omitted styles)*:

```typescript
// CartNavigationIcon.tsx
import { ReactElement } from 'react';

import { Link } from '@components/elements/Link/Link';
import { ShoppingCartIcon } from '@heroicons/react/outline';
import { Selectors, useCart } from '@hooks/use-cart';

export const CartNavigationIcon = (): ReactElement => {
  // items are saved and initially loaded from localStorage
  const cartItems = useCart(Selectors.cartItems);
  const numOfItemsInCart = cartItems.length;

  return (
    <Link href="/cart">
      {numOfItemsInCart > 0 && <div>{numOfItemsInCart}</div>}
      <ShoppingCartIcon />
    </Link>
  );
};
```

Can you spot the issue? Great! I couldn't...

Google told me it is related to pre-rendering and `ReactDOM`s hydration (also reffered to as rehydration). I digged deeper and tried to understand what was happening...

# Understanding Pre-rendering and Hydration
Probably you heard about terms like SSR, SSG and maybe also hydration. But do you really got the concept behind them - especially the hydration? I certainly didn't...

## Pre-rendering
Next.js supports [two forms of Pre-rendering](https://nextjs.org/docs/basic-features/pages#two-forms-of-pre-rendering): **Static generation** *(the SG in SSG, which stands for Static Side Generator)* and  **Server-Side Rendering** (SSR).

The **main difference between** these two: point in time when the HTML markup is  generated: 

* SG -> **at build time**
* SSR -> on the fly **at request time**
   
*(Note: SSR can be used with caching in order to not generate it every time.)*

But both of them have one important thing in common: **both serve pre-rendered HTML to the client**. This is why both of them are referred to as "Pre-rendering". *(This is what differentiates them from Client Side Rendering (CSR), where the page loads with something like `<div id='root'><div>`, which acts as the container for React rendering.)*

You can read more about the details, comparisons between  SSG, SSR and CSR and their performance implications in the great Google article [Rendering on the Web](https://developers.google.com/web/updates/2019/02/rendering-on-the-web). 

*(I was using SG when the error appeared, but the same applies for SSR.)*

## Hydration
Ok, fine, but what about Hydration?

There is a method called `ReactDOM.hydrate()` and this is how it's described in [Reacts docs](https://reactjs.org/docs/react-dom.html):
> Same as render(), but is used to hydrate a container whose HTML contents were rendered by ReactDOMServer. React will attempt to **attach event listeners to the existing markup**.

Great, but what is `ReactDOMServer`?
> The ReactDOMServer object enables you to render components to static markup. Typically, it’s used on a Node server.

`ReactDOMServer` methods are used for pre-rendering.

**TLDR;** Hydration makes the pre-rendered HTML interactive in the client.

But you should not stop there in the React docs, because the paragraph after the intro to `ReactDOM.hydrate()` explains the cause of my issue: 

>React expects that the rendered content is **identical between the server and the client**. It can patch up differences in text content, but you should treat mismatches as bugs and fix them. [...] There are **no guarantees that attribute differences will be patched up** in case of mismatches. This is important for performance reasons because in most apps, mismatches are rare, and so validating all markup would be prohibitively expensive.

💡 React expects that the rendered content is **identical between the server and the client**.

💡 There are **no guarantees that attribute differences will be patched up** in case of mismatches.

Hydration is done because of performance reasons. With hydration React does not have to render the whole page again in order to make it interactive.

# The cause of my problem
Take another look at my problem presented above. Do you spot the issue now?

When rendering the page during the build step there are no items in the cart. They are stored in the users browser and are not available during the build step. Therefore the server renders HTML with an empty cart. But that is not the case on the client side. The cart might have items there. The content of the pre-rendered HTML and the HTML in the client can therefore be different.

As we learned in the last section, this could lead to the UI not being updated. The cart icon could therefore show an empty cart *(it didn't in my case)*. The error in the console points us to this, because this can end up as a bug.

# How to solve the issue?

Quick answer: Make the pre-rendered content and the content in the client the same 🤷🏼‍♂️

But how?
The solution I applied, was to display the actual amount of items in the cart **after the first render** and start with no items initially:

  ```typescript
// CartNavigationIcon.tsx
import { ReactElement } from 'react';

import { Link } from '@components/elements/Link/Link';
import { ShoppingCartIcon } from '@heroicons/react/outline';
import { Selectors, useCart } from '@hooks/use-cart';
import { useMounted } from '@hooks/use-mounted';

export const CartNavigationIcon = (): ReactElement => {
  const { hasMounted } = useMounted();
  const cartItems = useCart(Selectors.cartItems);

  // ✅ ensure first render on server and client is the same
  const numOfItemsInCart = hasMounted ? cartItems.length : 0;

  return (
    <Link href="/cart">
      <div className={numOfItemsInCart === 0 ? 'hidden' : 'block'}>
        {numOfItemsInCart}
      </div>
      <ShoppingCartIcon />
    </Link>
  );
};

// use-mounted.tsx
import { useEffect, useState } from 'react';

export const useMounted = (): { hasMounted: boolean } => {
  const [hasMounted, setHasMounted] = useState(false);

  useEffect(() => {
    setHasMounted(true);
  }, []);

  return { hasMounted };
};
```

`useMounted()` is a simple hook providing us the information about the component being rendered. That way we can set the actual amount of items in cart after the first render and end up with the server content initially being the same than the client conent.

Please note that this solution will make the `CartNavigationIcon` render twice. Do not overuse this for to many and / or complex components and apply it as close to the root cause as possible. Otherwise child components will unnecessarily also be rerendered.

# Conclusion

I hope this article helps to clarify the concept of pre-rendering and hydration. I think the `ReactDOM` error message isn't to helpful to point the developer into the right direction. But maybe this article helps to not only resolve the problem but also understanding the root cause of it.

# Additional sources

* [Googles Rendering On The Web](https://developers.google.com/web/updates/2019/02/rendering-on-the-web)
* [@indigitalcolor](https://twitter.com/indigitalcolor)s article [Keeping Server-Side Rendering Cool With React Hydration](https://aboutmonica.com/blog/server-side-rendering-react-hydration-best-practices)
* [@joshwcomeau](https://twitter.com/joshwcomeau)s article [The Perils of Rehydration](https://www.joshwcomeau.com/react/the-perils-of-rehydration/)
* [React docs](https://reactjs.org/docs/react-dom.html#hydrate)