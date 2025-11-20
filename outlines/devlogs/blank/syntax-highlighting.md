# Abstract
- My personal weblog now highlights code snippet syntax.
- *highlight.js* integrates nicely with *Markdown It!* and the applications
  theming system.

# DevLog: Integrating Themed Syntax Highlighting
- Many of my articles include code snippets.
    - Proper syntax highlighting improves readability and comprehension.
- I did not initially implement syntax highlighting when
    [integrating a Markdown renderer] within my [personal website]
    [application].
    - Incremental improvements are preferred to avoid unnecessary complexity.
    - [Markdown It!] integrates well with [highlight.js].

## A Token of Appreciation
- [highlight.js] extends the HTML tokenization provided by [Markdown It!] to
    wrap code elements with style classes.
    - The [list of "scopes"] defines the common token types shared across
        programming languages.
    ```typescript
    function helloWorld() {
        const message = "Hello, world!";
        console.log(message);
    }
    ```
    ```html
    <pre>
        <code class="language-typescript">
            <span class="hljs-keyword">function</span> <span class="hljs-title function_">helloWorld</span>(<span class="hljs-params"></span>) {
            <span class="hljs-keyword">const</span> message = <span class="hljs-string">"Hello, world!"</span>;
            <span class="hljs-variable language_">console</span>.<span class="hljs-title function_">log</span>(message);
            }
        </code>
    </pre>
    ```

### The Perfect Prefix
- Highlighting is applied via predictable CSS class names.
    - Each code scope is given the class name `<prefix><scope>`.
        - The default prefix is `hljs-`.
- This is the same naming pattern used by the [application theming system].
    - Typography styles are all given respective `typography-<context>` class
        names.
    - Setting the [highlight.js prefix] to `typography-` maps the code scopes to
        theme-generated styles.
- All text elements are also given the base `text` class for resolving the theme
    CSS variables into properties.
    - [highlight.js prefix] accepts white space, so setting the prefix to
        `text typography-code-` applies both classes to each code scope element.
    - Adding the `code-block` and `typography-code` classes to the `<code>` via
        a custom *Markdown It!* extension applies base styles to all code
        blocks.
    ```html
    <pre>
        <code class="text code-block typography-code language-typescript">
            <span class="text typography-code-keyword">function</span> <span class="text typography-code-title function_">helloWorld</span>(<span class="text typography-code-params"></span>) {
            <span class="text typography-code-keyword">const</span> message = <span class="text typography-code-string">"Hello, world!"</span>;
            <span class="text typography-code-variable language_">console</span>.<span class="text typography-code-title function_">log</span>(message);
            }
        </code>
    </pre>
    ```

## Keeping Options Open
- Adding syntax highlighting to the template application requires additional
    typography settings to the [theme configuration].
    - Each code scope has its own typography style.
    - The typography styles set properties to `inherit` by default.
    - All customization is optional.
- The theme for my [personal website] provides [an example].
- [These changes] are great for maintaining a consistent visual style.
    - The result improves the readability and presentation of code snippets in
        my articles.
    - The required configuration makes the theme definitions more complex than
        anticipated.

[integrating a Markdown renderer]: ./markdown-parser.md
[personal website]: https://carledwardlyons.ca
[application]: https://github.com/systemcarl/blank
[Markdown It!]: https://github.com/markdown-it/markdown-it
[highlight.js]: https://github.com/highlightjs/highlight.js
[the application theming system]: ./sveltekit.md#theming
[list of "scopes"]:
    https://highlightjs.readthedocs.io/en/latest/css-classes-reference.html
[highlight.js prefix]:
    https://highlightjs.readthedocs.io/en/latest/api.html#configure
[theme configuration]: ./sveltekit.md#theming
[an example]:
    https://github.com/systemcarl/folio-assets/blob/3233735aa5594f3dc7c75b14fa6b2f2f11c23383/theme.json#L241-L274
[these changes]: https://github.com/systemcarl/blank/tree/v0.0.4
