# Mocking Svelte 5 Components
> [!NOTE]
Before diving into the tips and tricks for mocking Svelte components,
it's important to understand why component isolation is difficult in Svelte,
and whether you should be trying to work around these limitations in the first
    place.
>
> But, if you do want to get straight to the code,
start with [How You're Supposed to Test Svelte Components] for some simple
    recommended approaches,
or skip to [Other Tricks] to see some less conventional techniques and
    workarounds.
Don't forget to check out the [example project], too.

One of the best things about component-based frameworks that they help to break
    apart complex UIs into smaller, manageable pieces.
Isolating functional units and repeated UI elements into their own components
    helps to separate concerns and overall makes maintaining the codebase
    easier.
Testing each component, independently from the rest of the application, keeps
    tests focused on the component being tested.
Relying on other components to be present and working as expected can raise
    false negatives, misattribute failures, and obscure unintended side effects.

Many applications do not necessarily require such thorough testing
(and arguably, if you're using Svelte, that's most likely the case),
but large applications with intricate state management and complex interactions
    benefit greatly from isolated component tests.
To achieve this isolation, component dependencies — sub-components, or
    "child" components — need to be replaced with mocked implementations during
    tests.
This can be to replace an unrelated component with a "stubbed" placeholder, or
    to "spy" on a component to verify that the real component was invoked
    correctly.

Sadly, Svelte 5 does not provide much in the way of testing utilities.
The documentation for [testing Svelte applications] and the
    [Svelte Testing Library] provide only the most basic test cases
with no guidance on more advanced testing strategies.
This is likely because Svelte encourages developers to write simple,
    self-contained components that do not require — nor allow for —
    complex mocking strategies.
However, through some experimentation, I have collected several techniques
    that can be used to setup component tests, if needed.
These approaches rely heavily on poorly documented Svelte APIs
that could very well change, even in minor releases
 — so use them thoughtfully and with caution.

In case any of this is difficult to follow, the API has changed, or you just
    don't believe me,
I've created an [example project] that demonstrates each of these test cases
    in more detail, and preserve the intended testing outcomes.

## How You're Supposed to Test Svelte Components
Before you start ripping apart our applications with component hacks,
    it's worth understanding how Svelte expects components to be tested.
Svelte encourages developers to write simple, self-contained components
    that do not rely on complex interactions with other components.
Each component has it's own `.svelte` file;
components should manage their own state and behavior.
It's expected that components interact through well-defined props and events
    that are tightly coupled to the component's purpose.
This design philosophy reduces the need for complex mocking strategies,
    as components can be tested in isolation without implicitly relying on the
    behavior of other components.

Since each component is self-contained in it's own file,
    this means that the component test must import the component, as is.
With [Svelte Testing Library] (the recommended testing utility for Svelte),
    rendering a component in isolation does not allow any further composition.
Only props can be passed to the component under test;
child components are rendered as-is, with no way to intercept or replace them.
If a child component is required, the parent component cannot be tested directly
    on it's own.
```typescript
<!-- ComponentUnderTest.test.svelte -->
import { test } from 'vitest';
import { render } from '@testing-library/svelte';
import ComponentUnderTest from './ComponentUnderTest.svelte';
import Dependency from './Dependency.svelte';

test('ComponentUnderTest renders correctly', () => {
  render(ComponentUnderTest, {
    props : {
      property : 'value',
   // children : Dependency, // Doesn't work
   // children : () => Dependency, // Still doesn't work
      },
    });

  // assertions...
});
```

### Test Case Files
There is one easy, Svelte-friendly way to handle required child components:
test case components.
Additional svelte components files, exclusively for testing, can be created
    as separate `.svelte` files
— I prefer to name them with a `.test.svelte` suffix, by convention.
These test case components can import the component under test,
    and provide any required child components, including child props and events,
    as needed.
This allows the test case component to fully compose the component tree,
    while still isolating the component under test.
```typescript
<!-- ComponentUnderTest.test.svelte -->
<script lang="ts">
  import Dependency from './Dependency.svelte';
  import ComponentUnderTest from './ComponentUnderTest.svelte';

  const { property } : { property ?: string } = $props();
</script>

<ComponentUnderTest property={property}>
  <Dependency />
</ComponentUnderTest>
```
The test case component can then be imported and rendered in the test file,
    in lieu of the component under test.
```typescript
// ComponentUnderTest.svelte.test.ts
import { test } from 'vitest';
import { render } from '@testing-library/svelte';
import ComponentUnderTestWithDependency from
  './ComponentUnderTest.test.svelte';

test('ComponentUnderTest renders dependency', () => {
  render(ComponentUnderTestWithDependency, {
    props : {
      property : 'value',
    },
  });

  // assertions...
});
```

This also allows for the child components to be mocked or spied on,
    by stubbing the child component in place.
```typescript
<!-- ComponentUnderTestWithDependency.test.svelte -->
<script lang="ts">
  import ComponentUnderTest from './ComponentUnderTest.svelte';
</script>

<ComponentUnderTest>
  <!-- Stubbed Dependency component -->
  <div data-testid="mocked-dependency">Mocked Dependency</div>
</ComponentUnderTest>
```

## Other Tricks
If you're like me, you may find the above approach a bit tedious and very
    limiting;
if you're trying to build complex user interfaces, pages with many nested
    components,
or just try to implement a fine-grained composition design pattern,
the limitations of Svelte's component model can be frustrating.
To work around these limitations, I've found a few tricks that can sidestep some
    of Svelte's restrictions to allow for more flexible testing strategies.
Using low-level Svelte APIs, we can can achieve dynamic component composition,
    high-level component mocking, and simple prop spying.
> [!CAUTION]
These approaches rely on APIs that are not well-documented and may likely
    change in future Svelte releases.

### Generating Snippets
The most basic means of creating test components at runtime is to generate a
    Svelte snippet.
[Snippets] are new to Svelte 5, and allow for dynamic component definitions
    using raw HTML and a callback to mount and unmount static components.
Compared to the other suggestions here, snippets are the most recommended way to
    inject dependencies,
but still, it's not really documented at all.
The [`createRawSnippet` API] can be used to create a test component that can
    be rendered as a child component using Svelte Testing Library.
```typescript
import { test } from 'vitest';
import { render } from '@testing-library/svelte';
import { createRawSnippet } from 'svelte';
import ComponentUnderTest from './ComponentUnderTest.svelte';

test('ComponentUnderTest with mocked dependency', () => {
  const MockedDependency = createRawSnippet(() => ({
    // define some simple HTML for the mocked component
    render : () => `<div>Mocked Dependency</div>`,
  });

  render(ComponentUnderTest, {
    props : {
      children : MockedDependency,
    },
  });

  // assertions...
});
```
To include an existing component within the snippet, the `setup` callback
    can be used to mount the component to a root HTML node.
The [mount] and [unmount] APIs can be called to manage the component lifecycle.
The `setup` callback should return a cleanup function that unmounts the
    component when the snippet is destroyed.
Because a root HTML element is required, this means that the mounted component
    must be wrapped in a container element.
If structural issues arise from this extra element, the style property
    [`display: content`] can be used to effectively remove the container
    from the layout.
However, this does not remove the element from the DOM and it may still
    interfere CSS selectors that rely a direct parent-child relationship.
If this is a concern, consider [directly mounting components] instead.
```typescript
import { test } from 'vitest';
import { render } from '@testing-library/svelte';
import { createRawSnippet, mount, unmount } from 'svelte';
import ComponentUnderTest from './ComponentUnderTest.svelte';
import Dependency from './Dependency.svelte';

test('ComponentUnderTest with mocked dependency', () => {
  const MockedDependency = createRawSnippet(() => ({
    render : () => '<div style="display: contents;"></div>',
    setup : (node) => {
      const mounted = mount(Dependency, { target : node });
      return () => unmount(mounted);
    },
  }));

  render(ComponentUnderTest, {
    props : {
      children : MockedDependency,
    },
  });

  // assertions...
});
```

### Directly Mounting Components
If the goal is to simply mount a component within a specific layout,
creating a full snippet may not be necessary.
The [mount] API can be used directly to insert a component into the DOM,
    without needing to create a snippet wrapper.
This is useful for setting up complex component hierarchies in tests, including
    rendering a parent component with multiple child components.
```typescript
<!-- ComponentUnderTest.svelte -->
<script lang="ts">
  import { Snippet } from 'svelte';
  const { children } : { children ?: Snippet } = $props();
</script>

<div data-testid="component-under-test">
  {@render children?.()}
</div>
```
```typescript
import { test } from 'vitest';\
import { render, screen } from '@testing-library/svelte';
import { mount, unmount } from 'svelte';
import ComponentUnderTest from './ComponentUnderTest.svelte';

const mock = createRawSnippet(() => ({
  render : () => '<div>Mocked Dependency</div>',
}));

test('ComponentUnderTest with mocked dependency', () => {
  render(ComponentUnderTest);
  const container = screen
    .getByTestId('component-under-test') as HTMLElement;
  const mounted = mount(mock, { target : container });

  // assertions...

  unmount(mounted);
});
```
Of course, this assumes that reference to the container element
    (not the component root) is readily available,
which may not always be the case.

### Vitest Module Mocks
Svelte uses [Vitest] as it's core testing framework, so Vitest's
    [module mocking] capabilities can always be used to replace Svelte
    components.
However, since Svelte components are compiled into JavaScript modules,
    the mock must return a valid Svelte component snippet to render correctly.
This can be done by [generating a snippet] within the mock definition.
> [!IMPORTANT]
Vitest [hoists module mocks] to the top of the file, so any imports, including
    `createRawSnippet` will not be available when the mock is defined.
To work around this, use a dynamic import within the mock definition.
```typescript
<!-- ComponentUnderTest.svelte -->
<script lang="ts">
  import Dependency from './Dependency.svelte';
</script>

<div>
  <Dependency />
</div>
```
```typescript
import { test, vi } from 'vitest';
import { render } from '@testing-library/svelte';
import ComponentUnderTest from './ComponentUnderTest.svelte';
import Dependency from './Dependency.svelte';

vi.mock('./Dependency.svelte', async () => {
  // dynamic import to avoid missing initialization after vitest hoisting
  const { createRawSnippet } = await import('svelte');
  return {
    default : createRawSnippet(() => ({
      render : () => '<div>Property: mocked value</div>',
    })),
  };
});

test('ComponentUnderTest with mocked dependency', () => {
  render(ComponentUnderTest);

  // assertions...
});
```

#### Spying on Svelte Components
If you sill want to maintain the original component implementation, and simply
    intercept the props passed to the component to verify correct invocation,
`vi.mock` provides an option to spy on the entire module without any fuss.
```
vi.mock('./Dependency.svelte', { spy : true });
```
This will replace the component with a spy that records all calls to the
    component, including the props passed to it.
The original component implementation will still be used, so the component will
    render as it normally would.
Calls to the spy will contain the props passed to the component,
allowing for quick verification of the components' interaction.
> [!IMPORTANT]
It appears that the snippet is only called on mount, not an any subsequent
    updates and re-renders.
This means that the number of calls is (likely; no guarantees here) the number
    of times the component is mounted.
It also means that prop changes will not be explicitly recorded; the props
    object is mutated over the lifecycle of the component.
```typescript
<!-- ComponentUnderTest.svelte -->
<script lang="ts">
  import Dependency from './Dependency.svelte';
</script>

<div>
  <Dependency property="expectedValue" />
</div>
```
```typescript
import { test, vi } from 'vitest';
import { render } from '@testing-library/svelte';
import ComponentUnderTest from './ComponentUnderTest.svelte';
import Dependency from './Dependency.svelte';

// use the original module implementation and spy on it
vi.mock('./Dependency.svelte', { spy : true });

test('ComponentUnderTest renders Dependency', () => {
  render(ComponentUnderTest);

  expect(Dependency).toHaveBeenCalledExactlyOnce();
  expect(Dependency).toHaveBeenCalledWith(
    expect.anything(),
    expect.objectContaining({
      property : 'expectedValue',
    }),
  );
});
```

If you really want more control over the mock implementation,
a spy can be manually created by wrapping a [generated snippet] in [`vi.fn()`].
This can allow for a range of implementations, from simple stubs to prop
    substitution.
```typescript
import { test, vi } from 'vitest';
import { render } from '@testing-library/svelte';
import ComponentUnderTest from './ComponentUnderTest.svelte';
import Dependency from './Dependency.svelte';

vi.mock('./Dependency.svelte', async (originalModule) => {
// dynamic import to avoid missing initialization after vitest hoisting
const { createRawSnippet, mount, unmount } = await import('svelte');
// load the original component to wrap be wrapped
const originalComponent = (await originalModule()).default;
const spy = vi.fn(createRawSnippet(props => ({
    // props can be used to customize the mock behavior
    render : () => `<div data-testid="${props.testId}"></div>`,
    setup : (node) => {
    const mounted = mount(originalComponent, {
        target : node,
        // props can be overridden to mock specific values
        props : { ...props, property : 'mocked value' },
    });
    return () => unmount(mounted);
    },
})));

return { default : spy };
});

test('ComponentUnderTest with mocked dependency', () => {
  render(ComponentUnderTest);

  // assertions...
});
```

## Considerations
Many of the above examples rely on APIs that are not necessarily intended for
    these use cases,
and changes to Svelte — even minor — could easily break these tests.
You should weigh the benefits of isolated component tests against the risk of
    building upon experimental and unsupported patterns.

It's important to remember that Svelte's design philosophy encourages simple,
    self-contained components that do not require complex mocking strategies.
Before implementing any of these techniques, consider whether your components
    can be simplified or refactored to reduce dependencies.
It may also be worth reconsidering whether Svelte is the right framework for
    your application.
It's also possible that Svelte can be combined with other libraries or
    frameworks that provide more control over component composition, lifecycle,
    and testing.
Ultimately, the best approach will depend on the specific needs and constraints
    of the application.

In addition to the [example project], I've also used all of these techniques
    in [the application powering this weblog]
— where I developed these workarounds.
I documented the process of developing the application, and all it's
    motivations, in my development logs.
You can read more about my [approach to testing a SvelteKit application] there.
To mitigate the risks of using these unsupported workarounds,
    I bundled some of these techniques into a set of [testing utilities]
that might also be interesting to anyone pursuing more advanced Svelte testing
    strategies.

[How You're Supposed to Test Svelte Components]:
    #how-youre-supposed-to-test-svelte-components
[Other Tricks]: #other-tricks
[example project]: https://github.com/systemcarl/example-svelte-5-mocks
[testing Svelte applications]: https://svelte.dev/docs/testing
[Svelte Testing Library]:
    https://testing-library.com/docs/svelte-testing-library/intro/
[Snippets]: https://svelte.dev/docs/svelte/snippet
[`createRawSnippet` API]: https://svelte.dev/docs/svelte/svelte#createRawSnippet
[mount]: https://svelte.dev/docs/svelte/svelte#mount
[unmount]: https://svelte.dev/docs/svelte/svelte#unmount
[`display: content`]:
    https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/display#display_contents
[directly mounting components]: #directly-mounting-components
[Vitest]: https://vitest.dev/
[module mocking]: https://vitest.dev/guide/mocking.html#module-mocking
[generating a snippet]: #generating-snippets
[hoists module mocks]:
    https://vitest.dev/guide/mocking.html#mock-an-exported-function
[generated snippet]: #generating-snippets
[`vi.fn()`]: https://vitest.dev/api/vi.html#vinfn
[the application powering this weblog]: https://github.com/systemcarl/blank
[approach to testing a SvelteKit application]:
    ./../../devlogs/blank/sveltekit.md#component-testing
[testing utilities]:
    https://github.com/systemcarl/blank/blob/v0.0.4/src/lib/tests/component.ts
