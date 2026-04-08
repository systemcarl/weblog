# DevLog: Designing a Content Template
Not long after [I set up a SvelteKit application] for [my personal website]
    and blog,
I released and deployed [the first update].
[The original version] of the application had no content at all, only
    [a styled error message], which provided [an early proof-of-concept].
With this, the application's component framework
    and [underlying theming system] was already in place.
The update now added [a very simple landing page],
testing how well the design could be implemented and extended with the
    application structure already in place.

The basic, stylized landing page provided a brief
    personal introduction.
Working from [my design mock-up], I decided to include as little content as
    possible
(just to establish a workflow for iterating on the application design)
without leaving anything looking incomplete or otherwise needing to be reworked
    to finish the rest of the planned design.
Choosing to only implement [the profile] and [the contact information] sections,
    the goal was just to add some static content elements, including:
- my name,
- a personal summary,
- an avatar image,
- an inspirational quote,
- some personal likes and dislikes,
- and a list of contact information.

This alone is a very simple task for a web developer of any
    experience level.
However, my ambitions for this project go beyond just implementing a static
    web design.
At a minimum, I want to have a system in place to easily update and change the
    content and appearance of the website without needing to redeploy the entire
    application.
I also hope that the system(s) I'm developing for this website can eventually be
    repurposed for broader applications
— if not as a whole, then at least in parts.
To achieve this, I needed to implement a sufficiently sophisticated content
    system with a well-thought-out template design.

## Strong Opinions, Loosely Held
I had decided early to [abstract all of the page content] (including static
    elements)[^assets] away from the application code.
The idea was to remove any personalized content from the application code,
allowing for easier updates to both the application and the content.
In the context of the application powering my personal website, personalized
    content is anything not strictly necessary for the application function,
including:
- text content (both copy and long-form content),
- images and graphics,
- colours and backgrounds,
- and even layout and navigation configurations.

Abstracting the personal content away from the application code also makes it
    possible to reuse the same application code for different websites or blogs,
and fully enables localization support in the future.

The core of my application therefore needed to support this abstraction,
wherein the content is injected into the application pages when rendered.

> [!NOTE]
> In practice, this means that the design elements (*e.g.*, the page title;
    the text displayed in the browser tab) are
    defined in the application code.
The inclusion and layout of these elements are fixed, or restricted to certain
    content configurations.
However, the styled [Hypertext Markup Language (HTML)] elements rendered to the
    displayed page (*e.g.*, copy, size, font, and colour)
are all defined by the injected content text and style variables.

The core challenge when designing such a content system is balancing
    structure with flexibility.
On one hand, making the content structure very rigid takes the guesswork out of
    creating (and rendering) content,
but limits the creative possibilities for the composition as a whole.
On the other hand, a completely unstructured system allows for anything you can
    imagine,
but requires much more thought and consideration to create a final product that
    is both aesthetically pleasing and functionally effective.
Ultimately, responsibility for the design outcome needs to be either given to
    the content creator, or reserved by the content system itself;
    the responsibility cannot simply be shared.

### It Always Comes Back to Marketing
The solution to this problem came by considering the application's purpose.
My motivation for building this personal website comes down to (shameless)
    self-promotion.
I could instead post my articles to an established
    platform, simply to put the core information (the matter-of-fact tips
    and guides) out there.
But that doesn't really capture [my mission] to create an identity and brand
    that other people in the community can relate to and follow.

As I've mentioned, I also have a goal to reuse my application code for others'
    websites or blogs.
To approach this,
    I found it helpful to determine the purpose of the content design, not just
    the subject matter (me).
It occurred to me that, in the same way I am promoting myself,
    any content driven application is trying to market something.
So, to generalize the content design of my template effectively,
    the goal is to make a marketing template that is tailored specifically for
    personal profiles.

As such, I focused on ease of content creation (for both myself and others)
    with a strongly opinionated content design
(opinionated in the sense that the page structure and content elements are
    predetermined,
and the template constraints must be respected).
Limiting the degree of freedom in the content design makes it simpler to create
    content
— the fewer knobs and dials to worry about, the quicker the presentation can be
    customized and published.
Structuring the content design around only a few options and variables also
    makes it easier to ensure all the configurations play well together.

