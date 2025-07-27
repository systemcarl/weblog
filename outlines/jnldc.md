# Abstract
- Jerry Noordam Landscape Design & Consultating was my first ever production
    project.
- This simple digital brochure was as a way to get some experience in web
    design & development.
- The website's minimalist design was implemented with nothing more than vanilla
    HTML, CSS, and JavaScript, served by a Laravel application.

# Jerry Noordam Landscape Design & Consultating
- My first production project was a simple digital brochure for [Jerry Noordam
Landscape Design & Consultating](https://jerrynoordam.ca).
    - This was a *pro bono* project for a dear family friend.
    - Developing this website was a great introduction to web development and
        design.
- The project scope was an ideal opportunity to explore traditional web
    development technologies.
    - The website was built using Laravel, a historically popular PHP framework.
    - The design was implemented with HTML & CSS.
    - Some custom menu & gallery interactions were implemented with vanilla
        JavaScript.

## A Basic Solution
- The project was completed in 2020, and the website is still live today.
    - Development was completed over 2 - 3 months between 2019 and 2020.
    - Due to the simple, dependency free implementation, the website has been
        running with virtually no maintenance since its launch.

### A Deliberately Nostalgic Design
- The website features a minimalist visual design, focusing entirely on the
    content.
    - As simple, classic visual design was requested to match existing branding.
    - The request fit well with a minimalist, pure HTML & CSS composition &
        styling.
    - Aesthetic components require minimal JavaScript.
    - The result was a flat, crisp, approachable user interface (UI) reminiscent
        of the late 2000s or early 2010s.
    - The responsive layout ensures the website is sufficiently functional on
        displays at least 350px wide.
- The visual design was developed to match the graphic assets provided by Jerry.
    - The pallet was chosen to complement the existing logo, selecting a range
        of natural colours & earth tones.
    - A classic serif font was chosen to give the content a timeless,
        professional tone.

### Simple, Reliable Web Pages
- The website is structured to highlight various services offered by Jerry and
    related portfolio projects.
    - The website comprises several pages, some static and some dynamically
        populated with portfolio content.
    - Each offered service is cross-referenced with an applicable portfolio
        project.
- The website features a custom gallery, providing a curated view of each of
    Jerry's portfolio projects.
    - The pages are dynamically populated with images from a local directory.
    - Laravel's Blade templating engine was used to dynamically generate the
        gallery pages.
    - The gallery's expandable image view was implemented with a simple imbedded
        JavaScript.
- An embedded contact form was included as part of the contact section.
    - The form was implemented as a simple HTML form.
    - Custom Javascript was used to dynamically filter form options, validate
        information, and format input text.

## The Good, the Bad, and the Ugly
- I am very proud of this project.
    - It felt satisfying to use my skills to make a positive impact on a
        business.
    - The nostalgic feel of the website is refreshing to look back on, compared
        to cluttered complexity of contemporary web applications.
    - Despite some limitations and oversights, the project has aged relatively
        well.
- The skills and knowledge gained from this project were foundational to my
    work in web development.
    - The simplistic, traditional web design was a great introduction to basic
        web technology.
    - Larvel implements a standard request-response cycle, using an explicit
        MVC architecture, common across many web frameworks.
    - Direct implementation of HTML, CSS, and JavaScript helped me understand
        the fundamentals of front-end development.
- In retrospect, the end result of this project is closer to a polished
    prototype than a finished product.
    - The design and functionality were sufficient for the project's scope,
        but not a complete solution.
    - A proper scalable solution would likely require refactoring the
        application to use a suite of modern web development tools.

### Clean, Fast, but Not Perfect
- I still enjoy how fast and responsive the plain server-side rendered pages
    are.
    - The CSS state pseudo-classes provide a simple, efficient way to implement
        hover and focus states.
    - The JavaScript used requires no importing or bundling, and executes
        quickly.
    - Achieving to same responsiveness with a modern front-end framework is
        possible, but can require significant effort.
- The simplicity of the website makes it very reliable.
    - The only front-end dependencies are well established browser APIs.
    - The website has not required any maintenance since 2020, except for 2 or 3
        faults related to the Linux & Apache.
    - The small resource footprint of the website has made it easy to host
        alongside other projects, minimizing costs.
- Without a contemporary UI component library, the user experience (UX) of the
    website is limited.
    - The website does lacks essential accessibility features, such as
        keyboard navigation and screen reader support.
        - The main navigation menu is not keyboard accessible.
        - The gallery's expandable image view does not support keyboard
            navigation, or have ARIA attributes.
    - Some of the JavaScript interactions do not conform to modern expectations.
        - The gallery's expandable image view does not provide a clear way to
            close the view.
        - Expandable image view does have a means to zoom in on images, making
            images more difficult to view on smaller screens.
    - Addressing every accessibility & usability issue requires significant
        effort without the support of front-end tools.
- Overall, developing a project without a front-end framework is tedious and
    time consuming.
    - With many open source headless component libraries available, it is fast
        and easy to implement the basics of a modern web UI.
    - It is difficult to create modular design system within a front-end
        application without a CSS preprocessors or a component library.
    - If I were to develop this project today, I would likely use Svelte.

### Unrelatable Content Management
- The relational structure of the project content is simple and effective.
    - Customer types (homeowners & contractors) are mapped to services, which in
        turn are mapped to portfolio projects.
    - The dynamic content provided clear structure to the website.
    - Services and portfolio projects can be added or removed without without
        modifying the application.
- However, the content is not easily managed.
    - At the time, I was not familiar with common content management practices,
        or comfortable building & deploying a database.
    - The content is stored in a local directory.
        - Better hosted on a Content Delivery Network (CDN) or cloud storage
            service.
    - Aside from a page to view form submissions, there is no content
        management interface.
    - Building a content management interface would be challenging without a
        full stack Object Relational Mapper (ORM) or a headless CMS.
- Overall, the content is not served efficiently.
    - Full image resolution is served to all users regardless of the image
        size displayed.
    - Server responses do not take full advantage of browser caching.
