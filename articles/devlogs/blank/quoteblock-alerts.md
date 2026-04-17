# DevLog: Integrating Themed Alert Blocks
If you've ever read a programming guide or any online technical documentation,
you're probably familiar with alert blocks.
These sections of text are usually highlighted with backgrounds, borders,
    other accentuating styles, or icons to draw attention to information.
> [!TIP]
This is what I mean by an alert block.

By convention, these are often created from "quote blocks"
— or [block quotations],
a paragraph of text, often indented, to indicate a special section, often a
    quote from another source.
In markdown, quote blocks are created by prefixing lines with the `>` character.
```markdown
> This is a regular old quote block.
```
> This is a regular old quote block.

Alert blocks, on the other hand, often include a label to indicate a semantic
    meaning, such as the *NOTE*, *TIP*, *IMPORTANT*, *WARNING*, or *CAUTION*
    alert types [used by GitHub].

Common Markdown renders
— like [Markdown it!], which I'm using to [render Markdown files] on my
    [personal website] —
do not support alert blocks, only basic quote blocks.
So, to get the benefits of alert blocks in my weblog articles,
I had to implement custom *Markdown it!* parsing rules to add this additional
    information to the rendered HTML.

## If It's Not Broken, Don't File a New Issue
While [implementing Markdown parsing] and also integrating
    [code syntax highlighting],
I've thought a lot about [how I might extend Markdown parsing] in ways that
    don't compromise the portability of the Markdown files themselves.
Normally, I would prefer to avoid adding any special decorators or syntax to the
    Markdown files that would not render well with parsers that don't support
    them.
When using the GitHub alert block declarations, if the parser doesn't support
    the special syntax, an unappealing `[!NOTE]` label appears at the start of
    the quote block.
```
> [!NOTE]
This is a note alert block.
```
> [!NOTE]
This is a note alert block.

However, given the ubiquity of the GitHub,
and the fact that the source Markdown files are hosted on GitHub anyway,
I decided to implement the same syntax despite this drawback.
Worst case, the alert block is still legible, and the intent is clear,
even if the presentation is not ideal.

## Don't Quote Me
Unfortunately, there aren't any existing *Markdown it!* plugins
    (none that I could find, at least)
    that implement alert block parsing.
So, it was up to me to implement the [necessary logic] to parse the quote block
    alert type declarations and convert them into something more legible.
This isn't the first parser rule I added to format my weblog articles;
I had to [add custom theme classes] to integrate the
    [application theming system] to apply text fonts and other styles
    (especially to get footnotes looking right),
and [integrate syntax highlighting] required some additional parsing logic as
    well.
However, it was a bit trickier this time.
Since *Markdown it!* divides the content during parsing,
treating opening and closing tags as their own tokens,
it was difficult to link the content to the style of the wrapping quote block.
After some close inspection of the token structure,
I was able to crawl the tokens to apply the necessary theme classes and
    rewrite the label's HTML structure.

### Staying on Theme
While finding a way to parse the alert blocks was difficult,
the real challenge was deciding how to structure the HTML to take full advantage
    of the [application theming system].
Because alerts typically have an outer accentuating styles
— highlighting the label and boundaries —
and an inner content area that needs to be consistent and legible,
I had to find a way to intuitively separate these two areas despite there being
    no hierarchical division between the label and content.

By aggressively applying theme classes to all of the quote block elements,
I could apply the alert-specific styles to the outer container and first
    paragraph element — the label.
The base alert typography styles could then be applied directly to the remaining
    content paragraph(s), assuming the rest of the quote block plain text
    (we'll see if that assumption holds up later).
```html
<blockquote class="text typography-note typography-alert-note">
  <p class="text alert typography-alert">Note</p>
  <p class="text typography-note">This is a note alert block.</p>
</blockquote>
```
> [!NOTE]
The `typography-note` class applies the base styles to all note elements via
    inheritance from the outer blockquote
and reapplied to the inner content to revert any alert-specific styles.
You can see an example of this structure in the [theme configuration] used for
    this weblog.

The one obvious limitation of this structure is that all alert-specific styles
    must provide specific contrast to the base quote block content text.
If the quote block content is dark text, every alert background must be
    sufficiently light, or vice versa.
It is possible to work around this by not defining a base style,
but that would only work if regular quote blocks (without an alert declaration)
    are have similar styles to the article body text.

## [!SUCCESS]
I may still consider adding alternative syntax for alert blocks later, that
    would make the Markdown files more portable, and render better with generic
    parsers.
There's no reason my applications renderer can't support multiple syntax
    options for the same feature,
so long as there are no conflicts.
Of course, the more features and decorators added, the more likely it will be
    to run into conflicts between the syntax for different features.

For now, though, the way the articles are rendered, is very satisfying.
With alert blocks and [code syntax highlighting] both integrated with the
    [application theme],
my more technical articles now look nicely polished and professional.
If you want to see an example, I also just wrote a short tutorial on
    [mocking Svelte 5 components] that puts all this into action.

[block quotations]: https://en.wikipedia.org/wiki/Block_quotation
[used by GitHub]: https://en.wikipedia.org/wiki/Block_quotation
[Markdown it!]: https://github.com/markdown-it/markdown-it
[render Markdown files]: ./markdown-parser.md
[personal website]: https://carledwardlyons.ca
[implementing Markdown parsing]: ./markdown-parser.md
[how I might extend Markdown parsing]: ./markdown-parser.md#text-is-plain
[application]: https://github.com/systemcarl/blank
[necessary logic]:
    https://github.com/systemcarl/blank/blob/v0.0.5/src/lib/utils/weblog.ts#L60-L101
[add custom theme classes]: ./markdown-parser.md#parsing-the-options
[application theming system]: ./sveltekit.md#theming
[integrate syntax highlighting]: ./syntax-highlighting.md#the-perfect-prefix
[theme configuration]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.5/theme.json#L241-L271
[code syntax highlighting]: ./syntax-highlighting.md
[application theme]: ./sveltekit.md#theming
[mocking Svelte 5 components]: ../../tips/svelte/svelte-5-mocks.md
