# DevLog: Designing a Content Template
Not long after [setting up SvelteKit application] for my [personal website]
    and blog,
I released and deployed the [first update] to the site adding a very simple
    [landing page].
The [initial version] of the application had no content at all, only a styled
    [error message].
The component framework and underlying [theming system] was already in place,
but this was the implementation of the [visual design] beyond a simple
    [proof-of-concept].

This next step was to create a basic, stylized landing page to provide a brief
    personal introduction.
Working from the design mock-up, I decided to include as little content as
    possible without having to revisit the design later.
Choosing to only implement the [profile] and [contact information] sections,
    the goal was just to add some static content elements, including:
- my name,
- a personal summary,
- an avatar image,
- an inspirational quote,
- some personal likes and dislikes,
- and a list of contact information.

## Strong Opinions, Loosely Held
I had [decided early] to decouple the page content (including static
    elements)[^assets] from the application.
The idea was to remove any personalized content from the application code,
allowing for easier updates to both the application and the content,
making it possible to reuse the same application code for different websites
    or blogs,
and fully enabling localization support in the future.
However, the core problem when designing a content system is balancing structure
    with flexibility.
Making the content structure too rigid takes the guesswork out of creating
    (and rendering) content,
but limits the creative possibilities for the composition as a whole.
On the other hand, a completely unstructured system allows for anything you can
    imagine,
but requires much more thought and consideration to create a final product that
    is both aesthetically pleasing and functionally effective.
Ultimately, responsibility for the design outcome needs to be either given to
    the content creator, or reserved by the content system itself
— the responsibility cannot simply be shared.

I chose to focus this solution on ease of content creation, creating a very
    opinionated content design.
However, opinionated does not mean immutable.
At a minimum, the content needs to be flexible enough to allow for
    personalization,
and the opinions built into the design only need to be strong enough to ensure
    a good design (whatever that may be).
But so long as the system is intuitive and resilient against unpleasant
    outcomes,
there is still some room to explore different content configuration structures
to suit needs other than my own.

### Personal Preferences
To make the content design as easy to use as possible,
all of the content configurations are optional.
Technically, to set up the application template, you don't even need to provide
    a title,
but it would hardly make sense to have a webpage without one.
Beyond that simple requirement, I designed the content and theme together so
    that regardless the amount of configuration provided,
the design would still look complete.

Since the application is primarily concerned with visual presentation
(there's nothing particularly interactive about a personal website and blog
    though I hope to add comments and other features in the future),
the constraints of the design are limited mainly to the layout.
So beyond having to dictate where each content element should go on the page,
the system does not impose any semantic meaning on the content itself.
For example, the "likes" and "dislikes" sections specified in [my design] does
    not need to be used for this purpose at all.
Structurally, this element is simply a pair of lists that can be used to
    represent any kind of comparison
(if it even really needs to be a comparison).
This visual component could just as easily be used to list pros and cons about
    a particular product, service, or philosophy.
I could have just as easily listed what I believe is, or is not, a sandwich.

### It Always Comes Back to Marketing
To approach the potentially endless design possibilities,
I found it helpful to determine the purpose of the content design, not just the
    subject matter (me).
My motivation for building this personal website comes down to (shameless)
    self-promotion.
It would be much easier to simply post generic articles to an established
    platform, simply to put the core information (straight to the point tips
    and guides) out there.
But that doesn't really capture my mission to create an identity and brand that
    other people in the community can relate to and follow.
These journal entries, for example, are not just meant to be a technical
    resource, but also inspire and motivate others to pursue their own
    ambitions.

And while I'm not asking anyone to buy into anything (besides maybe a like or
    a follow),
most content driven applications are no different — they are trying to market
    something.
Here, I am just marketing myself; a business markets its products or services;
So, it occurred to me that to generalize the design of my template effectively,
    the goal is to make a marketing template that is tailored specifically for
    personal profiles.

Reflecting on the purpose of each design element from this marketing
    perspective
helped to simplify and clarify the intended effect towards the overall design.
Each element has a role to play in establishing identity, credibility, or
    connection with the audience.
For example, an avatar image and logo both serve to establish visual identity.
Therefore, the presentation of the avatar element should emphasize this
    function,
making it prominent and visually distinct from the rest of the content.
The end result is a cohesive design where my cartooned face could easily be
    swapped out for a brand logo without disrupting the design integrity.

## An Agnostic Application
Despite the strong opinions built into the content design, the resulting
    application was still completely agnostic to the actual content itself.
As per the original design of the application, all of the content is still
    [aggregated from static JSON files].
Now, the files just include [an addition configuration object] to control the
    composition of pages, beyond the text and styling definitions.[^assets]
It was a choice of personal preference to not combine all the configuration
    into a single file,
even though there is coupling between the content and its presentation.
Despite spanning multiple files, I still consider these files to be a single
    source of truth for the website's content and presentation.
Breaking it down just makes it easier for me to manage and navigate.

While some might disagree, I have found that the dynamic nature of the
    application components has made development and testing much easier.
Abstracting away text and other data into component properties and context
    stores,
requires values to be injected into components during testing.
This prevents changes to the content or style of the final application from
    affecting the setup or expected outcomes of the tests.
All of the [testing strategies] established at the start of development are
    still valid and effective.

The next step in the development of this application was to add some actual
    weblog content.
The [following release] of [this application] provided
    [integration of Markdown content],
populating article pages from externally hosted markdown files.
Incorporating the Markdown content continued to make the application more
    flexible,
and because of Markdown's simple, well established structure,
this could be done without inviting any more complexity into the application
    design.

[^assets]: The static assets are [versioned separately] from the application.
[Version 0.0.2] of the assets from my [personal website] released with
    [this version].
It includes file definitions for all the websites [content configuration],
    [text definitions], and [theme settings].

[setting up SvelteKit application]: ./sveltekit.md
[personal website]: https://carledwardlyons.ca
[first update]: https://github.com/systemcarl/blank/tree/v0.0.2
[landing page]: https://carledwardlyons.ca
[initial version]: ./sveltekit.md
[error message]: https://carledwardlyons.ca/errors/400
[theming system]: ./sveltekit.md#theming
[visual design]:
    https://www.figma.com/design/TJYtbshPU4K0CoXuYKqtwp/Portfolio?m=auto&t=IV2gWGSb6tnTZcel-6
[proof-of-concept]: ./sveltekit.md#an-empty-application
[profile]:
    https://github.com/systemcarl/blank/blob/v0.0.2/src/lib/components/profile.svelte
[contact information]:
    https://github.com/systemcarl/blank/blob/v0.0.2/src/lib/components/contact.svelte
[decided early]: ./sveltekit.md#knobs-and-dials
[my design]:
    https://www.figma.com/design/TJYtbshPU4K0CoXuYKqtwp/Portfolio?m=auto&t=IV2gWGSb6tnTZcel-6
[aggregated from static JSON files]: ./sveltekit.md#knobs-and-dials
[an addition configuration object]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.2/config.json
[testing strategies]: ./sveltekit.md#testing-testing-testing
[following release]: https://github.com/systemcarl/blank/tree/v0.0.3
[this application]: https://github.com/systemcarl/blank
[integration of Markdown content]: ./markdown-parser.md
[versioned separately]: https://github.com/systemcarl/folio-assets
[Version 0.0.2]: https://github.com/systemcarl/folio-assets/tree/v0.0.2
[personal website]: https://carledwardlyons.ca
[this version]: https://github.com/systemcarl/blank/tree/v0.0.2
[content configuration]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.2/config.json
[text definitions]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.2/locale.json
[theme settings]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.2/theme.json
