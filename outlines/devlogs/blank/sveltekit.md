# Abstract
- I released the first version of the application I'm developing to create my
    personal website and weblog.
- This release included logging, error monitoring, testing, and configuration,
    but no content.
- I found SvelteKit to be a capable framework, but it has limitations.

# DevLog: Setting Up SvelteKit for a Personal Blog
- I am building a [personal website] and blog.
    - The goal is to create a platform to share my [portfolio] and ongoing
        projects.
- I chose to build the frontend using [Svelte] and [SvelteKit].
    - Svelte provides a simple and efficient way to create reactive web pages.
    - SvelteKit extends the Svelte component framework to provide a full
        server-side rendered application.
    - Svelte's low overhead and portability make it ideal for a long-standing
        personal project that can be easily maintained and updated.
    - This is my first time using SvelteKit; this also serves as a personal
        learning experience.
- My first goal was to create the necessary scaffolding for the project.
    - I implemented simple infrastructure for observability, configuration, and
        testing.
    - Internal utility interfaces will make development of the application
        efficient and consistent.

## Laying the Groundwork
- The standard environment for building and running SvelteKit applications is
    [Node.js].
    - The application will also be deployed to a Node.js environment to
        facilitate real-time [server-side rendering (SSR)],
        - using the [SvelteKit Node.js adapter].
- [Vite] is commonly used to configure and build the SvelteKit application.
    - Vite also provides
        - a development server with hot module replacement (HMR);
        - a Jest-based testing framework, [Vitest];
        - and other useful plugins, such as the [Vitest Browser] plugin.
- Setup was easy with Vite's template wizard.
    - I installed and configured:
        - TypeScript support,
        - ESLint,
        - and Vitest.
    - I chose to manage dependencies using [npm], as [Yarn] support is still
        limited.
- The initial project provided a simple starting point to implement other
    development tools.

## Setting Up the Scaffolding
- Providing simple interfaces to implement common tasks makes development more
    efficient and maintainable.
    - ["Don't Repeat Yourself" (DRY)] is a common, long-standing programming
        principle.
    - Wrapping dependencies makes upgrading or replacing dependencies more
        manageable.

### Keeping Tabs on Things
- My priority when starting a project is establishing good observability.
    - Debugging can be difficult during early stages of development.
    - Adding support to log errors and other important events is essential.
- [Sentry] has been a favorite tool of mine, ever since they started advertising
    on [Syntax.fm].
    - I've always found the tools and services easy and reliable.
    - The [SvelteKit Sentry SDK] was easy to install and configure, with
        out-of-the-box support for both client-side and server-side error
        monitoring and automatic source map uploading.
- For now, the app forwards all log events to the application standard output.
    - This works well for local development and testing, and will be easy to
        integrate with a log aggregation.
        - I later used [Grafana Alloy] to collect and forward logs to a Loki
            backend when [deploying the application].
    - I used [Pino] to efficiently format log events.
        - [Pino-Pretty] can be optionally enabled to format logs during
            development.

### Testing, Testing, Testing
- Effective testing is essential to building a application that is reliable and
    maintainable.
    - Ideally, all the tests would provide complete [code coverage].
    - Tests still need to be easy to write and not raise false negatives when
        adding new features.
- I implemented several testing strategies to cover the entire application
    without excessive overhead or fragility.

#### Unit Testing
- [Unit tests] were used to cover utility functions and data loading.
    - Some consideration was required to mock SvelteKit-specific modules,
        such as `$env`.
    - There is no way to mock SvelteKit stores without a full mock
        implementation.
- I also included unit tests for package and application configuration.
    - These simple tests don't validate any particular functionality.
    - They instead protect against regression of important configuration and
        middleware.

#### Component Testing
- In my past experience, manageable [component testing] often requires
    categorizing components of different types.
    - I defined 3 types of components to separate concerns:
        - material components, which are simple components that bind styling,
            data, and events to the DOM;
        - functional components, which are composed of materials and other
            functional components to provide a unit of functionality;
        - and page components, which are required by SvelteKit to define routes.
- Functional and page components interface only with other Svelte components
    and utilities and not the DOM.
    - These code elements are easy to test using Vitest and the Svelte Testing
        Library.
- Material components interface directly with the DOM and rely on CSS for
    styling and layout.
    - A [Vitest Browser] plugin was recently released for Svelte to provide a
        real browser environment using [Playwright].
    - This provides a more realistic environment to test rendering with the
        option to target different browsers.
    - I chose to write state-based tests to
        - reduce the number of expensive browser tests by grouping coupled
            properties into unique test cases,
        - capture intent rather than implementation details.
- The only significant challenge was mocking Svelte components and dynamically
    rendering child components.
    - It wasn't too difficult to create a few simple utilities to mock
        Svelte components and mount generated child components.

#### End-to-End Testing
- [End-to-end (E2E)] tests are essential to ensure the application works, but
    are often expensive.
    - In this case, the goal is to test the application:
        - builds,
        - delivers the static HTML that wraps the rendered Svelte
            components,
        - and is capable of serving a simple request.
- There is no way to mock external dependencies.
    - Playwright only provides a way to [intercept network requests] from the
        client-side.
    - Mocking server-side service requests requires a custom mock
        implementation or intercepting requests at the network level.

### Knobs and Dials
- I made the decision to abstract all personal configuration away from the
    application code.
    - This includes theming and text localization.
    - Providing an interface for theming and localization early in development
        establishes simple interfaces to build upon.
    - Data can be provided as static JSON files, which can be easily updated or
        replaced.
