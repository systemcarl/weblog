# DevLog: Setting Up SvelteKit for a Personal Weblog
Hey! You found my blog.

There may be other places to find my content, but if you're reading this, you're
    probably on my [personal website].
If you're not, go check it out.
I built the website using [SvelteKit] to host my weblog articles and create a
    place to experiment with web technologies and share [my projects].

I decided to use the framework [Svelte], and its companion meta-framework
    [SvelteKit], because of its simplicity and performance.
I wanted a modern, fast technology stack that could support server-side logic.
SvelteKit provides a streamlined server-side rendering experience out of the
    box, with plenty of options for customizing server-client interactions.
Since Svelte can compile components down to portable JavaScript, it's also a
    safe bet for a long-standing project that will inevitably migrate to new
    technologies over time.
I haven't used either of these well-endorsed frameworks before myself, so I was
    also just excited to try something new.

With any new project, the fist step was setting up the development environment
    and application infrastructure
— the "scaffolding" of the application that will support development.
To start, I set up a new SvelteKit project with just most the necessary tools
    to provide:
- observability — functions and integrations to log events and track errors;
- configuration management — processes to load and retrieve application
    settings;
- and testing strategies — the dependencies and structure to easily write and
    run maintainable tests.

Taking the time to create these internal interfaces early on made the
    application code simple and consistent right from the start.

## Laying the Groundwork
To keep it simple, I began setting up the the SvelteKit project in the standard
    JavaScript run-time environment, [Node.js].
Because some of the content will be dynamic, the compiled
    [server-side application] will need to be run in a Node.js environment as
    well.
SvelteKit provides an [adapter] specifically for this scenario, providing the
    necessary build in a Node.js environment for a Node.js server.

These days, I like to use [Vite] whenever possible. It provides a fast and
    reliable development environment with a lot of useful tools:
- a development server with hot module reloading;
- [Vitest], a jest-based testing framework;
- no-hassle TypeScript integration;
- and a very useful plugin ecosystem.

Using Vite for React development made my life so much easier, so I was happy to
    see that SvelteKit is built on top of Vite.

The initial setup was painless. Using the Vite template wizard, I created a
    new SvelteKit project with:
- TypeScript support,
- ESLing pre-configured,
- and Vitest installed.

Normally, I prefer to use [Yarn] because I found it more flexible when
    installing local packages
(which can be nice for experimenting with custom dependencies).
However, it seems that SvelteKit doesn't quite support Yarn yet
    (or at least, I couldn't get it to work).
So, I used [npm] which gave me no issues.

From that point, I was ready to start building;
I could locally host a development server and run tests.
Later I would have to add a few more dependencies and tweak the some package
    configurations.
But with the initial setup being so quick and concise, any additional changes
    were easy to manage.

## Setting Up the Scaffolding
It's true, it may often be better to implement features first and refactor
    later to build in internal tools and establish patterns.
However, knowing exactly what I was going to need over the next several
    iterations of the application, I preferred not do everything twice.
I was careful to keep the initial setup minimal, only adding what I knew I
    would need right away.
The goal was really to follow the ["Don't Repeat Yourself" (DRY)] motto early on
    to avoid unnecessary rework later.
Abstracting common application logic into light wrappers and utility functions
    has the added benefit of decoupling the application code from third-party
    dependencies.

### Keeping Tabs on Things
My first priority was establishing good observability — the ability to monitor
    the application and track errors from the outside.
Debugging can often be challenging at the begging of a project as the code
    settles into the framework and runtime;
often times new dependencies and configurations can introduce unexpected issues
    that are hard to trace.
Having a way to investigate the application state beyond breakpoints and simple
    console logs can be a real lifesaver.

One tool I always like to use is [Sentry].
It's a popular error-tracking service that's hard to ignore if you're a fan of
    [Syntax.fm].
I've consistently found it very easy to use,
and in the case of SvelteKit, there is a [simple integration] that lets you
    handle errors both on the server and client, with automatic source map
    integration for easier debugging.

To handle all non-error-related logging events, I simply
    [sent logs to the application standard output]
 — via our friend, the console.
This works great during development,
and since I plan to host the application in my own server environment, I
    can easily aggregate these logs and handle them at the system level.
I later [implemented the log collection] using [Grafana Alloy] when I
    [set up the deployment environment].
To make sure the logging was efficient and consistent, I install [Pino],
    a fast JSON-friendly logging library, to structure and format the logs.
Using the [Pino-Pretty] plugin, I could also format the logs nicely during
    development to save me from squinting at compressed JSON strings all day.

### Testing, Testing, Testing
Ask anyone and they'll tell you how important testing is (at least while their
    boss is listening).
