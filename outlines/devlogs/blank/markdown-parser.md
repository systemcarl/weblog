# Abstract
- I have turned my personal website into a weblog.
- The first version of the blogging service I am developing supports converting
    static Markdown content into styled HTML.
- Implementing a Markdown parser required considering how to best integrate
    the theming system with the content structure.

# DevLog: Implementing a Weblog Markdown Parser
- I'm parsing weblog content stored in a [Markdown].
    - To centralize my content, all my weblog articles are stored in a [Git
        repository].
- Recently, I created a [personal website] where I wanted to host my articles.
    - Rendering the Markdown on the website will provide a more polished and
        unified reading experience.
    - Integrating the content into the application will also allow me to add
        features in the future.

## Parsing the Options
- There are many existing Markdown parsers available.
    - There's no need to create my own parser.
    - I just needed to find one that met my needs:
        - convert common Markdown to HTML,
        - render Markdown footnotes,
        - and supports adding conditional HTML classes for styling.
    - I chose to use ["Markdown it!"] because it was a popular choice that met
        the requirements.
- There might be a solution that doesn't require adding custom [HTML classes].
    - However, supporting a dynamic theme system prevents simply using a tag
        name for styling: e.g., `.article h1`.
    - To the best of my knowledge, there is no way to apply the CSS
        styles of one selector to another.
- This was very straightforward to implement with "Markdown it!".
    - Aside from limitations with the [footnote plugin], it only required a few
       lines of code to map tokens to typography classes.

### I Do Declare
- Since I had chosen to make the [theme styles available globally as plain CSS],
    there was no need to implement any theme-specific logic.
    - Had I used scoped styles or CSS in Javascript, this would have been more
        complicated.
- This is a great example of how [declarative design] can simplify development.
    - Simply mapping elements to style classes leaves the styling logic to the
        [CSS engine].
    - Since the CSS classes are just semantic tokens withing the theming system,
        the theme CSS generator takes care of the rest.

## Not-So-Hyper Text
- One question that always comes up when designing a content system is how to
    to format the content data.
    - The content needs to stored and transmitted in a consistent format.
    - If that format changes with new features, all existing content may need to
        be updated.
- There's nothing more simple than plain text.
    - Storing content as plain text defines a stable format that will not need
        to change.
    - With [Markdown], plain text can be converted to a rich text format for
        display.
    - If the Markdown syntax changes, the content can still be displayed as
       plain text with minimal disruption.

### Text is Plain
- This raises the question: how will I integrate more features?
    - *e.g.* embedding files or allowing comments.
- The obvious solution is to extend the Markdown syntax.
    - Adding custom link types or code blocks can provide a way to embed
        additional content
    - If done carefully, the parsers can fall back gracefully to common Markdown
        syntax if the new features are not supported.
- Another option is to also allow HTML in the content.
    - Generally, Markdown parsers support inline HTML.
    - Similar to Markdown, HTML is a plain text format that can be rendered to
        rich text.
    - HTML can represent more complex content structures and interactive
        elements.
    - However, HTML is not as readable as Markdown, and does not fall back to
        plain text as gracefully.

### A Common Content Concern
- Since the [start of the project], content formatting has been a common
    concern.
    - Deciding how to [structure the theming system] required careful
        consideration of the broader content design.
    - [Building out the content template] also required thinking about how the
        content might be generalized for different use cases.
- This will continue to be an important consideration as I add more features to
    the weblog application.
    - The application still does not provide an index for sorting and listing
        articles.

[Markdown]: https://en.wikipedia.org/wiki/Markdown
[Git repository]: https://github.com/systemcarl/weblog
[personal website]: https://carledwardlyons.ca
["Markdown it!"]: https://github.com/markdown-it/markdown-it
[HTML classes]:
    https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/class
[footnote plugin]: https://github.com/markdown-it/markdown-it-footnote
[theme styles available globally as plain CSS]: ./sveltekit.md#theming
[declarative design]: https://en.wikipedia.org/wiki/Declarative_programming
[CSS engine]: https://developer.mozilla.org/en-US/docs/Glossary/Engine/Rendering
[start of the project]: ./sveltekit.md
[structure the theming system]: ./sveltekit.md#theming
[building out the content template]:
    ./content-template.md#an-agnostic-application