- As development continued, I found that [abstracting the application content]
    was an integral part of the project's design.

#### Localization
- The localization of this application only requires mapping code tokens to
    appropriate text strings.
    - [Localization] can extend to personalization of content in addition to
        translating text.
    - A simple [object proxy] provided a straightforward way to retrieve
        optionally provided localization strings.

#### Theming
- I recently spent a lot of time working on white-label theming for the
    [Batched] platform.
    - [White-label software] requires flexible theming to support different
        client brands.
    - I learned a lot about the challenges of providing flexible theming
        options while maintaining aesthetic quality.
    - One solution is to define a theme data structure that guarantees
        consistency and completeness.
- The theme structure for this application defines sections of the application
    where theme variations can be applied.
    - Making these sections optional allows for both simple and complex themes.
    - Using named colours and other properties reduces redundancy and allows
        for section composition.
- Application backgrounds and accent graphics are also defined in the theme.
    - Background patterns and other images can be defined per section and theme.
    - The rendering of these graphics also supports SVGs that can be styled
        using theme colors.
- The supplied theme is converted to a predictable JavaScript object and CSS
    variables.
    - CSS variables provide a simple, efficient way to apply theming to
        application components.
    - Exposing the theme as a JavaScript object provides a way to access
        theme properties in functionality that cannot be achieved using CSS
        alone.
    - Most of the work was in defining intuitive fallbacks for missing or
        invalid theme properties.
- Both theme interfaces can be easily mocked for testing.
    - Theme CSS variables can be set on the browser test container to simulate
        theme application.
    - The JavaScript theme interface can be easily mocked.
- The biggest challenge moving forward will be coordinating concurrent themes
    between the server-rendered content and the personalized client-side
    presentation.
    - To present a personalized theme either
        - the server must render the application with the theme requested by the
            user,
        - or the client must re-render the application with the requested theme.

## An Empty Application
- The only feature I implemented with the first version of the application was
    error handling.
    - Error pages could be rendered with the error and logging utilities already
        implemented.
    - This also provided a way to test the Sentry integration.
    - Excluding placeholder content makes the first version of the application
        functionally empty.
- Implementing [test error routes] allowed me to validate the entirety of the
    application infrastructure.
    - Various error codes will return different localized error messages and
        generate log and error events.
    - The error pages fully implement theme colours, fonts, layouts, and
        graphics.

## Learning to Love Svelte
- [This iteration] of development was undoubtedly a success.
    - The interfaces and testing strategies development have so far made
        development extremely manageable.
    - [Continuing to build the application] within the SvelteKit framework has
        been frictionless.
- There were unexpected challenges when setting up the application with
    SvelteKit.
    - There is limited support and documentation for dynamically defining
        components children or slots.
    - Mocking Svelte components for testing is not officially supported.
    - Some SvelteKit modules do not test well, especially application state.
- I see the appeal of Svelte and SvelteKit, but has some drawbacks.
    - Svelte is great for easy, straightforward reactive components.
    - Svelte has limited support for fine-grained control of the DOM and
        application state.
    - SvelteKit & [Svelte 5] is relatively new; I expect the supporting tools
        and documentation to continue to improve.

[personal website]: https://carledwardlyons.ca
[portfolio]: https://github.com/systemcarl
[Svelte]: https://svelte.dev
[SvelteKit]: https://svelte.dev/docs/kit/introduction
[Node.js]: https://nodejs.org
[server-side rendering (SSR)]:
    https://en.wikipedia.org/wiki/Server-side_rendering
[SvelteKit Node.js adapter]: https://svelte.dev/docs/kit/adapter-node
[Vite]: https://vite.dev
[Vitest]: https://vitest.dev
[Vitest Browser]: https://vitest.dev/guide/browser/
[npm]: https://www.npmjs.com
[Yarn]: https://yarnpkg.com
["Don't Repeat Yourself" (DRY)]:
    https://en.wikipedia.org/wiki/Don%27t_repeat_yourself
[Sentry]: https://sentry.io
[Syntax.fm]: https://syntax.fm
[SvelteKit Sentry SDK]:
    https://docs.sentry.io/platforms/javascript/guides/sveltekit/
[Grafana Alloy]: https://grafana.com/docs/alloy/latest/
[deploying the application]: ./ci-cd.md#phoning-it-in
[Pino]: https://getpino.io
[Pino-Pretty]: https://getpino.io/#/docs/pretty
[code coverage]: https://en.wikipedia.org/wiki/Code_coverage
[Unit tests]: https://en.wikipedia.org/wiki/Unit_testing
[component testing]:
https://en.wikipedia.org/wiki/Component-based_usability_testing
[Playwright]: https://playwright.dev
[End-to-end (E2E)]: https://www.ibm.com/think/topics/end-to-end-testing
[intercept network requests]: https://playwright.dev/docs/mock
[abstracting the application content]:
    ./content-template.md#an-agnostic-application
[Localization]: https://en.wikipedia.org/wiki/Software_localization
[object proxy]:
    https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy
[Batched]: https://batched.ca
[White-label software]: https://en.wikipedia.org/wiki/White-label_product
[test error routes]:
    https://github.com/systemcarl/blank/blob/v0.0.1/src/routes/errors/%5Bcode%5D/%2Bpage.server.ts
[this iteration]: https://github.com/systemcarl/blank/tree/v0.0.1
[continuing to build the application]: ./content-template.md
[Svelte 5]: https://svelte.dev/blog/svelte-5-is-alive