I believe most developers understand the value of testing, but every developer
    knows how time-consuming and thankless writing tests can be.
So, in practice, there's often compromise between comprehensive testing and
    shipping features.
In the end, the best way to ensure tests are written consistently is to make the
    process as easy and painless as possible.

Since this is a personal project with the goal of learning and experimentation,
I saw this as a great opportunity to try out different testing strategies.
Ideally, I wanted to establish a solid testing foundation that would fully
    ensure the function and presentation of the application
(at least against known points of failure; there's never a guarantee).

In my opinion, comprehensive testing goes beyond just [code coverage]
— checking to see whether every line of code is exercised by a test.
Even when completely covering all logic branches with unit tests, it's still
    possible that the application could not behave as intended.
When dealing with complex, stateful applications, like a component-based web
    application, it can also be exhausting to test every possible interaction
    and state change.

Instead, my goal was to implement complete testing strategies that would
    provide confidence in the application while still being fun to develop
    against.
Some of the tests I implemented to test package configuration and middleware are
    quite simple and possibly redundant.
In that case, the goal is just redundancy — making sure that the code I
    accidentally typed in the wrong file doesn't torpedo deployment.
In other cases, I focused on testing the code from the consumer's perspective,
making sure each component does what has been promised.

#### Unit Testing
[Unit tests] are great for testing utility functions and other isolated pieces
    of logic.[^utilities]
Since most of the SvelteKit interface is [functional], it's easy to write unit
    tests for data loading functions and state management
    utilities.[^data-loading]
Testing the data loading functions did require some additional consideration to
    mock SvelteKit-specific modules,
such as the environment variable wrapper [`$app/environment`].
Also, there is no easy way to mock SvelteKit stores
    (at least without writing a full mock implementation).
So to test the stores, I chose simply to reset the application state between
    tests to prevent data from bleeding over.

Beyond testing application logic, I also included unit tests for package and
    application configuration.[^packages]
These tests don't directly test application functionality, but instead test that
    application configuration methods are called correctly during build and
    startup.
Mocking Vite and SvelteKit modules made it easy to verify calls to configuration
    functions are made with the expected parameters.
In practice, these tests simply preserve the exact configuration expected to
    deploy and run the application.

#### Component Testing
Component testing — by that I mean unit testing for components —
instead focuses on testing functional components as a whole.
These tests are especially useful for verifying user interface components,
    where presentation and interaction can intertwined.

Typically when writing component tests, I find it helpful to categorize
    components into different types to separate concerns.
In this project, I defined 3 types of components:
- **material components**,[^material] which are simple components that bind
    styling, structure, and basic interaction (like buttons and form inputs);
- **functional components**,[^functional] which combine multiple material
    components or other functional components to provide more complex
    functionality (like modals and dropdowns);
- **page components**,[^page] which are a requirement of SvelteKit for defining routes
    (generally prefixed with '+').

The functional[^functional] and page[^page] components were implemented to only
    interface with other Svelte components and the SvelteKit framework.
This made it easy to test with a the Svelte testing library, which can render
    Svelte components in isolation.
These tests borrow heavily from [component-based testing] strategies, primarily
    focusing on the "wiring" of components together with properties and state.

Material components,[^material] on the other hand, are responsible for
    interfacing directly with the DOM
— defining DOM element composition and styling, and handling user interaction.
These components are more challenging to test because they often rely on CSS for
    important layout and presentation.
On their own, Vitest and the Svelte testing library don't provide a full browser
    environment to accurately compute and apply styles.
Fortunately, a [Vitest Browser] plugin was recently released for Svelte to
    provide a more complete browser-like environment for testing these types of
    components using [Playwright] (or another compatible browser automation
    tool).
Since in-browser tests are generally slower to run, these tests will follow a
    more state-based approach,
grouping assertions into tests that verify each unique state of the component.
(There are no interactive elements currently planned for this project, so for
    now these will not need to test state transitions.)
These test then focus on capturing the intended presentation of the component,
    opposed to testing every individual property.

The only tricky part about component testing in SvelteKit is mocking child
    components.
Neither Svelte nor SvelteKit provide a built-in way to mock components.
I was able to work around this by manually [mounting test components] with the
    [`createRawSnippet`] function
Generated components could be use to mock comprised child components with
    [`vi.mock`]
and also passed directly as a `children` property.
I was not able to find a reliable way to substitute for named slots, so for now
    I simply avoided using them in any components.

#### End-to-End Testing
[End-to-end (E2E) tests], unlike unit and component tests, test the entire
    application as a whole.
These tests are useful to verify that each individually tested piece of the
    application works together as expected.
Also, E2E tests are great for testing elements of the application that are
    difficult to isolate, such as routing and
    [middleware interactions that can't be run in isolation].
However, E2E tests are generally the slowest to run and difficult to maintain
    if the specifications are too rigid.
So, for this project, I simply wanted to test that the application builds and
    starts correctly (at least within the test environment),
wraps the rendered application components is the correct HTML structure,
    and can handle a simple HTTP request.

It would also be nice to test each route to make sure the basic functionality is
    is place.
However, [Playwright] only provides an API for [mocking browser requests], not
    requests from the server to third-party services.
This will be a problem for testing any dynamically generated page that relies on
    external data.
This is a problem I left for another day, since there does not seem to be an
    easy solution to put in place.
It appears this will require some custom middleware to
    intercept server-side requests or intercepting network requests in the
    test environment.

### Knobs and Dials
Early on, I made the decision to abstract all personalized content away from
    the application code.
Providing configuration options as static JSON files seemed like the best way to
    separate content that I may want to change frequently away from my
    rigorously tested application logic.
The data is [loaded during each request] and made available throughout the
    application [via SvelteKit data stores].
Bundling these settings into utility modules provided a simple interface to
    retrieve configuration values throughout the application.
Abstracting all the visual theming and text content (including localization)
    also meant that this application could be easily re-branded for any other
    long-form content site.

The implementation of the configuration management system was quite simple.
But as development progressed, I found [abstracting the application content]
    was a very important part of the application structure and design.

#### Localization
Generally, it's always good practice to abstract out text that may need to be
    changed
— either because the application has changed, or because the application is
    being used in a different language or dialectic region.
Unless the structure of the text is fluid (like placing the currency symbol
    before or after a price), it's usually sufficient to simply map tokens to
    localized strings.
In this project, this can easily apply to any text being displayed, allowing the
    application to completely customized to display any text content.

To achieve this with little overhead, I implemented a [simple utility] that
    creates a simple [object proxy].
The proxy object looks for custom text and simply falls back to placeholder text
    if none is provided.
This means that very little customization is required to generate a page that
    feels complete.

#### Theming
I've spent a lot of time recently sitting with the question of how to style
    a application, working on the [Batched] platform.
If your brand is very rigid and unlikely to change, it fine to hard-code styles
    directly into the web pages.
However, in many cases, users expect to be able to customize the look-and-feel
    of their applications
— be it to match their preferred light or dark mode, or to re-brand the
    application to match their own identity.
Building [white-labelled] software, I learned the challenges of providing
    flexible theming options that enable customization while maintaining
    aesthetic quality.
There are many ways to approach theming, but I found that creating a quality
    generic design requires defining a theme data structure that guarantees
    a consistent and complete set of style properties.

The theme structure I defined for this project defines "sections" of the
    application that may (or may not) be visually distinct
(like the main content compared to the footer, for example).
This allows the content creator to specify the complexity of the broader
    applications visual composition
— a single theme section could be applied to the entire application,
    or each section could have its own distinct style definition.
Within these sections, using key-referenced properties for colours, fonts,
    spacing, and other common settings reduces redundancy and ensures
    consistency across the theme.
```json
{
  "themes": {
    "default": {
      "sections": {
        "default": {
          "palette": "regal",
          "background": "liner"
        }
      },
      "palettes": {
        "regal": {
          "text": "#FCFAED",
          "background": "#111818"
        }
      },
      "backgrounds": {
        "liner": {
          "img": {
            "src": "/cel-tile.svg",
          },
          "fill": "background"
        }
      }
    }
  }
}
```
The [full theme definition] for my [personal website] is included in the
    deployment packages.
These definitions are compiled to dereference the keyed properties into
    a predicable structure with concrete values.
```json
{
  "sections": {
    "default": {
      // section properties are dereferenced from theme definitions
      "palette": {
        "text": "#FCFAED",
        "background": "#111818"
      },
      "background": {
        "img": {
          "src": "/cel-tile.svg"
        },
        // fill is resolved from the referenced section palette
        "fill": "#111818"
      }
    }
  }
}
```

I chose also define backgrounds and accents graphics as part of the theme.
Since it's often better to store assets separately from code repositories,
    it seemed like a natural step to define these assets with the theme.
This makes it easy to swap out images and graphics or even host them
    externally if desired.
The application also supports SVG images that can be styled using the colour
    palette defined in the theme, so coupling graphics with the theme works
    well.
```json
{
  "themes": {
    "default": {
      "sections": {
        "default": {
          "palette": "regal",
          "graphics": "default"
        }
      },
      "palettes": {
        "regal": {
          "background": "#111818",
          "accent-primary": "#534260",
          "accent-secondary": "#FF9F1C",
          "accent-shadow": "#FCFAED"
        }
      },
      "graphics": {
        "default": {
          "titleAccent": {
            "src": "/chevron.svg",
            // map fill of SVG class to palette colours
            "colourMap": {
              "img-fill-primary": "accent-primary",
              "img-fill-secondary": "accent-secondary",
              "img-shadow": "accent-shadow"
            }
          }
        }
      }
    }
  }
}
```
The `colourMap` property is compiled to CSS class definitions that apply a
    fill colour to the corresponding [SVG elements].

The theme definition is compiled from the JSON data to a predictable JavaScript
    object structure and a set of class-scoped CSS variables.
I chose to expose both interfaces to provide flexibility in how styles are
    applied throughout the application.
CSS variables are simple and efficient, separating style from both structure and
    function.
```css
.theme-default .section-default .typography-body {
    --font-family: 'Public Sans', sans-serif;
    --font-size: 1.125rem;
    --font-weight: inherit;
    --font-style: inherit;
    --line-height: inherit;
    --letter-spacing: inherit;
    --text-decoration: inherit;
    --text-colour: #FCFAED;
    --text-shadow: none;
}
```
Occasionally, it's useful to access theme properties directly in JavaScript,
    when style value are needed to provide functionality not possible with CSS
    alone.
```typescript
const { getSection } = useTheme();
const section = getSection('default');

console.log(section.typography.body);
// {
//   fontFamily: "'Public Sans', sans-serif",
//   fontSize: '1.125rem',
//   fontWeight: 'inherit',
//   fontStyle: 'inherit',
//   lineHeight: 'inherit',
//   letterSpacing: 'inherit',
//   textDecoration: 'inherit',
//   textColour: '#FCFAED',
//   textShadow: 'none'
// }
```
The contextual theming by section, typography, or graphic is established with
    a custom helper that conditional synchronizes the CSS class context with the
    [SvelteKit context].
```typescript
<script lang="ts">
  import useThemes from '$lib/hooks/useThemes';
  const { makeProvider } = useThemes();

  const { children } = $props();

  // set the theme section context for child components
  const { provider } = makeProvider({ sectionKey : 'error' });
</script>

// apply the theme section CSS class to root component element
<div class={provider.className}>
  {@render children}
</div>
```
Implementing this system was less complicated than it sounds.
Most of the trouble was defining an intuitive system for
  [resolving missing properties] with sensible fallbacks or defaults.

Both the CSS and JavaScript interfaces test well.
CSS variables can be [set on the browser test container] to simulate different
    themes.
The JavaScript theme object can be [easily mocked] to provide different test
    values with needing to load a full theme definition.

The biggest challenge with this theming approach will be coordinating multiple
    concurrent themes.
The server generated pages will need to be resolved with the client's selected
    theme.
To present a personalized theme to each user based on their preferences, either
the server must render each page with the user's requested theme,
or the client must re-style the default page rendered by the server before
    painting it to the screen (something that SvelteKit does not like).
If the server does not render the page with the user's requested theme and the
    client does not successfully re-style the page before painting,
    the user may see a [flash of unstyled content (FOUC)] before the application
    finally resolves the user's settings.

## An Empty Application
With all the necessary tools in place, it was time to create a simple
    proof-of-concept.
To avoid getting bogged down in content creation, I wanted to simply create a
    simple page the could render and verify everything was working correctly.
Instead of a meaningless "Hello, World!" notice, I decided to implement some
    simple error handling, and add a route that would intentionally trigger an
    error.
With the logging and error tracking systems in place, I could fully implement
    this feature while verifying that errors were being captured and reported.
And by forgoing any actual content, the application was functional empty yet
    still complete.

Fully implementing this [test error route] could also validate the rest of the
    application infrastructure, besides just observability.
Each error code returns a different error message, defined by the localization.
The error page is styled using the application theming system, with colours,
    fonts, and a graphic defined by the theme (using the `error` section).

## Learning to Love SvelteKit
This [first iteration] of the application was undoubtedly a success, despite its
    underwhelming presentation.
The interfaces I had implemented and testing strategies I had developed made
    building out the application extremely manageable.
Since then, [adding content] to the SvelteKit application has been frictionless.

However, setting up the SvelteKit application was not without its challenges.
It was frustrating at times to work around limitations of the framework,
especially when the documentation was vague or just incomplete.
For example, mocking SvelteKit-specific modules for testing was not obvious from
    the documentation.
The solution I found was pieced together many examples of the `createRawSnippet`
    function from various issues ([like this one]) and Stack Overflow answers.
And I'm sure the solution I implemented is not officially supported, so I expect
    it may break in future releases.
Beyond the challenges of mocking components, it was disappointing that SvelteKit
    does provide an elegant way to isolate state between tests.

Overall, though, I do see the appeal of SvelteKit over other popular front-end
    frameworks.
The simplicity of Svelte's component model makes it easy to implement
— so long as you don't need fine-grained control over rendering or component
    state.
It's also worth noting that [SvelteKit], and the significantly redesigned
    [Svelte 5], are both relatively new.
So, hopefully, some of these more inconvenient deficiencies will be addressed
    as the frameworks mature.

[^utilities]: To separate [universal] and [server-only] code,
utilities are organized into their own respective directories within
    the [`src/lib/`] directory.

[^data-loading]: Data loading for the application is handled when generating
    the [main layout] by a [data loading function] that is also tested in an
    [adjacent test file].

[^packages]: To test that configurations are correctly applied to the Vite
    project and middleware,
[Vite configuration tests] and tests for both [client] and [server] SvelteKit
    [hooks] are included in the source repository.

[^material]: The [initial iteration] included several [material components] to
    populate the error pages, each with their own tests.

[^functional]: The [first release] of the application didn't include any
    [functional components]. However, [version 0.0.2] includes [examples] with
    tests.

[^page]: The Svelte components defined in the [routes] folder are used to
    define the [SvelteKit routing]. The [error page] is a simple example that
    also has its own [test file].

[personal website]: https://carledwardlyons.ca
[my projects]: https://github.com/systemcarl
[SvelteKit]: https://svelte.dev/docs/kit/introduction
[Svelte]: https://svelte.dev
[Node.js]: https://nodejs.org
[server-side application]:
    https://en.wikipedia.org/wiki/Server-side_rendering
[adapter]: https://svelte.dev/docs/kit/adapter-node
[Vite]: https://vite.dev
[Vitest]: https://vitest.dev
[Yarn]: https://yarnpkg.com
[npm]: https://www.npmjs.com
["Don't Repeat Yourself" (DRY)]:
    https://en.wikipedia.org/wiki/Don%27t_repeat_yourself
[Sentry]: https://sentry.io
[Syntax.fm]: https://syntax.fm
[simple integration]:
    https://docs.sentry.io/platforms/javascript/guides/sveltekit/
[sent logs to the application standard output]:
    https://github.com/systemcarl/blank/blob/v0.0.1/src/lib/server/logs.ts
[implemented the log collection]:
    ./ci-cd.md#phoning-it-in
[Grafana Alloy]: https://grafana.com/docs/alloy/latest/
[set up the deployment environment]: ./ci-cd.md#phoning-it-in
[Pino]: https://getpino.io
[Pino-Pretty]: https://getpino.io/#/docs/pretty
[code coverage]: https://en.wikipedia.org/wiki/Code_coverage
[Unit tests]: https://en.wikipedia.org/wiki/Unit_testing
[functional]: https://en.wikipedia.org/wiki/Functional_programming
[`$app/environment`]: https://svelte.dev/docs/kit/$app-environment
[component-based testing]:
    https://en.wikipedia.org/wiki/Component-based_usability_testing
[Vitest Browser]: https://vitest.dev/guide/browser/
[mocking browser requests]: https://playwright.dev/docs/mock
[mounting test components]:
    https://github.com/systemcarl/blank/blob/v0.0.1/src/lib/tests/component.ts
[`createRawSnippet`]:
    https://svelte.dev/docs/svelte/svelte#createRawSnippet
[`vi.mock`]: https://vitest.dev/guide/mocking#manual-mocks
[Playwright]: https://playwright.dev
[middleware interactions that can't be run in isolation]:
    https://github.com/sveltejs/kit/issues/14249
[End-to-end (E2E) tests]: https://www.ibm.com/think/topics/end-to-end-testing
[intercept network requests]: https://playwright.dev/docs/mock
[loaded during each request]:
    https://github.com/systemcarl/blank/blob/v0.0.1/src/routes/+layout.server.ts
[via SvelteKit data stores]:
    https://github.com/systemcarl/blank/tree/v0.0.1/src/lib/stores
[abstracting the application content]:
    ./content-template.md#an-agnostic-application
[simple utility]:
    https://github.com/systemcarl/blank/blob/v0.0.1/src/lib/utils/locale.ts
[object proxy]:
    https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy
[Batched]: https://batched.ca
[White-labelled]: https://en.wikipedia.org/wiki/White-label_product
[full theme definition]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.1/theme.json
[SVG elements]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.1/chevron.svg?short_path=a0e7720
[sveltekit context]:
    https://svelte.dev/docs/kit/state-management#Using-state-and-stores-with-context
[resolving missing properties]:
    https://github.com/systemcarl/blank/blob/v0.0.1/src/lib/utils/theme.ts
[set on the browser test container]:
  https://github.com/systemcarl/blank/blob/96343d1a78d60a59e9d36b0e1330cd8d33f50ee9/src/lib/materials/background.svelte.spec.ts#L105
[easily mocked]:
  https://github.com/systemcarl/blank/blob/96343d1a78d60a59e9d36b0e1330cd8d33f50ee9/src/lib/materials/background.svelte.spec.ts#L47-L57
[flash of unstyled content (FOUC)]:
    https://en.wikipedia.org/wiki/Flash_of_unstyled_content
[test error route]:
    https://github.com/systemcarl/blank/blob/v0.0.1/src/routes/errors/%5Bcode%5D/%2Bpage.server.ts
[first iteration]: https://github.com/systemcarl/blank/tree/v0.0.1
[adding content]: ./content-template.md
[like this one]:
    https://github.com/sveltejs/svelte/discussions/13518#discussioncomment-10871705
[Svelte 5]: https://svelte.dev/blog/svelte-5-is-alive

[universal]: https://github.com/systemcarl/blank/tree/v0.0.1/src/lib/utils/
[server-only]:
    https://github.com/systemcarl/blank/tree/v0.0.1/src/lib/server
[`src/lib/`]: https://github.com/systemcarl/blank/tree/v0.0.1/src/lib/
[main layout]:
    https://github.com/systemcarl/blank/blob/v0.0.1/src/routes/+layout.svelte
[data loading function]:
    https://github.com/systemcarl/blank/blob/v0.0.1/src/routes/+layout.server.ts
[adjacent test file]:
    https://github.com/systemcarl/blank/blob/v0.0.1/src/routes/layout.server.test.ts
[Vite configuration tests]:
    https://github.com/systemcarl/blank/blob/v0.0.1/vite.config.test.ts
[client]:
    https://github.com/systemcarl/blank/blob/v0.0.1/src/hooks.client.test.ts
[server]:
    https://github.com/systemcarl/blank/blob/v0.0.1/src/hooks.server.test.ts
[hooks]: https://svelte.dev/docs/kit/hooks
[initial iteration]: https://github.com/systemcarl/blank/tree/v0.0.1
[material components]:
    https://github.com/systemcarl/blank/tree/v0.0.1/src/lib/materials
[first release]: https://github.com/systemcarl/blank/tree/v0.0.1
[version 0.0.2]: https://github.com/systemcarl/blank/tree/v0.0.2
[examples]:
    https://github.com/systemcarl/blank/tree/v0.0.2/src/lib/components
[routes]: https://github.com/systemcarl/blank/tree/v0.0.1/src/routes
[SvelteKit routing]: https://svelte.dev/docs/kit/routing
[error page]:
    https://github.com/systemcarl/blank/blob/v0.0.1/src/routes/+error.svelte
[test file]:
    https://github.com/systemcarl/blank/blob/v0.0.1/src/routes/error.svelte.test.ts

