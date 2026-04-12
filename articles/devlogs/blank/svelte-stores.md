# DevLog: Managing State & Memory in a SvelteKit Application
The first real performance issue I encountered while developing
    [my personal website] came after [adding indexing support for my articles].
To compile the different lists of highlighted articles to be displayed on
    the website's landing page,
the web application needed to fetch [the weblog index file]
that categorizes all the available articles
and provides the information required to display them.
After deploying the update, I quickly realized that the application was fetching
    this index file (and every other resource) on every request
— including invalid requests made by bots and spammers.
This caused requests to [the article repository] to exceed
    [GitHub's rate limits].
With the rate limits exceeded, none my articles could be retrieved
and the landing page highlights were suddenly empty.

I was hoping to avoid any [premature optimization] during the early stages of
    development.
Collective industry opinion advises that optimization should be deferred until
    the application is developed enough to identify the true performance
    bottlenecks;
inefficiency identified early on may not matter in the end when other factors
    (*e.g.* network latency, third-party response times, *etc.*) dictate the
    overall performance of the application.
However, there needed to be some limits on how often these resources were
    fetched to guard against total system failures.

Since the solution involved re-evaluating how the application manages server
    [memory],
this was a good opportunity to address some state-related [memory leaks]
    (request-specific state persisting indefinitely),
Solving both these issues required understanding the nuances of [Svelte] and
    [SvelteKit] application state management
— which I didn't have when setting up a SvelteKit application for the first
    time.
Experimenting with [Svelte stores], [context], and module [scoped variables]
    eventually led to a much more reliable system form managing application
    state.

## If Memory Serves
Now, I'm not stranger to [server-side development], and I've worked with server
    frameworks, like [Django], that will maintain state between requests.
However, it had not occurred to me that [SvelteKit] applications would behave
    similarly.
As a [meta-framework], SvelteKit compiles client-side Svelte components to be
    generated and delivered by server-side code.
I had naively assumed that the Svelte compiler would isolate all internalized
    component state to individual requests.
— since [Svelte runes] are magically isolated during compilation, I figured
    this would apply to other Svelte state management mechanisms as well.
But at the end of the day, SvelteKit applications are still just Node.js
    applications
that tend to maintain their memory over the lifetime of the server process.

Many Svelte examples and tutorials show stores defined in module [scope]
    (outside of component scripts).
Information defined at the module scope is retained as long as the code is in
    use, and is shared by every piece of code that references it.
This is perfectly fine when running code only in the client,
as each client has its own isolated instance of the application that persists
    this information locally in the browser, only as long as the tab is open
(unless it's deliberately saved, of course).
However, on the server, there may be just one single process handling every
    request that ever made.
Any stores defined in module scope are retained and reused across these
    server requests.
This means that any data put into these stores on the server will persist
    indefinitely
— accumulating the data from every request —
possibly using excessive amounts of memory or leaking sensitive information
    about one user to another.

### In Memory of Past Requests
Not playing much attention to this [when I first created the application],
    every request would fetch required resources,
    ([mostly for configuration data]),
and expose it to the Svelte components via a module-scoped Svelte store.
I didn't realize that this store would persist between requests,
and each request would just overwrite the same store with fresh data that was
    almost always identical to the stale data already stored.
While this didn't lead to any sensitive data leaks,
it did lead to an excessive number of requests to external APIs that largely
just stored the same data over and over again.
Realizing now that the store was already being reused across requests,
it is obvious that these requests only really needed to be made periodically to
    check for updates.

The most direct solution would have been to explicitly manage the store updates,
only updating the store after the current data reached a certain age.
However, I realized that regardless the reason for fetching the data,
    repeating requests for the same resource was unnecessary, and generally problematic.
These resources rarely change, and requesting them often wastes my server's
    bandwidth and risks hitting rate limits.
Any time a specific resource (identified by it's [URL]) was required, the code
    responsible for sending the request could cache the responses.
If the cached response was still recent enough to assume no change had occurred,
    the cached response could be returned instead of making a new request.

Another Svelte store could have been used to manage this cache,
but there was not need for the store's reactivity features.
The main benefit of using a Svelte store is that elements of the application
    can subscribe to the store to receive updates about the store in real-time.
Since cache reads and updates are only used during specific server request
    lifecycle events, there's no need provide subscription-based updates.
Instead, [I simply defined a simple key-value structure] that maps resource URLs
    to their last fetched response and the time it was fetched.
Every server request still loads the resource
    (either for the cache or a request) and applies the result to the store,
resulting in unnecessary updates sent to the store subscribers.
Additional logic could be added to avoid redundant store updates,
    but the processing being wasted rewriting the store is relatively small.
The important point is that this strictly limits the number of requests made to
    external APIs, per URL,
theoretically capping the requests to one for each cache duration interval.

## This Requires Some Context
Fortunately, when the rate-limiting issue arose, I was already thinking about
    memory and state management.
My inexperience with [Svelte] and [SvelteKit] had also led me to create a
    significant [memory leak] elsewhere in the application.
The interface for retrieving contextual theming information for component
    rendering was holding references to temporary style data across requests.
Each component rendered (potentially dozens per request) has its own local
    Svelte store containing style properties specific to component instance
(*e.i*, every text element keeps its own independent set of style properties,
    despite many having the same visual appearance).
Since the theme interface is defined globally — at module scope —
the references it retains to the component's local stores are carried across
    requests
and past component style data is never [garbage collected]
    (automatically deleted from memory when no longer needed).
> [!NOTE] This is an easy mistake to make when working with Svelte stores in
    SvelteKit applications.
When subscribing to a global store from a component, the store needs to
    reference the subscriber to send updates to it.
If the subscriber never unsubscribes, deleting the reference, the store will
    continue to hold a reference to the subscriber indefinitely,
keeping the component and any data it references in memory after the request is
    completed.

### Getting Hooked on Svelte Stores
Leaning on patterns from more imperative front-end frameworks,
    like React,[^hook] I had chosen to implement a "custom hook"
    (a reusable function that encapsulates stateful logic)
to manage component context (including theming) based on initialization
   arguments.
As long as the custom hook does not rely on Svelte compiler magic, there's no
    reason it can't be implemented as a simple JavaScript function.
The only challenge is that
    [Svelte discourages side effects and manual lifecycle-driven management].
Even if I wanted to go against the grain and manually manage the component
    lifecycle I would have to ensure every component that used the custom hook
    also implemented the necessary lifecycle management to avoid memory leaks
— this would be a nightmare to maintain and easy to forget.
To implement the easy-to-use interface I desired (without memory leaks)
    the context initialization needed to automatically cleanup references
    automatically when the component was unmounted.
Otherwise, theme style objects would continue to accumulate in memory.
> [!NOTE] Svelte provides a way to reuse stateful logic between components
    using [`.svelte.ts` files].
However, since these are just modules that are imported by the component,
they have the same module scope as any other module.
Since the component theme state (*i.e.*, the instance-specific style
    properties) is contextual and not meant to be globally shared,
a simple `.svelte.ts` would not have sufficed.

After diving into the [Svelte store API], I discovered a helpful mechanism
    for automating event subscriptions.
[Initializing the stores with the optional start/stop callbacks] allows safely
    subscribing with guaranteed cleanup logic.
[By chaining global and local store subscriptions with unsubscribe callbacks],
references are no longer retained after the request is completed.
After the request is completed, the component is unmounted.
When the component is unmounted, the component's local store is stopped,
   triggering subscription from the global store.
Once unsubscribed, the component's local store and its data are eligible for
   garbage collection and will be automatically disposed of to free up memory.

Aside from fixing this memory leak, understanding the Svelte store API also
    led to a much better development experience overall.
Realizing that any [Javascript object] [with an applicable store contract] could
    be used in place of a Svelte store,
I rewrote the theme interface to follow Svelte store conventions.
This allowed me to use [the Svelte `$` syntax] directly against the theme
    interface to remove any need to manually subscribe from theme styles.
This [simplified theme-aware components] significantly,
    delivering an immediate return on the time invested both refactoring the
    theme interface and fixing the memory leak.

## A Sound Conclusion
When confronted with these issues, I considered just slapping a quick fix on the
    code to make the symptoms go away.
For example, simply adding logic on the server to not load the index until after
    the request was validated
would have reduced the number of requests enough to stay within the rate limits.
Individually unsubscribing from stores in component lifecycle hooks where
    the memory leaks were occurring would have solved each issue, case by case.
However, these solutions would not have addressed the foundational issues with
    how the application was managing state and memory.

Aside from having a much more stable and robust application,
taking the time to understand the underlying mechanisms of Svelte's state
    management was worth the extra time and effort.
It's fine to just read a few tutorials and dive headfirst into development with
    a new framework,
but it's easy to miss important details and apply patterns from previous
    experience that don't quite fit.
Eventually, to create something that is truly inline with the framework's design
    and best practices,
it takes a deeper understanding of the framework that can only come from not
    just asking "how?" but also "why?".
Searching out satisfying resolutions to these issues was exactly the kind of
    learning experience that leads to better solutions in future.

[^hook]: Since React's [introduction of hooks], [custom hooks] have become a
    staple of React application development.
