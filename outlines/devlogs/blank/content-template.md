# Abstract
- While building the static content for my personal website & blog, I have been
    exploring how to structure generic content designs.
- Creating a reusable content template requires consideration of both visual
    design and data structure.

# DevLog: Designing a Content Template
- The [first update] to my [personal website] was also the first step in
    creating a reusable content template.
    - The [first version] of [the application] had no content, only a styled
        [error message].
    - The component framework and underlying [theming system] was in place, but
        the [visual design] had not been enacted beyond the simplest
        [proof-of-concept].
- The next step was to create a simple [landing page] to give myself a very
    brief personal introduction.
    - Working off a simple design mock-up the goal was to implement the
        [profile] and [contact information] sections, adding:
        - my name,
        - a short description,
        - an avatar image,
        - an inspirational quote,
        - likes and dislikes,
        - and contact information.

## Strong Opinions, Loosely Held
- The core problem when designing a content system is balancing structure with
    flexibility.
    - A rigid structure makes it easy to create content, but limits the scope of
        possible applications.
    - A completely unstructured system allows can be used for anything, but
        requires more effort to create aesthetically pleasing content.
- This solutions focuses on ease of content creation.
    - The content will only be flexible enough to allow for personalization.
    - Any customization option will be designed to be intuitive and resilient
        against unpleasant design outcomes.

### Personal Preferences
- All of the content configurations are optional.
    - Aside from the bare minimum of a title, the design can be used with any
        combination of content elements.
- For this application, the constraints are limited mainly to the layout.
    - The semantics of the content elements are up to the content creator.
    - For example, the "likes" and "dislikes" sections could be used to
        represent
        - other pros and cons,
        - or any other comparison: what is or is not a sandwich.

### It Always Comes Back to Marketing
- Determining the purpose of the content design helps inform open-ended design
    decisions.
- Most content driven applications have some marketing aspect.
    - A personal website is simply a way to market oneself.
    - Therefore, to generalize the design effectively, the goal is to make a
        marketing template that works well for personal profiles.
- Reflecting on the purpose of each design element has helped to clarify the
    effect of the overall design.
    - An avatar image and logo both establish visual identity, and the
        presentation of the avatar element should emphasize this.

## An Agnostic Application
- Despite the opinions of the content design, the application itself is still
    agnostic to the content.
    - All content is still [aggregated from static JSON files].
    - A [new file] was added to define the content structure.
- The dynamic nature of the application components has made development and
    testing easier.
    - Abstracting away text and other data makes logic and tests resilient to
        application changes.
    - The [testing strategies] implemented in the initial application release
        are still efficient and effective.
- [Integrating Markdown content] to populate weblog posts provided more
    flexibility without sacrificing structure, at the expense of complexity.

[first update]: https://github.com/systemcarl/blank/tree/v0.0.2
[personal website]: https://carledwardlyons.ca
[first version]: ./sveltekit.md
[the application]: https://github.com/systemcarl/blank
[error message]: https://carledwardlyons.ca/errors/400
[theming system]: ./sveltekit.md#theming
[visual design]:
    https://www.figma.com/design/TJYtbshPU4K0CoXuYKqtwp/Portfolio?m=auto&t=IV2gWGSb6tnTZcel-6
[proof-of-concept]: ./sveltekit.md#an-empty-application
[landing page]: https://carledwardlyons.ca
[profile]:
    https://github.com/systemcarl/blank/blob/v0.0.2/src/lib/components/profile.svelte
[contact information]:
    https://github.com/systemcarl/blank/blob/v0.0.2/src/lib/components/contact.svelte
[aggregated from static JSON files]: ./sveltekit.md#knobs-and-dials
[new file]:
    https://github.com/systemcarl/folio-assets/blob/v0.0.2/config.json
[testing strategies]: ./sveltekit.md#testing-testing-testing
[integrating Markdown content]: ./markdown-parser.md
