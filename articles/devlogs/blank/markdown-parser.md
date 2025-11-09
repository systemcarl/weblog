# DevLog: Implementing a Weblog Markdown Parser
Over the past couple weeks, I've been [building a personal website] to showcase
    my projects and whatever else I want to share.
It makes sense to host a weblog on the site so I can incrementally add content
    over time that's both easy to manage and browse.
To keep the content management simple, I've chosen to store them in a
    [Git repository] separate from the website code.
This means my catalog is independent of the website [application]
    (which is actually [just a template]),
making it easy to manage with the deployed [configuration].

Since the articles files are meant to be human-readable, easy to edit, and
    meant to be universally compatible,
I wrote the content in [Markdown].
The simplest option would have been to just link to the source Markdown files,
which GitHub can render naively.
However, I wanted to provide a more polished reading experience on the site
and tie in [the application's theming].
I also plan to add more features in the future that would require integrating
    the content into the application, one way or another.

## Parsing the Options
There was no need to reinvent the wheel here; there are many established
    Markdown parsers out there.
The problem was simply finding one that met the requirements I had in mind.
The parser needed to:
- convert common Markdown syntax to HTML,
- render [Markdown footnotes],
- and include some support for adding conditional HTML classes for styling.

The most active and popular choice that met these requirements was
    ["Markdown it!"].

In most cases, it's not necessary to add custom [HTML classes] to the parsed
    elements.
Typically one could simply use tag names for styling: e.g., `.article h1`.
But, since the dynamic [theme system] needs to apply different styles based on
    additional context, this approach wouldn't work.
To the best of my knowledge, there currently isn't a runtime protocol for
    applying the [cascading style sheets (CSS)] definitions of one selector to
    another — something like [Sass]'s `@extend` directive.
```css
.article h1 {
    @extend .typography-heading-1;
}
```

In the end, it was easy enough to implement this with "Markdown it!".
The render interface exposes the tokenized elements during parsing,
    allowing class names to be added as needed.
The only exception was the [footnote plugin], which required a [small patch] to
    apply styling correctly to the footnote references
    (the superscript numbers inline with the text).[^footnote]

### Thinking Abstractly
One additional thing I needed to consider when rendering an article page was
    the metadata that browsers, search engines, and other services expect when
    displaying pages or links.
Basic [search engine optimization (SEO)] requires a page [title] and a
    summary [description] to be included in the HTML document.
To handle this, I created a separate directory of article abstracts that mirrors
    the article directory structure.
Each [abstract file] includes the title and description for the corresponding
    [article file].
Pulling this optional data into a separate file keeps the article content to the
    article itself,
and the abstract Markdown file can also rendered to provide a simple preview of
    the article content.

### I Do Declare
There wan't much else to it, really.
Since I the [theme styles are globally available as plain CSS], there was no
    need to implement any theme-specific logic in the parser.
Had I used scoped styles (e.g., [CSS Modules], Svelte
    [component style definitions])
    or CSS in Javascript (e.g., [styled-components], [emotion]),
this wouldn't have been as simple.
The article component would have needed to be aware of the current theme and
    dynamically apply the correct styles to match the current theme.

While I have [criticized declarative models] in the past, this is a great
    example of how [declarative design] can significantly simplify
    implementation.
Simply mapping elements to style classes leaves the styling logic to the
    [CSS engine], allowing us to focus on the composition and structure of the
    content.
And since the CSS classes the application uses are just semantic tokens within
    [the theming system],
the [theme CSS generator] takes care of the rest naturally.

## Not-So-Hyper Text
In terms of code, [this update] was quite trivial.
However, a lot had to be considered beyond simply choosing the right parser
    library.
When designing a content system, one of the core questions that must be
    addressed is: how to format the content data?
With any system that's not fully self-contained, data needs to be stored and
    transmitted in a consistent format.
If that format changes with new features, all existing content may need to be
    updated to match
— which can be challenging to manage, even if it's just emptying the browser
    cache of a few users.
Your data structure is always coupled with the software that consumes it like
    this, in some way.[^softyy]
And since I chose to store my articles outside the application repository,
    I needed to really think about how I structured the content.

So, the idea was to keep things as simple as possible,
and there's nothing simpler than plain text.
Markdown isn't much more than plain text, and the added syntax is as close to
   plain text styling as you can get.
At a glance, a Markdown document just looks like an old text file from before
    rich text editors were commonplace
and documents were broken up with dashes and pounds signs.
This means that the content will remain quite stable and accessible, regardless
    of how Markdown specifications change over time.
And with some added validation in the application, it shouldn't be too difficult
    to ensure that the content meets the expected structure,
as long as it's easily decoded text (which is really anything transferred over
    HTTP).

### Text is Plain
But this does raise another question: how will more features be added in the
    future?
For example, one of my goals is to allow embedding images and other media files
    directly in the articles.
It would be really great to feed in code files and design documents directly
    from the source files.
Since this would require merging content from multiple sources, and likely
    different formats,
this wouldn't be appropriate to handle with a Markdown parser alone.

The simplest solution to consider would be to extend the Markdown syntax with
    custom directives that the parser could interpret
and pass back the appropriate command to the application for embedding the
    content appropriately.
```markdown
On GitHub, you can just include a link to raw files directly, like so:
https://github.com/systemcarl/weblog/blob/2bd7694f413c43640033e09db42eca7aad80b58d/articles/hello-world.md?plain=1#L2
```
> On GitHub, you can just include a link to raw files directly, like so:
> https://github.com/systemcarl/weblog/blob/2bd7694f413c43640033e09db42eca7aad80b58d/articles/hello-world.md?plain=1#L2

