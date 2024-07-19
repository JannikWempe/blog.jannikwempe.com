---
title: "State Machines and Statecharts: What are they?"
datePublished: Sun Aug 08 2021 11:30:19 GMT+0000 (Coordinated Universal Time)
cuid: cks34jm860tnbees10voq1pwv
slug: what-are-state-machines-and-statecharts
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1628420625532/uKAJS0xuw.png
tags: development, computer-science, developer, state, mathematics

---

If you are working as a developer (software engineer, coding wizard, or whatever you want to call it), chances are high you came across the term *state machine*. But do you also know what they are? Can you explain them to others? What is a statechart? This article is here to answer exactly these questions.

If you are here for fancy code in a specific language using a specific library you are wrong. This will be more about theory and will be language and library agnostic. This is also not about praising state machines or about their various benefits (even though they are great!). These topics will be part of follow-up articles. The "What are they?" will be more than enough for one article, you will see. Let's get started.

## What are State Machines?

**What actually is *state* in the context of state machines?**

Think of it as a status or mode of a system. Let us take a simplified oven as an example. It can have the states `off` and `heating`. The temperature of the oven is NOT the state. A state is something qualitative, not quantitative, and it is finite. Therefore the term state as used in frameworks like React or Vue is different from state in state machines. In React you would also refer to something like a `searchTerm` as state, but that would not be a state in a context of a state machine (there are infinite variations of possible `searchTerm`s; a state would be something like `idle`, `searching` etc.).

---

Maybe you have also heard about *Finite State Machines (FSM)*? Have you googled "state machines" before? FSMs will be all over the place. This is how it looks like:

