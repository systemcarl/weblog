# Abstract
- I am currently developing a my own personal website.
- To begin developing the site front-end, I outlined the web pages with a simple
    design system.
- The design system was prototyped using [*Figma*].

# Personal Site Web Design
- In the past I have never used a design system to develop a website.
    - Rarely have I ever been given a web design to implement.
    - Before discovering [*Figma*], it was easier to prototype a design directly
        in HTML.
    - The process was always time consuming.
- A [design system] helps streamline the development process.
    - A web design provides a clear implementation goal.
    - A design system encourages consistency by limiting the options available
        to construct the design.
    - In a team, a design system provides a common language for collaboration.
- When building my [personal website], I preemptively created a design system.
    - To begin, I drafted the primarily pages of the website.
    - I identified the common elements of the design and created a set of
        reusable components.
    - I redesigned the pages using only:
        - the reusable components,
        - a consistent layout,
        - and a limited set of variables.

## The Draft
- Since I had no plan, I began with a rough draft.
    - I created a complete design without any guidelines or constraints.
    - The only goal was to satisfy high-level project requirements:
        - the necessary content,
        - semantic website structure.
- The lack of constraints allowed for free-flowing creativity.
    - I was easy to realize a vision of the website as a whole.
    - I could focus on the content and structure of the website, and how it
        should be presented.
    - I experimented with different layouts, fonts, color.
- The [rough design] is available on *Figma*.

## The Design System
- The rough design alone would be more than enough in some cases.
    - Inconsistencies with the design would likely resolve themselves during
        implementation.
    - The scale of this project is small enough that any future changes would
        be trivial to implement.
- A design system is still useful.
    - When adding new elements or entire pages, the design system will increase
        productivity.
    - Having useful, reusable building blocks removes the need to re-implement
        the same design elements.
    - Abstracting core design decisions reduces the cognitive load of extending
        the project.

### Components
- Reflecting on the rough design, I identified the common elements.
    - I broke the design down into the smallest atomic elements.
    - Comparing the atomic elements, I looked for common elements:
        - headings,
        - paragraphs,
        - links,
        - lists,
        - images.
- For each design element, I created a [set of reusable components].
    - When possible, I used existing components from the design system.
    - Often, components could be reduced to a set of sub-components to increase
        component reuse and simplify the design system.
- I created a [*Figma* component] for each design element.
    - [Variants] can be used to extend components to be more flexible.
    - I used component [properties] to allow for customization of the
        components.
    - The components also use [styles] to set consistent size, color, and
        typography.
    - Limiting style options to the smallest semantic set of named values will
        help with theming.
        - Equal or similar values should be grouped together, unless they
            represent distinct thematic elements.

### Slices
- With the basis of the design system in place, I began to recompose the design.
    - Since the webpage follows a traditional vertical scrolling convention,
        each section of the page could be designed independently.
    - I constructed the design of the [content sections] slices to resemble the
        rough design.
    - The design was created using only instances of the reusable components.
    - In many cases, the components had be be changed or extend to fit the
        design.
- To keep the layouts consistent between the content sections, I used
    [variables].
    - Spacing between elements were set to different spacing variables.
- For each content section, I adjusted the layout to a set of horizontal
    break points.
    - I designed the layout to the lower limit of each break point window
        because the design content stretches more easily than it shrinks.
    - Each layout was created as a variant of the content section component
        for easy page composition.
    - The break points were set to variables to make the *Figma* file resilient
        to specification changes.

## The Design
- With the elements of the design system in place, the final [page designs]
    could be composed.
    - Each page consists of content section instances.
    - Page instances can be used to visualize the complete design across all
        targeted [breakpoints].

## The Theme
- No good design is complete without theming.
    - Light and dark themes are expected.
    - Themes can also be used to address accessibility concerns:
        - color blindness,
        - low vision,
        - high contrast.
- My design system uses the file's [variables] and [styles] to prototype
    website theming.
    - Common style values can be organized by linking styles to variables.
    - Variable [modes] can be used to extend these prototyping capabilities.
        - This requires a paid or educational *Figma* account.

[*Figma*]: https://www.figma.com/
[design system]:
    https://www.figma.com/blog/design-systems-101-what-is-a-design-system/
[personal website]: https://carledwardlyons.ca/
[rough design]:
    https://www.figma.com/design/TJYtbshPU4K0CoXuYKqtwp/Portfolio?node-id=0-1
[set of reusable components]:
    https://www.figma.com/design/TJYtbshPU4K0CoXuYKqtwp/Portfolio?node-id=111-267&t=cKmjUdXUh4juIdZz-1
[*Figma* component]:
    https://help.figma.com/hc/en-us/articles/360038662654-Guide-to-components-in-Figma
[Variants]:
    https://help.figma.com/hc/en-us/articles/360056440594-Create-and-use-variants
[properties]:
    https://help.figma.com/hc/en-us/articles/5579474826519-Explore-component-properties
[styles]:
    https://help.figma.com/hc/en-us/articles/360039238753-Styles-in-Figma-Design
[content sections]:
    https://www.figma.com/design/TJYtbshPU4K0CoXuYKqtwp/Portfolio?node-id=162-24711&t=cKmjUdXUh4juIdZz-1
[variables]:
    https://help.figma.com/hc/en-us/articles/15339657135383-Guide-to-variables-in-Figma
[page designs]:
    https://www.figma.com/design/TJYtbshPU4K0CoXuYKqtwp/Portfolio?node-id=167-62355&t=cKmjUdXUh4juIdZz-1
[breakpoints]:
    https://www.figma.com/design/TJYtbshPU4K0CoXuYKqtwp/Portfolio?node-id=177-41506&t=cKmjUdXUh4juIdZz-1
[modes]:
    https://help.figma.com/hc/en-us/articles/15343816063383-Modes-for-variables
