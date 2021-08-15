## Benefits and Downsides of Using Statecharts

*This post is part two of my [series about state machines](https://blog.jannikwempe.com/series/state-machines). It is a follow-up post of [State Machines and Statecharts: What are they?](https://blog.jannikwempe.com/what-are-state-machines-and-statecharts). Start there if you'd like to learn about what (finite) state machines and statecharts are.*

---

After having answered the *"What are State Machines and Statecharts?"* in my last article, this will be about the *"Why are they?"*. What are their benefits? What are the downsides?

# What are the Benefits of Statecharts?

✅ **Less Bugs**

You probably all have seen code using various booleans to indicate the state of something. `isLoading` and `isError` are typical examples I see quite frequently when dealing with some kind of request. This often goes hand in hand with code like this:

```js
if (isError) {
  // return some kind of error
}

if (isLoading) {
  // show loading indicator
}

// It is neither loading nor error. It must have been successful
// return actual content
```

You have to make sure `isLoading` and `isError` are kept in sync. Should `isError === true` be allowed while loading? If that's not the case you will end up in an **impossible state** if both of them are `true` at the same time. You would still show the error even though a new request has been started.

Using a Statechart you would also model the success state explicitly. Statecharts require you to be in a single state at all times (*I consider parallel states still as a single state*).

*Yes, this is a simplified example where a single string `state` variable with values like `"loading"`, `"error"` or `"success` would already be a great improvement. But you hopefully get the point - it can get way more complex. (Also the string solution would also allow for hard-to-spot bugs due to typos).*

⚠️ **Sidenote about the flaws of Booleans:** Booleans are not only exclusively suitable for binary states like `isOpen`, but also they are not extensible (you can't add an `isOpening` if you'd like to use it for animation or something). Do not try to spread a single state across multiple booleans.

---

✅ **Exploring States and Transitions**

This benefit has already been implicitly described in the previous example. Since you would model the possible states beforehand and can (and also should) visualize them, you would spot missing states. A state machine must always be in a single state.

So what about the initial state in the previous example? Does it start in a loading state straight away? We were also missing some kind of a success state. Will the error state be a final state? Do we need some kind of automatic retry? A cache for the results?

It is way easier to even ask yourself these questions when looking at a diagram compared to (maybe already cluttered) code.

With statecharts, you will be using a **top-down approach**. You would first start with an idea, start the first scratch, flesh out some kind of visualization (at least in more complex scenarios) and write the actual code in the end. Whereas without statecharts you would more likely be using a bottom-up approach and write some code pretty early.

---

✅ **Easier Communication**

A visualization of the logic is not only very useful for implementation but also for communication. Can you show your code to business experts arguing about correctness or completeness? You can't. But you can show them a diagram with states, events, transitions, and so on. 

The visualization can help to bridge the gap between business experts and developers. It helps to solve the actual business problem in a correct way. You can (and should) use actual business terms for states and events. This makes it a handy tool in combination with [Domain-Driven Design](https://blog.jannikwempe.com/domain-driven-design-introduction). Using tools like Event Storming works well together with the concept of statecharts.

---

✅ **Improved Readability and Maintainability**

With statecharts you describe behavior in a single place instead of having it scattered everywhere in your code. This enables a nice **separation of concerns** between behavior/logic and presentation (in case of using it in a frontend). Both of these will be easier to understand when being separated. 

I highly recommend watching the talk [Finite State Machines in Vue 3](https://youtu.be/fT9p9CCSrn8) by [Sarah Dayan](https://twitter.com/frontstuff_io). She refactors some Vue code using [XState](https://xstate.js.org/) (a JavaScript statecharts library). The result is much more readable after the refactoring.

Having the behavior separated in a state machine also enables you to reuse it. You can replace the UI and keep the logic. You can't do that with presentation and behavior concerns being mixed.
 
This also allows you to share common logic — not only within your codebase but also with others. Great examples are [XState Catalogue](https://xstate-catalogue.com/) and [Stately Registry](https://stately.ai/registry)

*The benefit isn't unique to state machines. You can also separate behavior using something like hooks. Nevertheless, state machines somehow enforce to make use of that benefit.*

---

✅ **Better Handling of Complexity**

 Think back to the solution using booleans (or a string) to represent state. You will end up in pure chaos if it is getting complicated ⁠— I promise. How would you represent parallel or nested states?

Features will be added over time and things are getting out of control. It is starting to become unreadable, error-prone and no one likes to touch that code. Statecharts start to shine when things are getting complicated. I find the following diagram by [Matt Pocock](https://twitter.com/mpocock1) very accurate.

*The diagram is specific to React, but I think it is even valuable if you don't have experience using React. XState is a statechart library. It could be generalized to statecharts.*

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629041431148/X-yUOjhxl.png)

It visualizes nicely how statecharts will pay off over time. I'll get to the learning curve later on 😉

---

✅ **You Can Generate Stuff**

Modeling behavior using statecharts enables you to generate various things because their description can be interpreted.

My favorite tool and a great example for this is [XStates Visualizer](https://xstate.js.org/viz/). Just paste your state machine code in and get an interactive visualization. Does it get any cooler?

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629042148370/aJLxZU3Qb3.png)

But it doesn't end at visualizations. You could generate much more. You could even generate tests 🤯 [David Khourshids](https://twitter.com/DavidKPiano) talk [Write Fewer Tests! From Automation to Autogeneration](https://youtu.be/tpNmPKjPSFQ) is exactly about that. Definitely interesting to watch.

---

✅ **There is a Standard**

There is a W3C standard: [State Chart XML (SCXML): State Machine Notation for Control Abstraction](https://www.w3.org/TR/scxml/). Yes, the existence of a standard in itself isn't a benefit. But it comes with a few implicit ones. They worked for about 10 years on that. For me, it is also an indicator that the general concept is valuable and won't go away anytime soon.


# What are the Downsides of Statecharts?

❌ **Initial Learning Curve**

As with any new thing, there is an initial learning curve. You have to learn about the general concept of statecharts ([my first blog post of my state machine series](https://blog.jannikwempe.com/what-are-state-machines-and-statecharts) might help here). Probably you will use some kind of a library that you'll have to learn.

Scroll a little up and see the diagram about time and complexity. Initially, it will be harder and even comes with an overhead. But it will pay off.

The learning curve makes it even harder if you are working in a team. In that case, it is not only about you but also about the others.

---

❌ **Additional Library**

You don't need any library for simple state machines, but most probably you will use a library. That adds another dependency. You have to update that, take the risk of not being maintained, the bundle size increases, and whatever other downsides of using 3rd party libraries there are.

# Conclusion

*TLDR;*

✅ **Less Bugs**

✅ **Exploring States and Transitions**

✅ **Easier Communication**

✅ **Improved Readability and Maintainability**

✅ **Better Handling of Complexity**

✅ **You Can Generate Stuff**

✅ **There is a Standard**

❌ **Initial Learning Curve**

❌ **Additional Library**

---

As you may already have noticed by reading the article, I think the benefits outweigh the downsides by far. Statecharts are a great tool in a developer's toolbelt. It will not only improve your code's quality, readability and maintainability but it is also a great communication tool. At least I am sold on statecharts 😉

There may be situations when you don't need state machines. For example in the case of very simple logic with a low possibility of complexity added later on. But it will do no harm to also use statecharts here if you use it somewhere else in the project anyways.

---

Do you already use statecharts? Do you plan on using them? Have I missed any important benefits or downsides? I would love to get your point of view.