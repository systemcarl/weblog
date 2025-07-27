# Abstract
- This article describes an abstracted, non-prescriptive, step-by-step process
    for planning a software project.
    - The process described is intended to balance technical planning & product
        design.
- Throughout the article, the process is described in detail with reasoning
    grounded in common software development philosophies and practices.
- The purpose of the article is to provide a bridge between practical examples
    and the theoretical concepts of software architecture, design, and
    implementation.

# Planning a Software Project
- This articles describes an abstracted, step-by-step process for planning a
    software project.
    - While some steps are specific to software development, when applicable,
        the process has been generalized to be applicable beyond software
        development.
    - Project planning is complex; no single process is ideal for all projects.
- Most of this article is dedicated to explaining the reasoning behind the
    process.
    - The goal of the article is to make it easier to adapt the advice to other
        planning processes.
    - This process follows many commonly accepted software development
        philosophies, but includes some of my own opinions.
    - The focus of this article will be on abstract application, and will not
        include detailed discuss of various methodologies, philosophies, or
        frameworks.
- The process is a non-prescriptive, example of how a software project can be
    planned.
    - This process is an iterative, top-down approach to software architecture,
        design, & implementation.
    - The process comprises traditional technical project planning and more
        contemporary product design philosophies.
        - The process is compatible with a [modified waterfall model]()
            [*](/references.md#royce-w-1987).
        - The approach is influenced by "agile" software philosophies
            [*](/references.md#agile-manifesto).
    - Use your own knowledge, judgement, & experience to adapt the process to
        your or your team's needs.
    - Ideal for larger, longer-lived projects; not ideal for smaller,
        short-lived projects or projects with a high degree of uncertainty.
- Top-down software design may not be the best approach for all projects.
    - Bottom-up design is often suggested to forgo upfront planning and instead
        focus producing a viable product. <!-- Ref? -->
    - Since the priority is to have an actionable, long-term direction, a
        top-down approach is used.
- Many common industry philosophies are also used in this process.
    - Agile software development prioritizes production efficiency over
        strict proceed.
        - Even though the process is top-down, it is still iterative, flexible,
            and interactive.
        - The upfront planning is not a rigid, unchangeable plan; it is a
            flexible, living document that can be changed as the project
            progresses.
        - The architectural planning is limited to only the specifications
            necessary to guide the direction of the project and insure the
            project's long-term viability.
    - Test-driven development (TDD) is a common software development practice
        that is used to ensure the software is built to specification.
        - Well organized and maintained integrated testing is the best way to
            ensure the software is reliable and robust.
        - TDD is well suited for an iterative, top-down approach to software
            architecture, but not a good fit for design approaches where
            specifications are not defined in advance.
    - This process facilitates continuous integration (CI) and continuous
        deployment (CD).
        - CI/CD process are not described by this process, but all
            development-operations (DevOps) practices can can be considered
            part of the software plan.
        - Iterative design, implementation, and validation leaves room for
            CI/CD practices, and CI/CD may even be necessary for timely
            validation.

## <a id="project-planning-process">A Top-Down Iterative Planning Process</a>
- The flow of the process is outlined in this [*Project Planning Flowchart*]()
    that visualizes the iterative planning, designing, and implementation
    process.
    - The steps of the process are:
        1. [Establish a Goal](#step-1)
        1. [Outline the Software Architecture](#step-2)
        1. [Define the Scope](#step-3)
        1. [Design the Solution](#step-4)
        1. [Narrow the Scope](#step-5)
        1. [Implement & Test](#step-6)
    - Steps 2 through 6 may be repeated as necessary to continuously refine
        the project plan.
    - The entire process may be repeated over the lifetime of the software.

### <a id="step-1">Step 1: Establish a Goal</a>
- Understanding the motivation for the project is the most important step
    in the planning process.
    - The main goal:
        - sets the direction for the project,
        - influences decision making,
        - and reflects the reason for investing in the project.
    - The goal is internal to the project, and ought not be confused with the
        individual interests of external stakeholders.
    - There is no bad goals; especially if the goal is to learn something new.


#### <a id="step-1-values">Values</a>
- The mission of a project in turn can be broken down into values.
    - Breaking down the main goal into smaller, abstract values helps to direct
        decisions that cannot be directly dictated by the overarching goal.
    - Having consistent values helps to ensure that the entirety of the project
        is aligned with the main goal, not just the ultimate criteria for
        success.
    - Values can also help to reevaluate the main goal, in case the goal no
        longer aligns with the intention of the project.

#### <a id="step-1-validation">Validation</a>
- In this case, validation provides a way to evaluate project process, but not
    dictate the project direction.
    - Metrics for validation can be objective or subjective; quantitative or
        qualitative.
    - Validation is not prescriptive, as rigid absolutes can be easy misused to
        justify decisions that are not aligned with either the goal or the
        values. <!-- Ref? -->
    - Validation results are descriptive enough to inform future decisions.
    - Ideally, validation poses an open-ended question that evaluates the
        current state and trajectory of the project.

#### <a id="step-1-future-ambitions">Future Ambitions</a>
- The final direction of a project can end up being very different from the
    original goal.
    - The market, technology, and adoption can often change faster than projects
        can be developed. <!-- Ref: The Tar Pit / Lean Software ? -->
    - Even predicted changes can be difficult to account, and generally scale
        poorly. <!-- Ref: The Tar Pit / Lean Software ? -->
    - It can be better to consider these far-future plans to be beyond the
        scope of the current project.
- Considering future ambitions within the current project can still help extend
    the project beyond the initial goal.
    - It is important to keep the primary goal focused and achievable.
    - To build more ambitious aspirations into the project plan, additional
        goals can be appended to the main goal.
    - These additional goals do not supersede the main goal.
    - Additional goals can inform decisions that are ambiguous with respect to
        the main goal & values, but may have an impact on the software beyond
        the current project scope.

### <a id="step-2">Step 2: Outline the Software Architecture</a>
- Once the goal is established, the core tools and technologies can be selected.
    - Ideally, decisions are not limited to necessary choices that require
        upfront commitment.
    - Architectural choices outline architecture, but not dictate or constrain
        design.
- Architecture is typically focused on technical needs and dependencies.
    - Consider the fundamental requirements needed to achieve the goal.
    - These decisions consider both the goal and values.
    - Ideally, the mechanisms for validation are included in the architecture
        and design.
- Architecture requires investigation and research.
    - Beyond the technical specifications of the project, the architecture
        exhaustively explores all project values.
    - The architecture can considers the interests of external stakeholders,
        with respect to the goal and values.
    - Research can be theoretical or practical, and can include:
        - auditing documentation,
        - conducting case studies,
        - and building prototypes.
- The project architecture also may consider project constraints.
    - Validity of the architecture is evaluated against the limitations of the
        project:
        - assets,
        - time,
        - and staffing.
    - All projects have constraints, even if it is just your own time and
        energy.
    - Project management is historically a difficult task, especially in
        software development.
        - This topic has been discussed extensively in the literature.
        - A [*The Mythical Man-Month*](
            https://en.wikipedia.org/wiki/The_Mythical_Man-Month), by
            [Frederick P. Brooks Jr.](https://www.cs.unc.edu/~brooks/) contains
            a historically interesting discussion of project management.
- Generally, I believe project management is separate from project planning.
    - Project management is the task of allocating project resources.
    - Project planning is the task of defining the project and the tasks
        necessary to achieve the goal.
    - Project management fulfils the requirements of the project plan, and the
        project plan facilitates management of the project.
    - The later steps of this process themselves are designed to be compatible
        with popular project management philosophies.

### <a id="step-3">Step 3: Establish a Scope</a>
- Defining the scope of the project translates the goal, values, and
    architecture decisions into finite tasks.
    - The scope:
        - is more precise, and detailed than the goal,
        - provides a complete, clear, actionable definition of project
            completion,
        - and is not the design, and does not prescribe the design.
- The simplest scope, is the preferred scope.
    - Consistent with minimum viable product (MVP) philosophy, the scope is the
        simplest description of the final product that achieves the goal.
    - The end result is viable, but not necessarily ideal.
    - Balancing achievability with value is important.
        - It's possible that a slightly more complex scope may be much more
            valuable than the simplest scope.

### <a id="step-4">Step 4: Design the Solution</a>
- The design of the solution can be the most resource intensive step of any
    project.
    - In theory, there is no limit to the complexity of a design.
    - Upfront design has long been criticized for being prone to wasteful
        revision[*](#ref-royce-w-1987).
- Design can be limited to the choices necessary for efficient, consistent
    implementation.
    - Defining the design in terms of scope ensures that the design is not
        extending what is required.
    - Other mechanisms can be used to manage design complexity.
        - Design patterns & specifications can be applied, ideally considered
            as part of the architecture.
        - Iterative design & implementation allows for the design to be as
            simple as possible, and revised as necessary.
- The design is also part of the solution.
    - Ideally, the design consists of documents and specifications.
    - Elements of the design can be repurposed for documentation & testing.
    - The design can be a living document that is updated as the project
        evolves.
- In this process, the design is assumed to be continuously iterative.
    - During and after each iterative implementation, the design is reviewed
        and updated as needed.
    - Implementation can uncover unforeseen design problems that may be missed,
        through no fault of the designer.
    - Upfront collaboration between design and implementation can reduce the
        scale & complexity of revisions.
- The solution design may uncover issues with the existing plan architecture.
    - If the design is not compatible with the architecture, the architecture
        may be revised.
    - In turn, changes to the architecture may require changes to the
        scope, design, or implementation.
    - If the architecture and solution design can not be reconciled, the
        project goals, values, and validation may need to be revisited.

### <a id="step-5">Step 5: Narrow the Scope</a>
- To make implementation more manageable, the scope can be narrowed.
    - Breaking the scope into smaller, more manageable tasks can make the
        implementation more approachable.
    - Testing and validating smaller tasks before completing the entire
        scope can reduce the risk of losing work.
    - This step makes this process compatible with popular project management
        frameworks, such as SCRUM and Kanban.
- There are many ways to narrow the scope.
    - Separating related features can provide functional and verifiable
        milestones.
    - Implementing individual layers of the architecture can provide incremental
        but still testable progress.
    - Any means of splitting the scope into individual tasks is viable, as long
        each task can be validated without first completing the remaining tasks.

### <a id="step-6">Step 6: Implement & Test</a>
- The implementation is highly dependant on the architecture and design.
    - The implementation will be the most variable between projects.
    - Since the implementation becomes the final product, documentation of the
        implementation is often simply the product itself.
- Ideally, implementation will depend on testing, not the other way around.
    - TTD is a common software development practice that is well suited for
        this process.
    - Determining the tests before implementation can ensure that the
        implementation is consistent with the architecture and design before
        the implementation is complete.
    - Outlining the test criteria ahead of time can also validate the
        implementation against the goal, values, and validation, both before
        and after the implementation is complete.
- The implementation is the last, but not final, step of the process.
    - While implementing, it is important to remember the current implementation
        is not the final product.
    - Once the implementation is complete, the progress will be validated and
        evaluated against the plan as it currently stands.

## <a id="finishing-a-project">Finishing a Project</a>
- At some point, the project goals, values, validation, architecture, scope,
    design, and implementation will all be aligned.
    - The level of accuracy and precision depends entirely on the project and
        the return on investment.
    - This can be determined at any point in the process, but is typically
        determined after the implementation has be validated.

### <a id="iterative-validation">Iterative Validation</a>
- First, the implementation is compared against the current, narrowed
    scope.
    - If the implementation is not consistent with the scope, the
        implementation may need to be revised.
- Next, the implementation is compared to the entire project scope.
    - If the project scope is not complete, then another iteration of the
        the deign and implementation process can be started.
- Finally, the implementation is validated against the goal and values.
    - If the implementation does not meet the goal or values, then the
        the entire plan may be revisited.
    - The validity of the project scope is first be evaluated.
    - It is also possible that the goal or values may not have been
        properly aligned with the projects intention.

### <a id="moving-on">Death is Only the Beginning</a>
- What happens next depends on the project.
    - The software may be released and maintained, but the next project may be
        separate from the current assets.
    - New projects may be started as another iteration of the same software.
        - The future goals from previous project iteration can be used to
            define, or otherwise inform, the next project.
- Even failure is a valid outcome.
    - Sometimes projects end without achieving the goal; projects are often
        abandoned for a variety of reasons.
    - Some projects may not even get past the early stages of planning.
    - Knowledge and experience gained from the project, or just the planning
        process, is valuable to future endeavors.
- Project management is a tool.
    - Just like the entire process, the next steps are not prescribed.
    - Projects are a good way to define your progress and priorities, but
        they cannot describe everything you do.
    - What happens before, during, or after a project is up to you.
