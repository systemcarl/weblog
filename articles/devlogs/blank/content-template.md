# DevLog: Designing a Content Template
Not long after [I set up a SvelteKit application] for [my personal website]
    and blog,
I released and deployed [the first update].
[The original version] of the application had no content at all, only
    [a styled error message], which provided [an early proof-of-concept].
With this, the application's component framework
    and [underlying theming system] was already in place.
The update now added [a very simple landing page]
    including the implementation of [the website's visual design mock-up].

The basic, stylized landing page provided a brief
    personal introduction.
Working from the design mock-up, I decided to include as little content as
    possible
without leaving anything looking incomplete or needing to be otherwise reworked
    within the planned scope of the design.
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
However, my ambitions for this project go beyond just implementing a static web
    design.
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
    content is anything not strictly necessary for the application function, including:
- text content (both copy and long-form content),
- images and graphics,
- colours and backgrounds,
- and even layout and navigation configurations.

Abstracting the personal content away from the application code also made it
    possible to reuse the same application code for different websites or blogs,
and fully enable localization support in the future.
The core of my application therefore needed to support this abstraction,
wherein the content is injected into the application pages when rendered.

> [!NOTE]
> In practice, this means that the design elements (*e.g.*, the page title) are
    defined in the application code.
The inclusion and layout of these elements are fixed, or restricted to certain
    content configurations.
However, the styled [Hypertext Markup Language (HTML)] elements rendered to the
    displayed page (*e.g.*, title copy, size, font, and colour)
are all defined by the injected content.

The core problem when designing such a content system is balancing
    structure with flexibility.
On one hand, making the content structure very rigid takes the guesswork out of creating
    (and rendering) content,
but limits the creative possibilities for the composition as a whole.
On the other hand, a completely unstructured system allows for anything you can
    imagine,
but requires much more thought and consideration to create a final product that
    is both aesthetically pleasing and functionally effective.
Ultimately, responsibility for the design outcome needs to be either given to
    the content creator, or reserved by the content system itself;
    the responsibility cannot simply be shared.

I chose to focus this solution on ease of content creation with a strongly
    opinionated content design
(opinionated in the sense that the page structure and content are predetermined
    and the template constraints must be respected).
However, opinionated does not mean inflexible.
At a minimum, the content definitions
— the rules and restrictions that define what can and cannot rendered to the
    page —
need to be flexible enough to allow the results to feel uniquely personalized.
Moreover, the content definitions built into the design only need to be
    restrictive enough to ensure a good design (whatever that may be).
But so long as the system is intuitive and protects against illogical choices,
there is still some room to explore broader content configuration structures
to suit needs other than my own.

### It Always Comes Back to Marketing
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

### Personal Preferences
To make the content design as easy to use as possible,
all of the content configurations are optional.
Technically, to set up the application template, you don't even need to provide
    a page title (the text displayed in the browser tab),
but it would hardly make sense to have a webpage without one.
Beyond that simple requirement, I designed the content and theme together so
    that regardless of the amount of configuration provided,
the design would still look complete.

Furthermore, reflecting on the purpose of each design element from a marketing
    perspective
helped to simplify and clarify each element's intended effect towards the
    overall design
(e.g., establishing identity, credibility, or connection with the audience).
For example, an avatar image and logo both serve to establish visual identity.
Therefore, the presentation of the avatar element should emphasize this
    function,
making it prominent and visually distinct from the rest of the content.
The end result is a cohesive design where my cartooned face could easily be
    swapped out for a brand logo without disrupting the design integrity.

Since the application is primarily concerned with visual presentation
(there's nothing particularly interactive about a personal website and blog,
    though I hope to add comments and other features in the future),
the constraints of the design are limited mainly to the layout.
So, beyond having to dictate where each content element should go on the page,
the application does not impose any semantic meaning on the content itself.
For example, the section in [my design mock-up] listing my "likes" and
    "dislikes" do not need to be used for this purpose at all.
Structurally, this element is simply a pair of lists that can be used to
    represent any kind of comparison
(if it even really needs to be a comparison).
This visual component could also be used to list pros and cons about
    a particular product, service, or philosophy.
I could have just as easily listed what I believe is, or is not, a sandwich.

## An Agnostic Application
Despite the strong opinions built into the content design, the resulting
    application is still agnostic to the content itself
— unconcerned with the specific content being rendered to the page, and only
    concerned with how to render it.
That is, all of the content is still
    [aggregated by the application from static JSON files] after the application
    has been built and deployed,
as per the original design of the application.
The difference with this update is the files now include
    [an additional configuration object] to control the composition of pages,
beyond copy text and HTML style definitions.[^assets]

> [!NOTE]
>It was a choice of personal preference to not combine all the configurations
    into a single file,
even though the segregated files that comprise the configuration are strictly
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
[All of the previously implemented testing strategies] established at the start
    of development are still valid and effective.

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

[^assets]: The static assets [are versioned separately] from the application.
[Version 0.0.2 of the assets] from [my personal website] released with
    [version 0.0.2 of the application].
It includes file definitions for all [the websites content configuration],
    [text definitions], and [theme settings].

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
[aggregated by the application from static JSON files]: ./sveltekit.md#knobs-and-dials
[an additional configuration object]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.2/config.json
[All of the previously implemented testing strategies]: ./sveltekit.md#testing-testing-testing
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
[the websites content configuration]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.2/config.json
[text definitions]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.2/locale.json
[theme settings]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.2/theme.json