> [!NOTE]
> Opinionated does not mean inflexible.
With the flexibility built into [the underlying application's theming system],
which includes theme-specific, palette-colourized SVG graphics,
there's still plenty of room for personalization and creativity within the
     design constraints.

### Personal Preferences
To make the content design as easy to use as possible,
all of the content configurations are optional.
Technically, to set up the application template, you don't even need to provide
    a page title
(though it would hardly make sense to use the placeholder title for a real
    webpage).
Beyond that innate requirement, I designed the content and theme together so
    that regardless of the amount of configuration provided,
the design would still look complete.

Furthermore, reflecting on the purpose of each design element from a marketing
    perspective
helped to simplify and clarify each element's intended effect towards the
    overall design
(*e.g.*, establishing identity, credibility, or connection with the audience).
For example, an avatar image and logo both serve to establish visual identity.
Therefore, the presentation of the avatar element should emphasize this
    function,
making it prominent and visually distinct from the rest of the content.
The end result is a cohesive design where my cartooned face could easily be
    swapped out for a brand logo without disrupting the design integrity.

Since the application is primarily concerned with visual presentation[^comment],
the constraints of the design are limited mainly to the layout.
So, while the application dictates where each content element goes on the page,
it does not impose any semantic meaning on the content itself.
For example, the section in [my design mock-up] listing my "likes" and
    "dislikes" do not need to be used for this purpose at all.
Structurally, this element is simply a pair of lists that can be used to
    represent any kind of contrast, comparison, or any arbitrary dichotomy.
This visual component could also be used to list pros and cons about
    a particular product, service, or philosophy.
I could have just as easily listed what I believe is, or is not, a sandwich.

## An Agnostic Application
Despite the strong opinions (the rules and constraints) built into the content
    design, the resulting application is still agnostic to the content itself
— unconcerned with the specific content being rendered to the page, and only
    concerned with how to render it.
That is, all of the content is still
    [aggregated by the application from static JSON files] after the application
    has been built and deployed,
as per the original design of the application.
The difference with this update is the files now include
    [an additional configuration object] to control the composition of pages,
beyond the copy and HTML style definitions.[^assets]

> [!NOTE]
>It was a choice of personal preference to not combine all the configurations
    into a single file,
even though the segregated files comprising the configuration are strictly
    dependent on one another.
Despite spanning multiple files, I still consider these files, as a collection,
    to be a single source of truth for the website's content and presentation.
Breaking it down just makes it easier to navigate and manage changes.

While some might disagree, I have found that the abstracted nature of the
    application components has made development and testing much easier.
Removing hardcoded text and other data into component properties and context
    stores
requires values to be injected into components during testing,
making the intent of the test more explicit and the expected outcomes more
clear.
This prevents changes to the content or style of the final application from
    affecting the setup or expected outcomes of the tests.
So, [all of my previously implemented testing strategies] established at the
    start of development are still valid and effective.

The next steps in the development of this application added some actual
    weblog content (you can read about all of it in my [DevLogs]).
[The following release] of [this application] provided
    [integration of Markdown content with custom parsing],
populating article pages from externally hosted Markdown files.
Incorporating the Markdown content continued to make the application more
    flexible,
and because of Markdown's simple, well-established structure,
this could be done without inviting any more complexity into the application
    design.
And for now, this all works well for my own website and articles, but the real
    test will be applying this template to other applications
— be it for other portfolios, blogs, or anything else that fits the design.

[^assets]: The static assets [are versioned separately] from the application.
[Version 0.0.2 of the assets] from [my personal website] released with
    [version 0.0.2 of the application].
It includes file definitions for all [the website's content configuration],
    [text definitions], and [theme settings].

[^comment]: These early iterations of the application are primarily concerned
    with simply displaying weblog articles and other information about me.
However, I have plenty of ideas for integrating more interactive features,
like comments and social media.

[I set up a SvelteKit application]: ./sveltekit.md
[my personal website]: https://carledwardlyons.ca
[the first update]: https://github.com/systemcarl/blank/tree/v0.0.2
[a very simple landing page]: https://carledwardlyons.ca
[The original version]: ./sveltekit.md
[a styled error message]: https://carledwardlyons.ca/errors/400
[underlying theming system]: ./sveltekit.md#theming
[the website's visual design mock-up]:
    https://www.figma.com/design/TJYtbshPU4K0CoXuYKqtwp/Portfolio?m=auto&t=IV2gWGSb6tnTZcel-6
[an early proof-of-concept]: ./sveltekit.md#an-empty-application
[the profile]:
    https://github.com/systemcarl/blank/blob/v0.0.2/src/lib/components/profile.svelte
[the contact information]:
    https://github.com/systemcarl/blank/blob/v0.0.2/src/lib/components/contact.svelte
[abstract all of the page content]: ./sveltekit.md#knobs-and-dials
[Hypertext Markup Language (HTML)]:
    https://developer.mozilla.org/en-US/docs/Web/HTML
[my design mock-up]:
    https://www.figma.com/design/TJYtbshPU4K0CoXuYKqtwp/Portfolio?m=auto&t=IV2gWGSb6tnTZcel-6
[my mission]: ../../about-me.md#it-has-great-practical-value
[the underlying application's theming system]: ./sveltekit.md#theming
[aggregated by the application from static JSON files]:
    ./sveltekit.md#knobs-and-dials
[an additional configuration object]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.2/config.json
[all of my previously implemented testing strategies]:
    ./sveltekit.md#testing-testing-testing
[DevLogs]: https://carledwardlyons.ca/#devlogs
[The following release]: https://github.com/systemcarl/blank/tree/v0.0.3
[this application]: https://github.com/systemcarl/blank
[integration of Markdown content with custom parsing]: ./markdown-parser.md
[are versioned separately]: https://github.com/systemcarl/folio-assets
[Version 0.0.2 of the assets]:
    https://github.com/systemcarl/folio-assets/tree/v0.0.2
[my personal website]: https://carledwardlyons.ca
[version 0.0.2 of the application]:
    https://github.com/systemcarl/blank/tree/v0.0.2
[the website's content configuration]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.2/config.json
[text definitions]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.2/locale.json
[theme settings]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.2/theme.json