This only works if you're viewing the content in an environment that understands
    that github URLs link to a file resource.
Any other parser would just leave the directives as-is in the output HTML as
    unintelligible text.
This is not ideal, but there could be ways to mitigate this problem with
    cleverly designed link types or code blocks that would still fall back
    so something reasonable if the new features were not supported.
```markdown
Including a query parameter could signal that the link should be embedded:
[Example](https://github.com/systemcarl/weblog/blob/2bd7694f413c43640033e09db42eca7aad80b58d/articles/hello-world.md?embed=1&plain=1#L2)
```
> Including a query parameter could signal that the link should be embedded:
> [Example](https://github.com/systemcarl/weblog/blob/2bd7694f413c43640033e09db42eca7aad80b58d/articles/hello-world.md?embed=1&plain=1#L2)

When parsed with any standard Markdown parser, this would just render as a
    normal link, that directs to the file on GitHub
(so long as the `embed` parameter is irrelevant to GitHub).
The downside is that when [previewing the content on GitHub], you no longer see
    embedded content.

Another option would be to allow HTML in the content.
This is generally supported by most Markdown parsers.
Common HTML elements can be used directly within the Markdown content,
    allowing for more complex structures to be included in the document.
And since HTML is also just a plain text format,
it fits well within the design's principles of simplicity and stability.
However, HTML is not nearly as nice to read and thus requires a more involved
    HTML renderer to present the content in an acceptable way.
But if used sparingly within a Markdown document, this could still work well as
    a compromise between readability and functionality.

### A Common Content Concern
Since the [start of the project], content formatting has been a common concern.
This simple but critical element of the design is a key decision that will
    impact the overall structure and experience of the application.
The [structure of your data shapes the outcome] of project, well beyond the
    current iteration of the software.
Deciding how to [structure the theming system] also required carefully
    considering not just my initial design, but how the design might evolve
    over time, with more features and different themes.
In the same way, [building out the content template] required thinking about how
    the content might be structured to handle different use cases beyond just
    my own [personal website].

I expect this will continue to be a theme (philosophical, not visual) as this
    project continues.
As more features are added to the underlying application, the constraints will
    continue to tighten around how the content must be structured.
The application still needs to provide a way to browse, filter, and sort
    articles
— which will likely require some additional metadata to be included with each
    article.
But at this point, keeping the system simple and flexible was the best way to
    prevent the destiny of the application from being determined by today's
    assumptions.

[^footnote]: This is a footnote, here!
This definition links back to the text where it was referenced.

[^softyy]: Discussion of the content files coupling to the applications
    implementation
came from feedback provided by fellow software developer, [Chris Adkins].

[building a personal website]: ./sveltekit.md
[Git repository]: https://github.com/systemcarl/weblog
[application]: https://github.com/systemcarl/blank
[just a template]: ./sveltekit
[configuration]: ./sveltekit#knobs-and-dials
[Markdown]: https://en.wikipedia.org/wiki/Markdown
[the application's theming]: ./sveltekit#theming
[Markdown footnotes]:
    https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#footnotes
["Markdown it!"]: https://github.com/markdown-it/markdown-it
[HTML classes]:
    https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/class
[theme system]: ./sveltekit#theming
[cascading style sheets (CSS)]:
    https://en.wikipedia.org/wiki/Cascading_Style_Sheets
[Sass]: https://sass-lang.com/
[footnote plugin]: https://github.com/markdown-it/markdown-it-footnote
[small patch]:
    https://github.com/systemcarl/blank/blob/6db0c4fde3a3a966937dae62b76cd0bd0b4022c5/src/lib/utils/weblog.ts#L35-L49
[search engine optimization (SEO)]:
    https://en.wikipedia.org/wiki/Search_engine_optimization
[title]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/title
[description]:
    https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta/name#description
[abstract file]:
    https://github.com/systemcarl/weblog/blob/2bd7694f413c43640033e09db42eca7aad80b58d/abstracts/hello-world.md
[article file]:
    https://github.com/systemcarl/weblog/blob/2bd7694f413c43640033e09db42eca7aad80b58d/articles/hello-world.md
[theme styles are globally available as plain CSS]: ./sveltekit.md#theming
[CSS Modules]: https://github.com/css-modules/css-modules
[component style definitions]: https://svelte.dev/docs/svelte/svelte-files#style
[styled-components]: https://styled-components.com/
[emotion]: https://emotion.sh/docs/introduction
[criticized declarative models]: ./ci-cd#too-good-to-be-simple
[declarative design]: https://en.wikipedia.org/wiki/Declarative_programming
[CSS engine]: https://developer.mozilla.org/en-US/docs/Glossary/Engine/Rendering
[the theming system]: ./sveltekit.md#theming
[theme CSS generator]:
    https://github.com/systemcarl/blank/blob/v0.0.3/src/lib/utils/styles.ts
[this update]: https://github.com/systemcarl/blank/tree/v0.0.3
[previewing the content on GitHub]:
    https://github.com/systemcarl/weblog/blob/main/articles/devlogs/blank/markdown-parser.md#text-is-plain
[start of the project]: ./sveltekit.md
[structure of your data shapes the outcome]:
    https://notes.mtb.xyz/p/your-data-model-is-your-destiny
[structure the theming system]: ./sveltekit.md#theming
[building out the content template]:
    ./content-template.md#an-agnostic-application
[personal website]: https://carledwardlyons.ca
[Chris Adkins]: https://www.cjadkins.com/
