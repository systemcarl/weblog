# Abstract
- This is an example case study for planning and implementing a personal site.
- This guide is a non-prescriptive explanation based on the planning process
    suggested in
    [*Planning an Software Project*](./planning-a-software-project.md).
- This article includes the
    - architectural choices and the reasoning behind them,
    - a tutorial for designing & developing a minimal viable product (MVP) using
        SvelteKit.

# Planning a Personal Site
- This article is example case study for planning and implementing a personal
site.
    - This example is meant to provide a practical, but non-prescriptive
        explanation of the planning  process.
    - This guide is also a personal exercise in planning and implementing a
        web page.
        - The process described is curated for simplicity; not a comprehensive
            log of my personal site development.
        - The true history of my personal site can be found in the project's
            Git history.
    - The process follows an iterative, top-down approach outlined in [*Planning
        an Software Project*](./planning-a-software-project.md).
- This article also serves as a tutorial for designing & developing test-driven,
    server-rendered Svelte applications.
    - [Later steps](#step-4) of this guide include an example design &
        implementation using
        - Figma design tools,
        - a Vite development environment,
        - the SvelteKit application framework,
        - the Vitest testing framework,
        - Docker containerization,
        - Terraform IaC deployment,
        - DigitalOcean hosting,
        - GitHub Actions CI/CD.
    - I will discuss my choices and the reasoning behind them, but for brevity,
        I will not list all the alternatives considered.
    - Scope of project is too large to detail every step of the development
        process, but the end result is available on GitHub.
    - Other tutorials are available for more detailed explanations of specific
        tools & technologies.

## <a id="step-1"></a>Step 1: Establish a Goal
- The fundamental goal is to promote personal interests, style, & professional
    skills.
- The project will be building a platform for sharing interests, including:
    - a weblog,
    - a professional portfolio.
- Guiding values for the project include:
    - Maintainability; the site should be easy to maintain, update, & extend
    to reduce time spent on upkeep.
    - Portability, as the solution should be easy to reuse and share with
    the technical community.
    - Accessibility; the final product should be easy to access & navigate by
    the widest possible audience.
    - Quality, as the site should reflect the best of the creator's abilities.
    - Modernity, as the site should showcase experience with latest technologies
        & design trends.
- Metrics for validation of the project's success will be:
    1. Can the solution be updated with little-to-no effort?
    1. Can the project resources be easily repurposed within another project?
    1. Does the implementation conform to applicable accessibility
        standards?
    1. Is the end result the best representation of the creator's abilities?
    1. Are all possible points of failure accounted for within the solution?
    1. Does the end result elicit a positive response from the audience?
    1. Does the solution use the best available technologies & design patterns?

### <a id="step-1-future-ambitions"></a>Step 2: Propose Additional Ambition
- No additional long-term objectives are (currently) applicable for this
    project.
    - Since the primary goal is to create a simple, modular, & extendable
        webpage, future scope may be externalized to other projects.
    - Ideal for experimentation with new technologies that likely are not
        currently available.

## <a id="step-2"></a>Step 2: Outline Project Architecture
- Architectural basis will be determined by the chosen technology stack.
    - Since the website ecosystem is mature and diverse, choose an existing
        framework that fits the project's requirements.
    - Narrow chosen framework features and implementation options to fit the
        project's goal.
- Additional dependencies will need to consider integration, deployment, &
    hosting requirements.
- Requirements for successful implementation include:
    - deployment, as it must be easy to deploy & maintain;
    - performance, as it must be optimized for universal client-side rendering.

### <a id="step-2-application"></a>Application Architecture
- The structure of the application is the most important aspect of developing
    any software project.
    - Application framework typically determines the fundamental architectural
        pattern.
- The architecture should be modular, scalable, & maintainable (as per the
    project's goal).

#### <a id="step-2-sveltekit"></a>Static Site Generator: SvelteKit
- Static site generators extend traditional web technology, combining static
    HTML documents with dynamic content management.
- Static site generation optimizes HTML payload for high client-side performance
    across all devices.
    - Instead, requires a server for dynamic content integration.
- Automated re-deployment via integrated content management.
- SvelteKit extends Svelte.
    - Svelte supports web components (both publishing & consuming).
        - Separates design & UI from application business logic.
        - Components will be developed independently, separating concerns
    - Svelte is designed for low development overhead:
        - Mature development environments are available for Svelte (*e.g.* Vite)
        - Svelte uses simple syntax for component-based design.
    - Allows for optimized client loading.
        - See [Svelte performance documentation](
            https://svelte.dev/docs/kit/performance
        ).
    - This project does not require low-level state management (*e.g.* React,
        Vue)
    - Provides an opportunity me to a learn new front-end web framework.
- Refer to [*Personal Site Web Application Architecture*]() design documents.
    - Outlines the deployment & network dependencies between the browser client,
        the web server, & the content management systems.

#### <a id="step-2-design"></a>Component Architecture
- Implementation of SvelteKit application can vary depending on the project's
    style & requirements.
- Components will be developed independently, separating concerns, & packaged
    into external modules when applicable.
    - Separating components into modules allows for easy sharing & reuse.
    - Modules can be developed independently, allowing for parallel or
        open-source development.
    - Module components will be packaged as web components for easy integration
        with other projects.
        - Exporting web components are supported by Svelte, and can be
            integrated with other web frameworks.
        - Components packaged as web components more easily be migrated to
            future projects, or future iterations of this project.
- Organizing components by concern allows for easier maintenance, testing,
    & documentation.
    - Clean component definitions establish resilient interface boundaries
- Refer to [*Personal Site Component Architecture*]() design documents.
- Components will be organized into the following categories:
    - **Material components**, smallest unit of design & user interaction.
    - **Stateful components**, components that require internal state
        management that is not shared with other components or persistent within
        the page.
    - **Slice components**, comprises components that are independent of
        context, with respect to both design & state.
    - **Layout components**, templates that organize common page elements.
    - **Page components**, components that represent a single page or view;
        renders URL-specific content & manages query-based state.
    - **Application components**, components that manage global state,
        configuration, or other application-wide concerns.
- Values & styles used in components will be decouple from the component
    definitions to allow for further customization & sharing.
- All dependencies will be injected via the Svelte framework (contexts, stores)
    - This allows for testing components in isolation using Vitest.
    - Svelte contained components are also easier to distribute as web
        components.

### <a id="step-2-cloud"></a>Cloud Architecture
- Application deployment is typically separate from application architecture,
    mirroring the separation between environment & application.
    - This is true in this case, as the SvelteKit framework & Vite are meant to
        be portable development environments.
- A plan for deployment & hosting helps avoid conflicts between the application
    development & product deployment.
    - Application architecture may be influenced by the cloud architecture or
        vice versa, depending on the project's priorities.
    - In this case, the cloud architecture is designed to distribute the final
        application, and is less of a concern.
- To make sure the application is easy to update and maintain, the focus of
    the cloud architecture will be on automation.
    - There are many tools available for automating the build, testing, and
        deployment process.
    - Reducing the amount of manual work required to deploy a new version of
        the application code will be essential for achieving the project's
        goal.
- Because cloud resources are out of my control, the cloud architecture
    will be designed to be portable.
    - Since most resources are provided a commercial, proprietary service, it
        is important to avoid vendor lock-in.
    - Decoupling the application from the cloud provider, or a specific dev/ops
        tool all together, will allow for easier migration between cloud if
        necessary.

#### <a id="step-2-docker"></a>Containerization: Docker
- To make sure the application is portable, it will be packaged as a Docker
    container.
    - Docker containers bundle all the necessary dependencies for the
        application to run, ensuring a consistent environment between contexts.
    - Tests can be run against the container independently of the environment.
    - Automation of Docker deployment is ubiquitous across tools and cloud
        providers.
    - Leveraging docker will make the automation process much more approachable.

#### <a id="step-2-terraform"></a>Infrastructure as Code (IaC): Terraform
- Terraform is a cloud-agnostic IaC tool that allows for simple, repeatable
    deployment.
    - Terraform's declarative syntax allows for stateless management of stateful
        cloud resources.
        - Declarative structure makes it resilient to changes and highly
            compatible with version control.
    - Cloud-agnosticism allows for easy migration between cloud providers.
        - Agnosticism decouple operational concerns from specific cloud
            providers or environments.
        - Reduced provider dependence is great for experimentation and sharing
            code for educational purposes.
- Terraform alone does not automatically scale or maintain resources.
    - Not a concern considering the project's scope.
- Refer to [*Personal Site Cloud Architecture*]() design documents.
    - Outlines the simple network dependencies between the cloud provider, the
        web server, & the control node.
    - Control node may be a local machine or a cloud-based development
        environment, or a CI/CD platform.
        - Requires a state backend if the control node is not a dedicated,
            stateful development environment.
- During implementation, the resource requirements will be decoupled from the
    provider specific definitions to allow for easy portability.

#### <a id="step-2-github"></a>CI/CD: GitHub Actions
- GitHub Actions is a CI/CD tool that facilitates the automation of development
    workflows.
    - GitHub Actions naturally integrates with `git` repositories hosted on
        GitHub, making it easy to tie events to automated processes.
    - Actions also provide a cloud hosted environment for running
        workflows, which reduces the need for a dedicated control node or
        dependence on a local development environment.
    - To keep things simple, the GitHub Actions workflow be used mostly for
        triggering processes and hosting, and rely on other elements of this
        projects to define the processes.

#### <a id="step-2-github-actions"></a>Cloud Hosting: DigitalOcean
- For a content-driven cloud architecture should be simple, but allow for
    low-effort deployment, scaling, maintenance.
    - *DigitalOcean* provides a simple cloud hosting interface, but still
        provides the level of control required for fine-tuning a production
        environment.
    - DigitalOcean, for the most part, provides direct access to the
        underlying infrastructure, making the implementation of infrastructure
        generic and reasonably portable.

## <a id="step-3"></a>Step 3: Establish a Scope
- The scope of the first iteration of the project will be defined by the minimum
    viable product (MVP).
- Simplest possible implementation would be a static page with minimal content
    - Best approach for a novice developer.
    - Product would be immediately outdated and require development overhead
        to maintain.
- Skipping to a simple, dynamically generated weblog will provide an MVP that
    provides immediate value.
    - Simple static page will still be an iteration of the MVP
        [implementation](#step-6).

### <a id="step-3-mvp"></a>MVP: Weblog & Portfolio
- Render a page, linking to dynamic weblog & portfolio articles along with
    minimal static content.
    - Including as much content as possible in articles will prioritize dynamic
        content management.
- Display full list of articles on a weblog page with optional filtering
    by category.
    - May not be possible to highlight all articles on the front page.
    - Providing index allows for easy navigation & search beyond main personal
        page.
- Full articles will be displayed on separate, dedicated pages.
    - Dedicated URLs allow for easy sharing & linking to specific articles.
    - Some articles may be longform, requiring too much space to be
        conveniently displayed within compound layouts.
- Static content will be decoupled from the application, when possible, but
    still be deployed & hosted with the application.
    - Ensures static content can me modified without
- Articles will be dynamically integrated via external article dependency.
    - Article hosting is a separate concern from the personal site and
        has potential for reuse in other projects.
    - Planning, design, & implementation is similarly described in
        [*Planning a Blog Service*](./planning-a-blog-service.md).

### <a id="step-4:"></a>Step 4: Design the Solution
- SvelteKit application will use the default Vite template, which will provide
    all the necessary site generation infrastructure.
- The project-specific application design will be composed of:
    - routing structure,
    - article & configuration dependencies,
    - component design.
- SvelteKit provides simple file-based routing, which will be used to define 3
    pages:
    - the main personal page, `/`;
    - the article index, `/articles/`;
    - the page for displaying a full article, `/articles/<article>/`.
- Category and article pages will be dynamically generated based on Additional
    URL path tokens.
    - A subset of the article index will be displayed based on chained category
        path tokens: `/articles/<category>/<subcategory>/`.
- Pre-rendering will only be used to generate the main page, as the other pages
    will require up-to-date article content.
- Articles and configuration will be wrapped in dedicated stores, respectively.
    - Article data will be server-rendered and not required to be accessible
        from the client.
    - Configuration data, specifically theme variables, will be used by the
        client to render the page.
- The component-based web design is outlined in the [*Personal Site Design*]()
    Figma document.
    - The design components will be mapped to Svelte components.
    - The web design is composed of hierarchical components to mirror a proposed
        design system following organization prescribed in
    [Component Architecture](#step-3-design).
    - Figma design does not match HTML/CSS behaviour exactly; changes to the
        design will likely need be made during implementation, when necessary,
        to achieve the specified design.
