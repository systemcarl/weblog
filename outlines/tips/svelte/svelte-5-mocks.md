# Abstract
- Finding effective techniques for testing Svelte components can be difficult.
- There is little documentation on Svelte testing tools and techniques.
- I have found several approaches for manipulating Svelte components that can be
    used for dynamic composition and mocking during tests.

# Mocking Svelte 5 Components
- Component-based frameworks help isolate functional units and UI elements.
    - This isolation helps to separate concerns and improve maintainability.
    - Component-based testing allows each component to be tested in isolation.
- Good testing requires the ability to decouple components from their
    dependencies.
    - Mocking is a common technique for isolating code during any unit test.
- Svelte does not provide as much built-in support for mocking as some other
    frontend frameworks.
    - Svelte's opinionated component structure and compilation process can
        conflict with certain testing strategies.
    - I have explored several techniques for mocking Svelte components in tests.
        - Most of these techniques are unofficial workarounds and rely on
            undocumented APIs.
- All of these approaches are demonstrated in the [example repository].

## How You're Supposed to Test Svelte Components
- Svelte expects all components to be self-contained within their individual
    `.svelte` files.
    - Each component includes its own markup, styles, and logic.
    - Svelte 5 accepts a children prop for passing nested content, and named
        snippets for constructing more complex layouts.
- These components must be composed in a component file to be rendered for a
    test.
    - Using [Svelte Testing Library], it is only possible to specify the
        component under test, and its props.
    - There's no defined API for building the component tree beyond the
        component under test.
    - Children can't be passed as props, as is.
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
- Additional svelte components for testing can be created as separate `.svelte`
    files.
    - These files are often named with a `.test.svelte` suffix, by convention.
    - Each test case component defines a pre-configured component to be tested.
    - The test suite then imports the test case component for use.
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
- Dependencies can be mocked to completely isolate the component under test.
    - Requires an additional test case component for each mock variation.

## Other Tricks
- Low-level APIs can be used to manually construct component trees.
    - This can be used to reduce boilerplate when setting up test cases.
    - These APIs are not well-documented and may break in future Svelte
        releases.

### Generating Snippets
- Svelte snippets can be programmatically generated for testing.
    - [`createRawSnippet`] can be used to manually inject elements.
    ```typescript
    import { test } from 'vitest';
    import { render } from '@testing-library/svelte';
    import { createRawSnippet } from 'svelte';
    import ComponentUnderTest from './ComponentUnderTest.svelte';

    test('ComponentUnderTest with mocked dependency', () => {
      const MockedDependency = createRawSnippet(() => ({
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
- Components can be mounted within these snippets to configure complex layouts.
    - The child component must be mounted to a root HTML node.
    - Using [`display: content`] can help avoid structural issues, but can
        interfere with certain CSS selectors.
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
- Components can be directly mounted within test suites.
    - This allows children to be added to rendered components *post hoc*.
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
    - Mounting the component requires reference to the correct target node.
        - This is not always feasible.

### Vitest Module Mocks
- Vitest's module mocking can be used to replace Svelte components.
    - A valid snippet must be returned from the mock.
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
- Vitest spies can be used to monitor Svelte component usage.
    - It is possible to determine if a component was rendered, and with what
        props.
        - This is not necessarily supported and props appear to mutate.
- A simple spy can be created by mocking the component with `vi.mock`.
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
- The original component can be wrapped in a spy function for more control.
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
        render : () => '<div></div>',
        setup : (node) => {
        const mounted = mount(originalComponent, {
            target : node,
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
- None of these mocking strategies are officially supported by Svelte.
    - They may break with future Svelte releases.
- Svelte is primarily designed for templating.
    - The purpose is to compose HTML and binding data and events to them.
    - Complex application logic is often better abstracted away from the
        framework.
- I use all of these techniques to [test this personal weblog application].
    - The application also includes [testing utilities] that encapsulate these
        techniques to decouple them from test implementations.

[example repository]: https://github.com/systemcarl/example-svelte-5-mocks
[Svelte Testing Library]:
    https://testing-library.com/docs/svelte-testing-library/intro/
[`createRawSnippet`]: https://svelte.dev/docs/svelte/svelte#createRawSnippet
[`display: content`]:
    https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/display#display_contents
[test this personal weblog application]:
    ./../../devlogs/blank/sveltekit.md#component-testing
[testing utilities]:
    https://github.com/systemcarl/blank/blob/v0.0.4/src/lib/tests/component.ts
