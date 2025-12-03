<!-- # Abstract
- Svelte stores provide a simple way to manage state in Svelte applications.
- Stores can be used to share state between components or even between requests
    in a web application.
- Understanding how to manage application memory has been crucial in learning to
    use SvelteKit.

# DevLog: Managing State & Memory in a SvelteKit Application
- Svelte reactivity is known for being simpler to implement than other
    popular frameworks.
    - Svelte 5 provides automatic reactivity for state and stores.
    - `$derived` runes can remove the need for side effect management.
    - State runes for each component are compiled during the Svelte build
        process.
- Svelte stores provide an alternative way to manage state.
    - Stores are not compiled like component state.
    - Stores use simple subscription mechanisms to notify components of state
        changes.
- Understanding module scope and application memory is important for performance
    and security.
    - Any module-scoped variable, including stores, will persist between
        requests in a SvelteKit application.
    - References to component variables held in module scope will not be
        garbage collected.

## This Requires Some Context
- Shared state is often needed in web applications.
    - Both state runes and stores can be defined in module scope to share state
        across components.
    - Shared state is globally accessible and consistent across all components.
    - Module scope state will persist between requests in SvelteKit.
- Svelte also provides context APIs to share state between parent and child
    components.
    - Stores or simple variables can shared within localized component trees.
    - Context is not persisted between requests in SvelteKit.

### Contextual Context
- Svelte does not provide an interface for reusing component state management
    that's not globally shared.
    - There is no concept to React's custom hooks.
- Creating a

## If Memory Serves -->
