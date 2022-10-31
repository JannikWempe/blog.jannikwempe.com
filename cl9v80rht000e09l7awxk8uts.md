# My thoughts after trying to port a Shopify store from NextJS to Shopify Hydrogen

**EDIT (2022-10-31):** Just one day after publishing this post, there is a huge announcement: [Remix joins Shopify](https://remix.run/blog/remixing-shopify) and [Hydrogen v2 will be leveraging Remix](https://hydrogen.shopify.dev/roadmap#remixing-hydrogen). Also, there is a great thread on Twitter about design mistakes being made in Hydrogen v1:

%[https://twitter.com/jplhomer/status/1587095536833859584] 

* * *

A while ago I created a Shopify shop for my little brother who is selling handmade, unique furniture: [glückweiser.de](https://www.xn--glckweiser-beb.de/) (all German; currently mainly [selling on Etsy](https://www.etsy.com/de/shop/Glueckweiser)). It is written in NextJS 12 and uses Shopify's GraphQL Storefront API. The Developer Experience (DX) is okay. Everything is nice and typesafe with Typescript, graphql-codegen and react-query. BUT it is quite some manual work. Especially dealing with the cart. Initializing it, storing the id, updating etc.

I like trying out new things and this is why I thought I'll give [Shopify Hydrogen](https://hydrogen.shopify.dev/) a shot and recreate the shop using it. In this article, I want to share my experiences and thoughts after a few days of coding in Hydrogen.

## What is Shopify Hydrogen?

Before I go ahead and share my experiences, I want to give a quick intro to Hydrogen:

These are the main features as per the Hydrogen website:

![Hydrogen features](https://cdn.hashnode.com/res/hashnode/image/upload/v1667121544202/lXAOXU3fU.png align="center")

I would say its main selling point is obviously the integration with Shopify. It comes with hooks, components and types to make it really easy to interact with the Shopify API. (They also use an [XState state machine for the cart](https://github.com/Shopify/hydrogen/blob/6310974ff8f8fdcf742bc7a1f5af1370e221c6fa/packages/hydrogen/src/components/CartProvider/useCartAPIStateMachine.client.tsx).)

From an architectural perspective, it heavily relies on [React server components](https://shopify.dev/custom-storefronts/hydrogen/framework/react-server-components) and [Vite](https://vitejs.dev/). React server components allow for fetching data right in the React component. It is run on the server and streamed to the client. You can safely use secrets there since only the resulting HTML gets streamed to the client. ([React server components were just released in NextJS 13 a few days ago.](https://nextjs.org/docs/advanced-features/react-18/server-components)). React server components require quite some mind shift because previously components were primarily run on the client (neither a pro nor a con of Hydrogen for me since it's related to React).

In addition to that, Hydrogen comes with out-of-the-box support for Tailwind and Typescript.

# What I like about Hydrogen

Let us start with the pros of using Hydrogen first:

### Shopify integration

As mentioned previously, this is the main selling point in my eyes. You configure your Shopify integration in `hydrogen.config.ts` and can start using hooks like `useShopQuery` that let you fetch data from the Storefront API. The types are also shipped with Hydrogen.

Besides that, it comes with [handy components](https://shopify.dev/api/hydrogen/components) that are required for almost any shop: `Money`, `AddToCartButton`, `Image` and many more.

This nice integration lets you get started pretty quickly.

### Components provide primitives

Hydrogen components are primitives. Hydrogen doesn't provide components like `Cart` or `ProductPage`. They are different for most shops anyway. Instead, Hydrogen provides small building blocks. These can be styled and customized easily with Tailwind (and most likely CSS). They are focused on functionality rather than design (which makes sense).

### Up-to-date tech stack

I think Hydrogen was the first framework heavily betting on React server components. Now that NextJS 13 is also defaulting to server components for routes (in the `/app` folder), it is probably safe to say that server components are the future of React. Mastering Hydrogen will improve your overall React skills and will be transferable to other frameworks that are also relying on React server components.

In addition to that, Vite is also probably one of the most modern dev- and build tools out there. (At least until [Turbopack](https://turbo.build/pack) will eventually takes over 🤪)

## What I don't like about Hydrogen

Besides the pros, there are also some cons:

### Development and production environments behave differently

It is always painful if a development environment is very different from the production environment. This opens the doors for bugs in production that you were not able to catch during development.

Unfortunately, this is exactly my experience: In production, I started seeing errors that my client components could not be found. That crashed the whole app. I was able to fix this be adding `{ optimizeBoundaries: false }` to the `hydrogen` Vite plugin – but I have honestly no idea what this is doing and what other implications it might have.

Another issue: I created an API route for sending emails. I was using `nodemailer` for that. Everything works fine in dev, but I saw various errors when trying to build the app. It took me a while to find out that Hydrogen is creating edge functions for API routes. Edge functions only provide a limited feature set and therefore some node internals were not working.

I made it work by ditching `nodemailer` and using the Mailgun API with fetch directly. It worked on dev, and the bundling also succeeded but it failed after deployment (on Vercel) because the environment variables could not be found anymore (yes, I have added them to Vercel).

The DX is lacking and I was encountering multiple such issues... A huge bummer for me.

### Docs leave room for improvement

You can imagine that I was reading the docs and API references a lot. The API routes are explained in a few sentences. Potential limitations or something were not covered, edge functions or worker environments were not mentioned. I had to figure out a lot of stuff myself...

## Additional thoughts

I think Hydrogen has a high potential and I would probably default to it for creating custom Shopify shops. But the DX is not there yet. I encountered too many roadblocks that took me a while to resolve.

I would prefer to have the primitives and nice Shopify integration without having to adapt a whole new framework. I think this is exactly what [Hydrogen UI](https://shopify.dev/custom-storefronts/hydrogen/alternate-frameworks) is about (not in beta yet and I am not sure which features are Hydrogen and which are Hydrogen UI). I would really like to have Hydrogen features be easily usable in NextJS. I wonder if it would have been a better idea for Shopify to focus on something like Hydrogen UI rather than creating a full framework. All the Vite SSR stuff is the most complicated of Hydrogen for sure. Maybe that has something to do with [Shopify's Oxygen](https://shopify.dev/custom-storefronts/oxygen/getting-started) – the recommended deployment target for Hydrogen? I don't know...

Most likely Hydrogen will improve a lot in the future but I currently have a bad gut feeling about fully committing to it after my initial experiences...