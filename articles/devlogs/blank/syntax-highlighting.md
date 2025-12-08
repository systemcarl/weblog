# DevLog: Integrating Themed Syntax Highlighting
Being a software developer, I often augment my articles with code in some
    form or another.
When this involves rendering the odd line of code,
    a simple monospaced font can be enough.
But when the code blocks get longer and the language context is important,
formatting matters.
Often referred to as ["syntax highlighting,"]
language-specific keywords, tokens, and variables are visually differentiated
    using various typographic styles,
like colour, weight, and emphasis.
Today, syntax highlighting is the norm for code presentation in the code editor,
    and on the web.

To keep the development process of [my personal website] streamlined,
I opted not to implement syntax highlighting when
    [I first integrated a Markdown renderer].
Initially, [this web application] simply used the default [Markdown it!]
    renderer output without any additional processing.
Fortunately, *Markdown it!* integrates well with [highlight.js], a quaintly
    simple syntax highlighting library.

## A Token of Appreciation
The [highlight.js] library can be used as a plugin for [Markdown it!] to extend
    the HTML tokenization process.
When code blocks are rendered, highlight.js wraps the code elements with
    specific style classes based on symbolic code token types.


These tokens are defined in the highlight.js [list of "scopes"], which outlines
    the common semantic code elements shared across programming languages.
The syntax highlighter automatically assigns the common scopes based on the
    grammar of the specified programming language
(so long as it [is supported]).
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
Conveniently, [highlight.js] applies syntax highlighting styles to the code
    tokens via predictable CSS class names.
The class name for each code scope follows a predictable pattern of
    `<prefix><scope>`,
where the default prefix is `hljs-`
(*e.g.*, `hljs-keyword`, `hljs-string`, *etc.*).

This naming pattern is very similar to [the pattern I used to apply theme styles]
    throughout my application.
To specifically target text elements, the application automatically generates
    typography style classes with the format `typography-<context>`,
where `<context>` is the semantic usage of the typography style
(*e.g.*, `typography-body`, `typography-title`, *etc.*).
By setting the [highlight.js prefix] to `typography-`,
the code scopes are automatically mapped to the corresponding theme-generated
    typography styles, using the highlight.js scope as the theme context.
And just like that, the theme-defined typography styles are applied to the
    applicable code tokens.

The rest of the implementation is just to address the base styles for code
    blocks, and text elements in general.
All text elements throughout the application are also given the base `text`
    class for resolving the theme CSS variables into finalized style properties
(this allows classes to provide context without necessarily applying styles
    directly).
Since the [highlight.js prefix] accepts white space without complaint,
    setting the prefix to `text typography-code-` injects the extra `text` class
    alongside the typography styles for each code scope element.
Additionally, adding the `code-block` and `typography-code` classes to the
    wrapping `<code>` element via a custom *Markdown it!* extension applies base
    styles to all code blocks.
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
Now these code blocks can be styled by simply defining the right typography
    definitions for each theme, or even for each theme section to allow for
    light and dark variations, depending on the background.

## Keeping Options Open
Over and over, the fundamental questions concerning the application's design
    come back to [balancing the complexity of features with ease of use].
To fully enable the syntax highlighting feature, several new typography
    definitions need to be added to
    [the theme configuration used to style this website].
Each code scope requires its own typography style definition, even if the
    typography styles are shared across multiple scopes.
Fortunately, I had already designed the typography styles to set properties to
    `inherit` by default, allowing new styles to only apply a subset of
    properties, as needed.
This means that every code scope does not need to define the font family to the
    same monospace font
because the base `typography-code` style has already taken care of that.
It also means that every scope is optional; the theme only needs to define the
    typography of the scopes that need to be highlighted.

I've already added this syntax highlighting support to [my personal website]
(and this blog, as you've probably noticed).
You can see [an example of the theme configuration] on GitHub. This file defines
    the typography styles for the code you see here.

Once the logic was in place, it was a lot of fun to experiment with different
    color schemes and typographic styles for the various code scopes.
I haven't yet decided on additional font options for the theme, but it would
   be interesting to use alternative monospace fonts for certain tokens
— like many developers do in their code editors, using cursive fonts for
    comments for example.
But with [this new application version] and some quick adjustments to the theme
    configuration file,
the code blocks are already much easier to read and look much more consistent
    with the overall visual style of the website.
However, the theme file is already much larger than I had originally
    anticipated,
and there's always the risk of bloating the generated CSS, reducing overall page
    performance.
So, in the future, I may need to revisit how themes are defined to better manage
    the increasing complexity of the typography styles.

["syntax highlighting,"]: https://en.wikipedia.org/wiki/Syntax_highlighting
[I first integrated a Markdown renderer]: ./markdown-parser.md
[my personal website]: https://carledwardlyons.ca
[this web application]: https://github.com/systemcarl/blank
[Markdown it!]: https://github.com/markdown-it/markdown-it
[highlight.js]: https://github.com/highlightjs/highlight.js
[list of "scopes"]:
    https://highlightjs.readthedocs.io/en/latest/css-classes-reference.html
[is supported]:
    https://highlightjs.readthedocs.io/en/latest/supported-languages.html
[the pattern I used to apply theme styles]: ./sveltekit.md#theming
[highlight.js prefix]:
    https://highlightjs.readthedocs.io/en/latest/api.html#configure
[balancing the complexity of features with ease of use]:
    ./content-template.md#strong-opinions-loosely-held
[the theme configuration used to style this website]: ./sveltekit.md#theming
[an example of the theme configuration]:
    https://github.com/systemcarl/folio-assets/blob/3233735aa5594f3dc7c75b14fa6b2f2f11c23383/theme.json#L241-L274
[this new application version]: https://github.com/systemcarl/blank/tree/v0.0.4
