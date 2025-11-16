# Abstract
- I implemented custom alert blocks rendering for my personal weblog.
- The application parses quote blocks and converts them into styled alert boxes.
    - The parser uses the same alert declarations as GitHub.

# DevLog: Integrating Themed Alert Blocks
- Alert blocks are a common way to highlight important information within
    articles.
    - This is also supported some Markdown renders,
        - *e.g.*, GitHub.
    - I added support for alert block support to my personal weblog.
- My personal weblog uses [Markdown It!] to [parse Markdown files].
    - [Markdown It!] supports quote blocks via the `>` character.
    - I implemented custom rules to parse quote blocks and convert them into
        alert blocks.

## If It's Not Broken, Don't File a New Issue
- I've put a lot of thought into [how I might extend Markdown parsing]
    within my [personal website] [application].
    - I would prefer to avoid adding decorators to the Markdown files that do
        not render well universally.
- I still chose to implement the same syntax used by GitHub.
    - The presentation in most Markdown renders in not ideal, but still legible.
    ```markdown
    > [!NOTE]
    This is a note alert block.
    ```
    > [!NOTE]
    This is a note alert block.

## Don't Quote Me
- The base [Markdown It!] parser and official plugins only support basic quote
    blocks.
    - It took some work to implement the necessary [logic].
    - [Markdown It!] divides the content into tokens during parsing.
        - The content of the quote block is not a child of the opening or
            closing quote block tokens.

### Staying on Theme
- The biggest challenge was structuring the alert blocks to use intuitive theme
    rules.
    - The [application theming system] uses CSS class names generated from the
        theme configuration file.
    - It was important not to add redundant configuration options.
- I structured the alert blocks so that the outer quote block container uses the
    typography styles for the alert label.
    - The inner content container uses it's own typography styles to be
        consistent across the page section.
    ```html
    <blockquote class="text typography-note typography-alert-note">
      <p class="text alert typography-alert">Note</p>
      <p class="text typography-note">This is a note alert block.</p>
    </blockquote>
    ```
    - The outer blockquote uses the `typography-alert-<type>` class to apply
        alert-specific styles to itself and the label.
    - The inner content paragraph uses the `typography-note` class to revert
        the note content to the base note typography styles.
    - The [theme configuration] for my [personal website] provides an example.
- This structure currently prevents notes from using different contrast
    spectra from one another.
    - If the note content is dark, all note backgrounds must be light, and vice
        versa.

## [!SUCCESS]
- I may consider adding alternative syntax for alert blocks in the future.
    - It would still be preferable to use Markdown files that render well
        universally.
- The renderer can now present all the necessary elements for simple coding
    tutorials.
    - Code block [syntax highlighting] is already implemented.
    - I recently published a tutorial for [mocking Svelte 5 components].

[Markdown It!]: https://github.com/markdown-it/markdown-it
[parsing Markdown files]: ./markdown-parser.md
[how I might extend Markdown parsing]: ./markdown-parser.md#text-is-plain
[personal website]: https://carledwardlyons.ca
[application]: https://github.com/systemcarl/blank
[logic]:
    https://github.com/systemcarl/blank/blob/v0.0.5/src/lib/utils/weblog.ts#L60-L101
[application theming system]: ./sveltekit.md#theming
[theme configuration]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.5/theme.json#L241-L271
[syntax highlighting]: ./syntax-highlighting.md
[mocking Svelte 5 components]: ../../tips/svelte/svelte-5-mocks.md
