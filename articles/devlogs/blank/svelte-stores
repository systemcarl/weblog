# DevLog: Managing State & Memory in a SvelteKit Application
The first real performance issue I encountered while developing my
    [personal website] came after adding indexing support for my articles.
To compile the different lists of highlighted articles to be displayed on
    the landing page,
the web application needed to fetch an index file categorizing all the available
    articles.
After deploying the update, I quickly realized that the application was fetching
    the index file (and every other resource) on every request,
including invalid requests made by bots and spammers.
This caused the article repository to exceed its API rate limits.
With the rate limits exceeded, none my articles could be retrieved and
    served by the application.

I was hoping to avoid any premature optimizations during the early stages of
    development,
but there needed to be some limits on how often these resources were fetched.
While implementing a cache to address this issue and some tangentially related
    memory leaks (request-specific state persisting indefinitely),
I learned a lot about managing state and memory.
Understanding Svelte stores and context, and also JavaScript module scope,
significantly improved the stability of the application and the experience
    of working with SvelteKit.

## If Memory Serves
Now, I'm not stranger to server-side development, and I've worked with server
    frameworks, like Django, that can maintain state between requests.
However, it had not occurred to me that SvelteKit applications would behave
    similarly.
Being a meta-framework that compiles client-side Svelte components to
    server-side code,
I naively assumed that the Svelte compiler would isolate all component state
    to individual requests
(this is only true for runes, as I'll explain later).
But at the end of the day, SvelteKit applications are still just Node.js
    applications
that maintain their memory over the lifetime of the server process.

Many Svelte examples and tutorials show stores defined in module scope
    (outside of component scripts).
This is perfectly fine for client-side applications,
as each client has its own isolated instance of the application.
However, in a server-side context,
any stores defined in module scope are retained and reused across requests.
This means that any data stored in these stores will persist indefinitely,
accumulating data from every request leading to excessive memory usage,
and potentially leaking sensitive information between users.

### In Memory of Past Requests
Not playing much attention to this when I [first created the application],
    every request would fetch required resources,
    ([mostly for configuration data]),
and expose it to the Svelte components via a module-scoped Svelte store.
I didn't realize that this store would persist between requests,
and each request would just overwrite the same store with fresh data that was
    almost always identical to the stale data already stored.
While this didn't lead to any sensitive data leaks,
it did lead to an excessive number of requests to external APIs.
These requests only really needed to be made periodically to check for updates.

The most direct solution would have been to manage the store updates, only
    updating the store after the current data reached a certain age.
However, I realized that regardless the reason for fetching the data,
    repeated requests to the same URL were unnecessary.
Any time a specific resource was required, the fetching method could return
    the last response if it was still recent enough to assume there had been no
    changes.

Since module scope variables persist between requests in SvelteKit applications,
I simply defined a simple key-value structure that mapped resource URLs to
    their last fetched response and the time it was fetched.
Every server request still loads the resource and stores the result,
    but the fetching method now return the cached response, if still applicable,
    forgoing unnecessary network requests.
This strictly limits the number of requests made to external APIs, per URL,
    theoretically capping the requests to one per cache duration interval.
Additional logic could be added to avoid redundant store updates,
    but the processing required to rewrite the store is relatively minimal.

## This Requires Some Context
Fortunately, when the rate-limiting issue arose, I was already thinking about
    memory and state management.
My inexperience with Svelte and SvelteKit had also created a significant memory
    leak elsewhere in the application.
The interface for retrieving contextual theming information for component
    rendering was holding references to style objects across requests.
Each component render would create a new set of applicable styles that would not
    be garbage collected,
eventually exhausting all available memory.
The references were held by the global theme store (one of the global Svelte
    stores updated for each request),
to subscribe to theme changes and update the component styles accordingly.

### Getting Hooked on Svelte Stores
Leaning on patterns from more imperative front-end frameworks, like React,
    I had implemented a "custom hook" to manage component context using
    specialized initialization arguments.
Because the component theme state is contextual and not globally shared,
    and the context requires component-specific configuration,
a simple `.svelte.ts` would not have sufficed.
The difficulty however, was that Svelte discourages side effects and manual
    lifecycle-driven management.
To implement an easy-to-use interface without memory leaks, the context
    initialization needed to automatically cleanup references when the component
    was unmounted.
Otherwise, theme style objects would continue to accumulate in memory.

After studying the Svelte store API, I discovered several helpful mechanisms
    for automating event subscriptions
Initializing the stores with the optional start/stop callbacks allowed me to
    safely subscribe to persistent global stores.
When the local component store was first subscribed to, the start callback would
    itself subscribe to the global theme store.
When the component was unmounted and the local store was no longer needed,
    the stop callback would automatically unsubscribe from the global store,
    freeing any references for garbage collection.

Aside from fixing this memory leak, understanding the Svelte store API led to
    a much better development experience overall.
Realizing that any object with an applicable store contract could be used in
    place of a Svelte store,
I rewrote the theme interface to follow these conventions.
The Svelte `$` syntax could then be used directly against the theme interface
    to remove any need to manually subscribe and unsubscribe from theme styles.
This simplified any theme-aware component significantly,
    delivering an immediate return on the time invested both refactoring the
    theme interface and fixing the memory leak.
