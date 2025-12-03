# Abstract
- Building a web application often requires managing state and reactivity.
- Svelte is known for its simple reactivity model and state management.
- I have been learning how to better use Svelte's state management to improve
    the performance and reliability of my personal website.

# DevLog: Managing State & Memory in a SvelteKit Application
- I encountered performance issues after implementing article indexing on my
    personal weblog.
    - I had not realized that data fetching may occur even when the request was
        not valid.
        - Spam requests caused API rate limits to be exceeded.
- While addressing this issue I learned more about Svelte stores and SvelteKit
    application state.
    - This is the first time I have used Svelte or SvelteKit in an application.
    - Experimenting with Svelte stores, context, and module scoped variables
        led to additional improvements.

## If Memory Serves
- It had not occurred to me that SvelteKit applications maintain application
    memory between requests.
    - This was something I was familiar with from other server frameworks.
    - The compiled nature of Svelte components had convinced me that the all
        state would be compiled to component-specific memory.
        - This is only true for runes.
- Module scoped variables persist between requests in SvelteKit applications.
    - This includes Svelte stores defined in module scope.

### In Memory of Past Requests
- My application was fetching resources for every request and storing them all
    to a shared Svelte store.
    - Each request would rewrite the same data to the same store.
- I realized that fetching could be throttled instead of managing store updates
    directly.
    - The resources only needed to be fetched every so often to check for
        updates.
- I implemented a in-memory cache to limit the frequency of data
    fetching.
    - The store would still be rewritten on every request, but the data fetching
        would only occur periodically.
    - This limits the number of requests made to external APIs.
    - The processing required to rewrite the store is minimal.

## This Requires Some Context
- I had been thinking about state management in Svelte applications wile
    investigating a memory leak.
    - The context-based theming system I had implemented was holding references
        to styles objects across requests.
    - The component stores were subscribed to global theme stores.

### Getting Hooked on Svelte Stores
- The system is meant to provide a "custom hook" that could initialize component
    context without requiring cleanup.
    - Svelte discourages side effects and manual lifecycle-driven management.
- I discover that Svelte stores provide several helpful mechanisms for
    automating event subscriptions.
    - Stores can be initialized with callbacks that trigger on the stores own
        lifecycle events.
    - Careful use of store start/stop callbacks allowed safe subscription to
        persistent global stores.
- Better understanding the Svelte store API significantly improved the
    development experience.
    - I rewrote the theme interface to use Svelte store contracts.
    - This allows the component to use Svelte's `$` syntax to automatically
        subscribe and unsubscribe from the store.