![state machines google search](https://cdn.hashnode.com/res/hashnode/image/upload/v1628324514880/K8F9JoCmB.png)

Ok fine, but **what is the difference between state machines and finite state machines?** It is rather obvious: it's the word "finite".

FSMs are a subset of state machines. There are also possibly infinite state machines (like the [Turing Machine](https://en.wikipedia.org/wiki/Turing_machine)). BUT almost always if someone is talking about state machines the person is in fact talking about FSMs. The Google search should already prove that point, right? Also, [Wikipedia redirects you to the FSM article if you search for "state machines"](https://en.wikipedia.org/wiki/state_machine). And let's be honest, a person talking about something like a Turing Machine will most probably let you know 😉 Therefore I will from now on also use state machines and FSMs interchangeably.

### Finite State Machines (FSM)

With the terminology about state machines and FSMs out of the way let us now answer the question about what FSMs actually are.

An FSM is a mathematical [model of computation](https://en.wikipedia.org/wiki/Model_of_computation). It is always exactly in one state at a time. And, as the name suggests, consists of a finite number of states. FSMs are part of the automata theory. They are also called *finite state automata (FSA)*.

![state machines as part of automata theory](https://cdn.hashnode.com/res/hashnode/image/upload/v1628336465395/khx5SInlYp.jpeg)

I can already hear you thinking that this is being way too theoretical. It is getting more practical now. Let me show you the different parts of a state machine using a state(-transition) diagram showcasing the behavior of a video player:

![state machine example](https://cdn.hashnode.com/res/hashnode/image/upload/v1628420019200/X1wpLZ9Ix.png)

This diagram depicts all mandatory parts of an FSM: There are three different **states** (`stopped`, `playing`, `paused`) whereas `stopped` is the **initial state**. There are multiple **transitions** between states. A transition is a function that determines the next state based on the current state and an **event** (`f(currentState, event) = nextState`), e.g. being in the `playing` state and receiving a `pause` event would lead to a transition to the `paused` state.

It is worth mentioning that this has the result that there is no next state (and therefore no transition) if the current state does not react to the given event. Given being in the `stopped` state and receiving a `pause` event, we would stay in the `stopped` state (`f(stopped, pause) = stopped`).

These are the mandatory building blocks of a state machine:
- finite, non-empty set of **states**
- **initial state**
- finite, non-empty set of **events** (aka input alphabet)
- **transitions** (aka state-transition function)

Optionally there are zero or more **final states**, which are states without any outgoing transitions, meaning there is no possible next state (therefore final).

Remember state machines being a mathematical model? This is how you would define the aforementioned using math:

![state machine mathematical model](https://cdn.hashnode.com/res/hashnode/image/upload/v1628337989607/oGAPOoD6C.png)

That is already a very good foundational knowledge about state machines. There is more to it, like different classifications (deterministic vs. non-deterministic and Moore machines vs. Mealy machines), but I would consider them as extended knowledge. Therefore I will not go into details here.

Let's answer the next question with more practical use: What are statecharts?

### Statecharts

Okay great, that's it about state machines, but what are statecharts? State machines are helpful, but they can get very complex and lack some functionality.

Imagine the video player example above would be extended to also include whether the player is maximized or not. You would have to duplicate the states to something like `playing maximized`, `stopped maximized`, and so on.

Can you imagine how wild this would get with complex systems? There is even a term for that: [state explosion](https://statecharts.dev/state-machine-state-explosion.html) (aka exponential blow-up problem). But don't worry, statecharts to the rescue. To describe the need for statecharts with the words of its inventor David Haral (therefore they are also known as Harel statecharts):

> However, it is also generally agreed that a complex system cannot be beneficially 
described in this naive fashion [referring to FSMs], because of the unmanageable, exponentially growing multitude of states, all of which have to be arranged in a ‘flat’ unstratified fashion, resulting in an **unstructured, unrealistic, and chaotic state diagram**.

(taken from [STATECHARTS: A VISUAL FORMALISM FOR COMPLEX SYSTEMS by David Harel from 1987](http://www.inf.ed.ac.uk/teaching/courses/seoc/2005_2006/resources/statecharts.pdf)

Statecharts are an extension to state machines. They extend state machines with various features. Some of them are *(the list is not exhaustive)*:
- **Hierarchical states** (aka nested states)
- **Orthogonal states** (aka parallel states)
- **History state**
- **Conditions** (enabling guarded transitions)
- **Actions** (aka outputs)

I would refer to hierarchical and orthogonal states as the main features because they are directly tackling the problem of state explosion.

I'll explain the aforementioned features by extending our video player example:

![statechart example](https://cdn.hashnode.com/res/hashnode/image/upload/v1628418544061/o1JdyQ0RP.png)

*Please note that this is a simplified example. It is missing crucial parts like some kind of an error state.*

**Hierarchical States**

I extended the example with the concept of the player being loaded. Therefore I added a new `loading` state and a `loaded` state containing the logic from the first example. We refer to `loading` as an **atomic state** (or simple state) and to `loaded` as a **compound state** (a state having nested states). We refer to `loaded` as a **parent state** of `stopped`. Whereas `stopped` is a **sub-state** of `loaded` (ignoring the `window` part for now).

The compound state behaves just like a statechart on its own. Entering a compound state always means also entering one of its sub-states (remember a state machine always being in a state). Think of the semantics of `loaded` like an **XOR**. `loaded` itself is always in a single state.

Hierarchical states allow us to cluster states and to zoom in and zoom out on the level of detail. `loaded` is an abstraction of details. We could zoom out by just hiding the content of `loaded` and only looking on a more abstract level on the `loading` and `loaded` state without caring about the sub-states of `loaded`.

Also, note the transition from `loaded` to `loading` with the `reload` event. This means that our machine would always transition from `loaded` to `loading` on receiving the `reload` event no matter in what sub-state of `loaded` it is in.

---

**Orthogonal States**

Orthogonal states are depicted by a dashed line. `loaded` has an orthogonal (or parallel) state describing the `play` **AND** `window` **component** (or orthogonal region). The "AND" is important here. It refers to the logical AND (in contrast to the XOR above).

We are always in a state of all the orthogonal components at a time. Therefore we would refer to the substate of `loaded` like `(stopped, minimized)` as a single state (remember a hierarchical state always being a single state). We would say "The video player is in the stopped, minimized sub-state of loaded" or "The video player is in the minimized state of the window component". A component also acts like a statechart on its own.

Orthogonal states reduce the required states. Without it, we would need a single state for all possible combinations of the `play` and `window` component states.

---

**History**

History is a useful concept for nested states. You can see an example of that in the `window` component. With a history state, you remember the last state. In our example we would end up in `(stopped, maximized)` if we enter `loaded` again and have previously left it in the `window` components `maximized` state.

If we enter `loaded` for the first time we would end up in the `(stopped, minimized)` sub-state. With the introduction of the history state, the initial state is often also referred to as **default state**, because it will only be transitioned to if there is no previous history or no history state at all.

---

**Conditions**

Conditions are enabling **guarded transitions**. A transition only happens if the condition is fulfilled. In our example, it is only possible to enter `maximized` if `play` is in `playing`. If the machine would receive a `maximize` event but the player is stopped nothing would happen. In this example, a condition is enabling us to synchronize parallel states.

In addition, conditions allow us multiple outgoing transitions for the same event and state (all but one has to have a condition). For example, conditionally transitioning to a light or dark mode based on user preferences.

---

**Actions**

Actions are outputs of your statecharts. They are side-effects allowing us to interact with the world outside of the machine itself or to send events to the machine. They are instantaneous and ideally take zero amount of time (you are not waiting for something to be finished, e.g. a request to respond). Think of actions like "fire and forget".

Actions can happen in three different scenarios: **entering** a state, **exiting** a state, or when **transitioning**. I included examples for all of those in the video player. 

When entering `loading` we fire the `showLoadingIndicator` action and when exiting that stat we would fire `hideLoadingIndicator` *(This is probably not what you would do, because you could just show the loading indication by checking the state. But you could do it this way and I was lacking a better example)*. Also, for the transitions of the `play` events, I added the `start` action. This action could send a signal to an actual video player implementation.

---

Wow, that was more than I thought myself beforehand 😅 Even though it is by no means complete. There are a lot of nuances to the mentioned features and even more features (like actions, delays and timeouts, and much more). But by now you should be familiar with the most important concepts.

## Conclusion

State machines are great but they are much more powerful (and useful) when they are extended by the stateschart features. In fact, often people talking about state machines are referring to FSMs or even statecharts (also myself).

There was a W3C standard created (after about ten years of work): [State Chart XML (SCXML): State Machine Notation for Control Abstraction](https://www.w3.org/TR/scxml/). You can check it out if you are interested in the details. Also, I recommend checking out [statecharts.dev](https://statecharts.dev/). They also have a nice [glossary](https://statecharts.dev/glossary/).

I hope this article was useful for you. I will follow up with articles about the benefits of state machines and their actual implementation using XState in TypeScript - I promise. You could help to make the world of developers a better place by sharing your knowledge about state machines.

## Ressources

[National Institute of Standards and Technology](https://xlinux.nist.gov/dads/HTML/finiteStateMachine.html)

[XState: Finite State Machines](https://xstate.js.org/docs/about/concepts.html#finite-state-machines)

[Wikipedia: Finite State Machines](https://en.wikipedia.org/wiki/Finite-state_machine)

[statecharts.dev](https://statecharts.dev)

[STATECHARTS: A VISUAL FORMALISM FOR COMPLEX SYSTEMS by David Harel from 1987](http://www.inf.ed.ac.uk/teaching/courses/seoc/2005_2006/resources/statecharts.pdf)

[SCXML](https://www.w3.org/TR/scxml/)