# DevLog: Exploring CI/CD for a Modern (SvelteKit) Web Application
Over the summer, [I have been working on a new web application] to host my
    [personal portfolio] and weblog.
[The application I've been developing] dynamically generates pages on demand,
    which are [populated and styled using static JSON files].
To achieve this, I've been using [SvelteKit]'s [server-side rendering (SSR)] to
    produce the HTML pages live.
Unlike a traditional, statically-served website,
deploying a server-side application requires the application to be
    built, installed, and run (and remain running) on a production server.
To handle this non-trivial deployment process, I decided to implement
    a [continuous integration and continuous deployment (CI/CD) pipeline],
automating away the whole ordeal
(though admittedly, implementing a complete CI/CD pipeline is a whole ordeal in
    itself).

The goal of implementing [this CI/CD pipeline, to which I'm referring]
    (and any CI/CD pipeline, I imagine) is to reduce common tasks into single,
    self-contained actions.
If I were to develop and maintain my personal website without a CI/CD pipeline,
every update to the application code would need to be followed by a tedious
    series of unreliable, *ad hoc* steps,
probably including and surely not  limited to:
1. Executing several commands in a terminal to test and build the application;
2. Copying the built application to a publicly accessible server;
3. Fiddling with server configuration to install new dependencies or curate
    environment variables;
4. Manually verifying you didn't somehow mess it all up; and
5. Realizing you forgot to uncomment a critical line of production code before
    building the application.

Instead, implementing an automatic process to build, test, and deploy the
    application means there is no tedious, risky manual work discouraging
    regular updates.
Automation also makes it possible to institute more intensive processes, like
    deploying each update to a brand new, freshly updated server instance
— something that would be insane to do by hand, but is essential for maintaining
    a secure, reliable application.

Aside from the obvious benefits of automation,
implementing a CI/CD pipeline is also a great way to learn about the blessings
    and pitfalls of modern software deployment.
My humble SvelteKit application, meant simply for hosting my personal website,
    likely doesn't warrant the complexity of a full-fledged CI/CD pipeline.
However, it is the perfect sandbox to explore and experiment with different
    tools and practices — to satisfy my own curiosity, if nothing else.

## Putting the Pieces Together
True to its namesake, a CI/CD pipeline is made up of many smaller pieces that
    deliver artifacts (*e.g.*, code, configuration files, build outputs)
from one segment to the next, end-to-end.
Each piece is responsible for a specific task; every step brings the final
    product closer to delivery.
This project, [the CI/CD pipeline solution I implemented]
    for [my personal website],
combines several tools and services to create a hands-off, repeatable process
    for deploying the application.
Combining command-line scripting, containerization, infrastructure as code,
    log aggregation, and platform automation,
the pipeline provides a tractable means for managing the entire deployment
    lifecycle with a balance between control and convenience.

### Sharing is Caring
When selecting each tool and service, one of my goals was to make the pipeline
    as self-contained and portable as possible,
so the pipeline might be useful to other software developers or otherwise
    easily adapted to other projects.
Choosing tools that were platform-agnostic and widely supported across operating
    systems was a priority.
Ideally, each piece of the pipeline could be run on any UNIX-like system
    — even Windows, via [Windows Subsystem for Linux (WSL)].
Minimizing external dependencies also helped keep the pipeline easier to manage,
    maintain, and replicate for use in other projects.
With less tools and services required, there are fewer installation steps and
    environment configurations needed to get started or make changes.

Most projects choose to include the CI/CD scripts and configuration files
    that make up the pipeline
directly within the same repository as the main application source code.
However, I decided to keep the CI/CD pipeline files separate, in their own
    repository,
to decouple the application from the specific CI/CD implementation I have been
    developing.
Since [the application is effectively just a template]
    — the content and style all being loaded in live, for each request —
the application could be re-used for a host of different purposes
    (*e.g.*, personal portfolios, blogs, documentation sites, *etc.*).
Every deployment, for each different purpose, would likely require its own
    deployment configuration and CI/CD requirements
that my personalized CI/CD pipeline would not be able to satisfy.
Besides these potential different purposes,
    the separation proved to be beneficial for my own project as both my
    application and CI/CD pipeline evolved.
Almost all of the operations were specific to the architecture prescribed by the
    CI/CD pipeline, not the application.

The only step that did not end up being specific to the CI/CD pipeline and
    deployment customization (*i.e.* content, themes, *etc.*)
    was building the SvelteKit application from the source code;
    in the end, the build step was more independent of customization than I had
    expected.
All customization comes from static resources that are fetched live.
This means that the resulting artifacts from the application build can be
    versioned with [the application source code]
without sacrificing any flexibility.
In the future, I will likely migrate the application build process from the
    pipeline repository into the application repository.
Building from the application repository would inherently bind the version of
    the build artifacts to the corresponding application source version,
and effectively separate the continuous integration (CI) from the
    continuous deployment (CD).

### Following the Script
To balance portability with convenience, I chose to implement the majority of
    my CI/CD pipeline using [Bash].
The bulk of the pipeline is [defined by a collection of Bash scripts] that can
    be run on any system with a compatible shell environment
(which is most systems at this point).

Bundling the pipeline logic into a set of Bash scripts clearly defines the
    pipeline actions.
I built each script to be responsible for a specific step in the pipeline
    process,
combining shared functions, systems commands, and in many cases, other
    prerequisite scripts.
This modularity makes it easy to manage the individual steps of the pipeline
    for troubleshooting or maintenance;
running a single script executes a specific step of the process, without
    requiring the rest of the pipeline to be executed.

It took some trial and error (a lot of error) to determine the best way to
    structure and arrange the Bash scripts and break down the pipeline tasks.
This exercise in software design gave me a much deeper appreciation for the
    scope and complexity of CI/CD and [DevOps] in general.
The final [CI/CD pipeline use-case diagram] illustrates the overall flow of the
    pipeline, describing the overall functionality provided.
Documentation for the resulting command line interface (CLI) can be found in
    [the project README file].

#### Bash With Bats
Since the vast majority of the pipeline logic is implemented in Bash scripts,
continuing to maintain the pipeline would be a real nightmare without some way
    to continuously verify the scripts work as intended.
To properly deliver a working solution, each step must deliver a very specific
    outcome;
the smallest deviation from the expected result can cause a deployment to fail.
Thorough testing is needed to ensure that each step of the pipeline delivers the
    necessary results to proceed to the next step,
or at least fails with clear, actionable feedback.

To test the Bash scripts, I used [Bash Automated Testing System (Bats)], a
    testing framework for Bash.
There are a few other, very similar testing frameworks for Bash scripts,
    but Bats seemed like the easiest to get started with
— especially with [its easy Docker execution], which
    [I'll come back to in a minute].
The Bats tests for the pipeline are executed as part of the CI process to verify
    that any changes to the scripts do not break the process as a whole.

Writing the tests was easy.
The only thing that Bats did not handle out-of-the-box was mocking system
    commands or other script calls.
To handle this,
    I took a bit of time to implement [a simple mocking framework] that utilizes
    Bash function shadowing.
In addition to stubbing dependencies,
these helper functions also [store mock call counts and arguments] in a
    temporary file to [test mocked dependencies are invoked] as expected.

### I Could Hardly Contain Myself
For most of [my software development career], I've been deploying my projects
    manually, copying files and typing out every system command.
For statically-served sites made of pre-baked HTML and stand-alone JavaScript,
    manual deployment is easy enough:
1. Install a web server.
2. Copy the files to the server.

However, anything more complex than a simple static site quickly becomes a
    challenge to manage.
In the past, when I needed to set up a more complex environment, I would rely
    entirely on one-off virtual runtime environments, like [Python]'s [venv].
Runtime environments help isolate runtime dependencies (like Python packages)
    from other applications using similar dependencies (however, they still
    interact directly with the host system).
This worked well-enough at first, but later updates to the deployments became a
    horrific experience filled with unexpected issues, often stemming from
    malformed commands.
On their own, these environments can be time consuming to set up.
Due to the time required for setup, I'd reuse them indefinitely,
leading to problems over time;
unexpected dependency conflicts within the runtime environments and stale
    system packages with major security vulnerabilities
regularly crippled production environments.

It was about time I stopped relying on these fragile, manual processes and
    started containerizing my applications.
[Containerization] is a powerful strategy for packaging and deploying
    applications.
It allows applications to be bundled with all their dependencies in a relatively
    lightweight, portable format
    that can be run on just about any system with a compatible container engine
    installed.
In [my CI/CD pipeline], I used [Docker] to containerize and deploy my
    SvelteKit application.
By bundling up the application and its dependencies into a single
    [Docker image],
I no longer have to worry about inconsistencies within server environments and
    every installation is the same command on any system with Docker installed.
Each Docker [container] runs an application (or sometimes multiple applications)
    in isolated environments, preventing conflicts with other applications
    running on the same host.

I used Docker extensively throughout this project.
In addition to containerizing and deploying my application to a production
    server,
Docker also provides a standardized development environment to test deployment
    configurations on my local machine.
On the production server, I also used Docker to install and run other services
    required by the application.
Docker was also useful for running pipeline dependencies without worrying about
    system installation or compatibility issues
(*e.g.*, [I ran Bats tests] in a Docker container, without needing to manually
    install Bats via a package manager or from source).

If anything, I don't think I used containerization enough in this project.
In retrospect, it would be ideal to containerize the entire CI/CD pipeline
    itself as an image to simplify setup of the CI/CD environment
(the environment used to create and manage the testing and production
    environments).
However, to achieve this for my specific use-cases, it would require Docker
    executions to themselves run other Docker containers.
It is possible by running Docker containers within a running Docker container,
often referred to as *[Docker-in-Docker (DinD)]*,
or by exposing the host's Docker engine to the container,
    *Docker-out-of-Docker (DooD)*.
Both approaches have their own drawbacks that would require careful
    consideration.
At this point, it was probably best that I didn't attempt any of this as
    [things were already getting too complicated].

### Terraform the Cloud
Something else I had little experience with before this project was using
    [Terraform], or any [infrastructure as code (IaC)] tools for that matter.
Without an IaC tool, the
    setup and maintenance of web servers has always required long stressful
    [secure shell (SSH)] sessions,
copying and pasting artisanal shell commands to craft the perfect machine.
Again, this manual process discouraged frequent updates;
scaling was out of the question.
Terraform instead allows infrastructure to be defined using simple,
    declarative configuration files that magically spawn virtual machines
    into existence in exchange for your cloud provider credentials.

There are a few different IaC tools available,
but Terraform has a history of wide support across many cloud providers
    and services.
It also has a certain reputation for being easy to get started with, and
    many discussions about IaC tend to revolve around Terraform.
Despite having my reservations about declarative languages
    (I just prefer a level of transparency and control that declarative tools
    rarely provide),
and the recent changes to the Terraform's licensing model
    (downgrading it from open-source to source-available),
I decided it was at least worth a try.

#### Pie in the Sky
Terraform itself doesn't actually create or manage any infrastructure, it
    generally just converts the configuration files into API calls to your cloud
    provider.
So to use Terraform, I had accounts with cloud
    providers I wanted to use for each element of my infrastructure.
In this case, I used:
- [DigitalOcean] for hosting the application server;
- [Cloudflare] to manage [Domain Name System (DNS) records] and provide a public
    facing proxy; and
- [Google Cloud Services (GCS)] for persistent storage.

A GCS bucket wasn't necessary for the deployment itself, but rather to store
    [Terraform's state] files in a remote location that could be accessed by
    any pipeline execution.
There was no real reason for these specific choices beyond the fact that I
    already had accounts with these providers.

#### Too Good to Be Simple
Using [Terraform] was quite straightforward overall.
Since Terraform deployments are executed by the command line,
I could invoke Terraform commands directly from the [pipeline Bash scripts].
Most of the tests were quite simple to write with Terraform's
    [testing framework] (if you can really call it a framework).
And since cloud initialization can be done with [a Bash script],
    the provisioned initialization script could be [tested with Bats] as if
    it were part of the pipeline itself.
I was also very happy with how easy it was to
    [separate state files for different environments]
    (*e.g.*, staging or production).

Despite the simplicity Terraform provides, I did encounter a significant
    challenge provisioning my server configuration:
copying environment files to the provisioned server.
Terraform
    [does not fully support provisioning resources through remote connections].
While possible, documented, and maintained, transferring files and variables
    from the pipeline environment to the cloud resources is discouraged.
Instead, the idea is to use a cloud provider's native secret management system
    to handle sensitive data.

In my case, this wasn't a straightforward option.
DigitalOcean doesn't have a secret management system that naively integrates
    with their virtual machines.
Another option, subscribing to a third-party secret manager,
    would have added unnecessary
    complexity and cost to the project
(and somehow I would have had to configure the provisioned server to
    authenticate with the server manager
— begging the question of how to securely configure the server).
So, I just used the [file] and [remote-exec] provisioners to copy over the
    files anyway.

It's not hard to configure these file and remote-exec provisioners.
The problem is reliability.
Since these operations depend on the remote server being reachable over [SSH],
    any network hiccup can cause the provisioner to fail.
Moreover, since these operations are not seen as first-class citizens in
    Terraform, there isn't official support for testing a connection
    configuration.

In the end, instead of unit testing the file provisioners ahead of time,
I implemented a [runtime verification] step within the Terraform configuration.
During the deployment, this verification step confirms the files are copied
    successfully through [a remote execution].
    The deployment only proceeds if the file verification is successful.
This meant I also had to add some [checks] to the initialization script, on the
    cloud side, to prevent a race condition between the file transfer
    verification and the finalization of the server setup;
it's expected that the files will be copied long before the server setup
    finalizes and starts the application,
but a deliberate check is still necessary to ensure the files are present before
    proceeding.
Since the runtime verification happens mid-deployment, it doesn't prevent the
    deployment from failing unexpectedly (because, by that point, the deployment
    is already in progress, and an aborted deployment is a failed deployment).
However, it at least provides a clear indication of what went wrong
and assures that a deployment is only deemed successful if the files were
    actually copied correctly.

Using this runtime verification, in lieu of pre-deployment testing,
the only part of the whole pipeline that I still could not entirely test was the
    re-provisioning of the configuration files between releases.
The [trigger argument] that causes the configuration to be re-applied if a new
    server has been provisioned cannot be tested without actually running two
    full, live deployments to verify rollover between deployed versions.
While I understand the reasoning and necessity of this limitation, considering
    Terraform's plan and apply workflow,
I can't justify using a provisioning strategy I can't thoroughly test for a
    production project.
The only alternative I can imagine would be to fully image the virtual machine
    with a pre-configured application
— something I will carefully consider before using Terraform for future
    projects.

### Stacking Containers
To fully implement a modern web application deployment, I had to
    install more than just my [SvelteKit] application on the server.
I needed something more robust than the built-in Node.js server application to
    effectively handle incoming requests and monitor application health and
    performance.
Fortunately, since [I was already using Docker], I could containerize the other
    required services as well.
The [Docker package registry] makes it easy to install most applications as
    pre-configured Docker containers from any system with Docker installed and
    with access to the internet.
A ready-to-use application image can be
    [downloaded and run with a single command].
These containerized applications have the benefit of being isolated to prevent
    conflicts with each other,
but still have the ability to communicate — with each other, the host, or the
    internet —
through [Docker's internal networking interface].
The final steps of the server initialization script starts a container to run
    my SvelteKit application,
and starts containers for other necessary services not pre-installed on the
    Linux server.

#### Serving the Web
Technically, a SvelteKit application, or any [Node.js] application really, can
    directly serve [Hypertext Transfer Protocol (HTTP)] requests.
However, this is not ideal; it's much easier and more secure to use a dedicated
    web server application as a [reverse proxy].
In this case, I needed to handle [Transport Layer Security (TLS) termination]
    for [Hypertext Transfer Protocol Secure (HTTPS) connections]
 — an encrypted HTTP protocol.
SvelteKit applications are also not designed to directly serve static assets.
It's expected that static files will be bundled with the application during
    the build process,
and my website's configuration
    [was not something I wanted to include in the application artifacts].

Instead, I explored web server options and discoverd [Caddy], which
    [automatically provisions and renews TLS certificates] using
    [Let's Encrypt].
The simple configuration made it an easy choice for setting up a reverse
    proxy with minimal configuration overhead.

#### Phoning It In
It doesn't take long to realize that storing application logs on the server is
    not a great idea.
More than anything, it's horribly tedious to access the log files remotely,
    either by "SSH-ing" into the host machine or by copying them out manually.
Exporting logs automatically from the server to a centralized logging service
    makes it much less painful to actively monitor an application.
Having another copy of the logs (and ideally, yet another backup) is also a very
    good idea.
Logs can often go missing due to disk failures, application crashes, or a sneaky
    hacker helping you hide all your security breaches.

[Grafana Cloud] was another product I had been meaning to try for some time.
I set up a Grafana Cloud account to outsource my log aggregation and monitoring.
With a free account, you can get a hosted instance of [Loki], an open-source log
    aggregation system,
with managed [Grafana] dashboards (also open-source) for easy visualization.
It also comes with a [plugin] to export logs to persistent storage (like GCS)
    for long-term archival, or to migrate logs to another service.
Integrating Loki was also quite easy using [Grafana Alloy], a lightweight log
    forwarder that I simply ran as another Docker container on the server.
The only sizeable effort was [configuring Alloy to parse and format the logs] to be
    optimized for Loki
— though most of this might be optional if you're not picky about how your logs
    are labelled and organized.

While system errors from the SvelteKit server are included in the aggregated
    application logs sent to Grafana Cloud,
the real error monitoring is handled by [Sentry].
I installed Sentry within the application itself,
so as long the correct Sentry Data Source Name (DSN; the unique identifier for a
    Sentry project) is set in the environment variables,
error reports are automatically sent to Sentry's servers and can be viewed
    through their portal.

### Taking Action
The last big piece of the CI/CD pipeline was automation.
For this project, working only by myself with fairly infrequent updates,
it would have been fine to just manually trigger the pipeline using
    [the simple CLI I had created].
But since this is a project to explore CI/CD practices,
I wanted to implement some form of automated triggering.
This was to create an ideal system,
    in which any change to the application source code
    automatically delivers itself to production without requiring any human
    intervention.
Also, having a more centralized CI/CD environment (instead of just running
    scripts locally) provides a way to monitor and manage deployments as the
    project scales to host more content and serve more users.

After some internal deliberation, I decided to use [GitHub Actions] as the CI/CD
    platform to automate the pipeline execution.
Since [GitHub] is by far the most popular code hosting platform,
most developers have at least heard of [GitHub Actions].
At first I had some concerns about using a platform-specific CI/CD solution
    (like GitHub Actions), as I worried about portability and [vendor lock-in].
However, I did find that there are now many open-source solutions (like [Gitea])
    that can be used to run GitHub Actions workflows outside of the GitHub
    platform itself.
Aside from my (arguably undue) concerns,
GitHub Actions workflows are nice because of their simplicity
and tight integration with other services on the GitHub platform.
Readable [YAML] configuration files define workflows that can be triggered
    manually with control over environment variables and secrets,
or automatically on [repository events], like updates to specific files or
    branches, or in response to a pull request.
GitHub Actions also provides free [hosted runners] to execute workflows within
    a preconfigured, on-demand environment, which is incredibly convenient.

#### Countering Compute Time
While Github Actions can be used for free, it's not without limitations.
Only public repositories can use Github Actions without restrictions,
and even then, there are limits on the amount of compute time available
    per month before billing starts.
And while Github Actions workflows can be run on self-hosted runners, or through
    another compatible CI/CD platform,
these options require more setup and maintenance overhead that are ideally
    avoided.
Many of the choices I had made earlier in the project were to decouple my CI/CD
    pipeline from supporting CI/CD platforms and mitigate these issues.

One such choice was [designing a clean CLI interface], in which my
    primary motivation was to avoid
    unnecessary dependency on a specific platform.
With a clean, self-contained CLI, I can run the pipeline anywhere with a
    Bash-compatible shell — a GitHub Actions runner, or really any other modern
    system.
The [Github Actions workflows] for the project are therefore simply just
    wrappers for invoking my CI/CD pipeline scripts.
These workflow files simply define the conditions for execution:
defining the environment variables to pass into the scripts,
and configuring the triggering events.
Some additional structure to the workflows did help simplify them;
by generalizing the pipeline script workflows, chaining [workflow dispatches]
    made it easy to reuse setup between different [environment deployments].

The automation for my CI/CD pipeline is also completely optional. The pipeline
    can still be executed locally at any time, or even run concurrently on
    multiple CI/CD platforms.
Since the Terraform state files are stored remotely in GCS, both local and
    remote executions of the pipeline can share state, synchronizing deployment
    processes.
Otherwise it would be very difficult (or just stupidly risky) to locally
    execute commands against automated deployment environments
— resources could easily be duplicated, destroyed, or orphaned.
But with this shared state, it's easy to follow up automated deployments with
    local commands to troubleshoot issues or manage resources between releases.

One particular benefit of using GitHub Actions that I didn`t want to live
    without is the automatic [commit status] integration:
the execution status of GitHub Action runs are automatically stored within the
    repository's commit history.
To provide this execution status feedback during local executions, I needed to
    add [an additional GitHub API integration] to manually update commit
    statuses when running the pipeline outside of GitHub Actions.
While this doesn't provide the same resolution as the GitHub Actions runs
    themselves,
it standardizes the most important information between execution environments.
With a consistent way to monitor deployment status, it's easy to diagnose
    deployment issues,
and other integration with other GitHub features (like pull requests) will be
    more manageable in the future.

#### Testing the Test Runner Tests
I always struggle to decide how much testing is enough, and designing a CI/CD
    pipeline has probably been the most challenging example.
It stands to reason, that if you are reliant on an automated process to test
    your solution, then the automation should also be tested thoroughly.
However, the cycle of testing the tests can potentially go on forever.
To provide a complete testing strategy, the solution often requires a clever
    way to "close the loop", and create a test that effectively verifies itself.

Unfortunately, there are no official, off-the-shelf testing tools for GitHub
    Actions workflows.
Even third-party tools that help with local development, like [Act], don't
    provide any testing frameworks.
There's no way to easily verify workflow functionality; testing typically
    requires executing the full workflow against a full environment using real
    production resources and manually verifying the results.

With all the limitations and general lack of standard practices, this seemed
    like an interesting problem to explore.
If I planned to ever develop a serious CI/CD pipeline for running production
    deployments at any sort of scale,
I would need a reliable way to validate a GitHub Actions workflow (or whatever
    other CI/CD platform equivalent).
After some thought and research, I found [an example] that used the artifacts
    produced by a workflow run to verify the expected outcomes of the workflow.
While my workflows don't produce any meaningful artifacts, I realized that the
    workflows can be configured to return [workflow outputs].
Since all I was concerned about was validating the CLI input and environment,
    I could [dry run] each script with a verbose flag to output a summary of
    the evaluated configuration.
To make sure secret values aren't accidentally exposed in the outputs,
    the scripts return truncated hashes of the combined variables.
[The verification workflow] dry run outputs can then be verified against the
    outputs from the expected CLI invocations.
As a bonus, this verification workflow also first verifies all the environments
    (including the verification environment itself) are configured correctly,
to make troubleshooting variable misconfigurations much easier
and prevent tests from accidentally acting against production resources.

Similar to testing the Terraform triggers, I still haven't found a way to test
    the GitHub Actions workflows are dispatched correctly based on repository
    events.
This isn't as much of a concern for me as the [limitations with Terraform]
    since I'll still be checking the deployment results after each release.
No news is bad news; if the workflow fails to trigger, a missing "in progress"
    status will make it obvious the deployment was not initiated.
This testing blind spot could possibly be resolved by abstracting the workflows
    into reusable [actions]
that are tested in their own isolated repositories with controlled triggering
    events.

## An Ambitious Task From End to End
Now, even if I were to say that this project would only qualify as a pipeline if
    it delivers the application from end to end — from code changes to a live
    deployment —
I still grossly overshot the idea of a [minimal viable product (MVP)].
If I were to describe the bare minimum requirements of a CI/CD pipeline, it
    would only need to include:
- building the application (though [as discussed earlier], this would better
    fall under the concern of the application template itself);
- running application and pipeline tests; and
- deploying the application to a publicly accessible host.

My [initial iteration] of this project also included:
- automated code quality checks via GitHub commit statuses;
- support for concurrent pipeline executions over multiple environments; and
- log aggregation with monitoring dashboards.

Clearly, my ambition to create a fully-featured, modern CI/CD pipeline
    *clouded* my judgment (sorry; not sorry),
distracting me from trying to deliver a simple viable solution quickly.
Because this was my first serious attempt at designing a CI/CD pipeline
and I wanted to explore many different tools and techniques,
I was anxious to solve all of the design challenges early on.
The worries that my vision for the project was hopelessly flawed
caused me to procrastinate the initial release until the proof-of-concept was
    actually a polished, working solution.
In retrospect, to address my concerns that future features might not work as
    planned,
I should have just spun up a quick-and-dirty prototype (without all the tests
    and documentation) to prove the overall architecture was sound.
From there, I could have iteratively added the refined features incrementally,
    as needed.

#### From One Project to the Next
Despite this oversight, and all my doubt and reservations while working on the
    [initial iteration] of the project,
the result has been truly satisfying.
Continuing to release and deploy [new versions of the application] has been
    trivial compared to my past experiences publishing web applications
— every update so far has gone out without a problem.
But most importantly, I learned a ton about common DevOps tools and practices
    that I had little experience with before.
I have a much better understanding at this point of what makes a good CI/CD
    solution,
and what core problems need to be considered when deciding on a design.

At this point, I don't have much doubt that the issues still remaining with this
    pipeline can be addressed as I continue to iterate on the project.
I think that instead of continually spinning up virtual machines for each
    deployment — updating and installing dependencies for every release —
it makes much more sense to build a pre-configured virtual machine image that
    can be deployed instantly to my cloud provider.
This would greatly reduce the deployment time and also eliminate many of the
    reliability issues discussed.
I also think taking the time to abstract the GitHub Actions workflows into
    reusable [actions] would provide a better way to manage workflows in
    the long run.

However, by far, the biggest issue with this CI/CD project is simply its scope.
The solution is very specifically tailored to this particular SvelteKit
    application template, and to the exact tools and services I chose to
    use.[^softyy]
The pipeline logic (the abstracted processes to define the CI/CD steps) is
    mixed in with all these specific implementation details.
While this may be acceptable for certain projects where the application
    architecture generates long-term value and is unlikely to change,
it's not ideal for a small project meant for learning and experimentation.
The time invested into managing this very specific deployment process can't be
    justified when it cannot be easily adapted to other projects.

In the future, I would like to explore creating a more generalized CI/CD
    framework (probably more of a micro-framework) that can be easily extended
    to support common application architectures and deployment strategies.
This would provide a more solid foundation to build CI/CD pipelines, on top of
    reducing the amount of boilerplate and duplicated effort required
    between projects.
I think exploring alternate IaC tools that can integrate directly with
    higher-level languages (like [Pulumi]) would also be worthwhile.
After this experience, I think there may be an elegant way to create a
    generalized CI/CD library that can be containerized to provide more robust
    tools without sacrificing portability.

In the meantime, I am quite happy with the current state of the project;
I'm sure this solution will save me plenty of time and stress while I continue
    developing and improving my [personal portfolio] and weblog.
After some more time using the pipeline and better understanding its
    limitations, I will definitely revisit these problems.
Hopefully, by the time I am ready to begin deploying more applications, with
    more complex requirements and at a larger scale,
I will be well prepared to try this again.

[^softyy]: Discussion of this projects strict coupling to the application and
    deployment architecture
came out of a conversation with fellow software developer, [Chris Adkins].

[I have been working on a new web application]: ./sveltekit.md
[personal portfolio]: https://carledwardlyons.ca
[the application I've been developing]: https://github.com/systemcarl/blank
[populated and styled using static JSON files]: ./sveltekit.md#knobs-and-dials
[SvelteKit]: https://svelte.dev/docs/kit/introduction
[server-side rendering (SSR)]:
    https://developer.mozilla.org/en-US/docs/Glossary/SSR
[continuous integration and continuous deployment (CI/CD) pipeline]:
    https://en.wikipedia.org/wiki/CI/CD
[this CI/CD pipeline, to which I'm referring]:
    https://github.com/systemcarl/folio
[the CI/CD pipeline solution I implemented]:
    https://github.com/systemcarl/folio
[my personal website]: https://carledwardlyons.ca
[Windows Subsystem for Linux (WSL)]:
    https://docs.microsoft.com/en-us/windows/wsl/about
[the application is effectively just a template]:
    ./content-template.md
[the application source code]: https://github.com/systemcarl/blank
[Bash]: https://en.wikipedia.org/wiki/Bash_(Unix_shell)
[defined by a collection of Bash scripts]:
    https://github.com/systemcarl/folio/tree/v0.0.1/cli
[DevOps]: https://en.wikipedia.org/wiki/DevOps
[CI/CD pipeline use-case diagram]:
    https://www.figma.com/board/KlVlC2x59WcyfeWcGDwh5A/Portfolio-Plans?t=VNSCcSgmN5HuSPOm-6
[the project README file]:
    https://github.com/systemcarl/folio/tree/v0.0.1#readme
[Bash Automated Testing System (Bats)]:
    https://bats-core.readthedocs.io/en/stable/
[its easy Docker execution]:
    https://bats-core.readthedocs.io/en/stable/installation.html#running-bats-in-docker
[I'll come back to in a minute]: #i-could-hardly-contain-myself
[my software development career]: ../../about-me.md#i-love-deadlines
[a simple mocking framework]:
    https://github.com/systemcarl/folio/blob/v0.0.1/cli/tests/mocks
[store mock call counts and arguments]:
    https://github.com/systemcarl/folio/blob/v0.0.1/cli/tests/deploy.bats#L16-L43
[test mocked dependencies are invoked]:
    https://github.com/systemcarl/folio/blob/v0.0.1/cli/tests/deploy.bats#L315-L317
[Python]: https://www.python.org/
[venv]: https://docs.python.org/3/library/venv.html
[Containerization]:
    https://en.wikipedia.org/wiki/Containerization_(virtualization)
[my CI/CD pipeline]:
    #continuous-integration-and-continuous-deployment-cicd-pipeline
[Docker]: https://www.docker.com/
[Docker image]:
    https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/
[container]:
    https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/
[I ran Bats tests]: #bash-with-bats
[Docker-in-Docker (DinD)]:
    https://www.docker.com/resources/docker-in-docker-containerized-ci-workflows-dockercon-2023/
[things were already getting too complicated]:
    #an-ambitious-task-from-end-to-end
[Terraform]: https://developer.hashicorp.com/terraform
[Infrastructure as Code (IaC)]:
    https://en.wikipedia.org/wiki/Infrastructure_as_code
[secure shell (SSH)]: https://en.wikipedia.org/wiki/Secure_Shell
[DigitalOcean]: https://www.digitalocean.com/
[Cloudflare]: https://www.cloudflare.com/
[Google Cloud Services (GCS)]: https://cloud.google.com/
[Domain Name System (DNS) records]:
    https://en.wikipedia.org/wiki/Domain_Name_System#Resource_records
[Terraform's state]: https://developer.hashicorp.com/terraform/language/state
[pipeline Bash scripts]: #following-the-script
[testing framework]:
    https://developer.hashicorp.com/terraform/tutorials/configuration-language/test
[a Bash script]:
    https://github.com/systemcarl/folio/blob/v0.0.1/infra/cloud-init.tftpl
[tested with Bats]: #bash-with-bats
[separate state files for different environments]:
    https://github.com/systemcarl/folio/blob/v0.0.1/cli/deploy#L88-L89
[does not fully support provisioning resources through remote connections]:
    https://developer.hashicorp.com/terraform/language/resources/provisioners/file
[file]:
    https://developer.hashicorp.com/terraform/language/resources/provisioners/file
[remote-exec]:
    https://developer.hashicorp.com/terraform/language/resources/provisioners/remote-exec
[SSH]: https://en.wikipedia.org/wiki/Secure_Shell
[runtime verification]: https://en.wikipedia.org/wiki/Runtime_verification
[a remote execution]:
    https://github.com/systemcarl/folio/blob/v0.0.1/infra/main.tf#L129-L137
[checks]:
    https://github.com/systemcarl/folio/blob/v0.0.1/infra/cloud-init.tftpl#L75-L91
[trigger argument]:
    https://registry.terraform.io/providers/hashicorp/null/latest/docs/resources/resource#triggers-1
[I was already using Docker]: #i-could-hardly-contain-myself
[Docker package registry]: https://www.docker.com/products/docker-hub/
[downloaded and run with a single command]:
    https://docs.docker.com/get-started/introduction/get-docker-desktop/#run-your-first-container
[Docker's internal networking interface]:
    https://docs.docker.com/network/network-tutorial-standalone/
[Node.js]: https://en.wikipedia.org/wiki/Node.js
[reverse proxy]: https://en.wikipedia.org/wiki/Reverse_proxy
[Hypertext Transfer Protocol (HTTP)]:
    https://en.wikipedia.org/wiki/HTTP
[Transport Layer Security (TLS) termination]:
    https://en.wikipedia.org/wiki/Transport_Layer_Security#TLS_termination
[Hypertext Transfer Protocol Secure (HTTPS) connections]:
    https://en.wikipedia.org/wiki/HTTPS
[Caddy]: https://caddyserver.com/
[was not something I wanted to include in the application artifacts]:
    #sharing-is-caring
[automatically provisions and renews TLS certificates]:
    https://caddyserver.com/docs/automatic-https
[Let's Encrypt]: https://letsencrypt.org/
[Grafana Cloud]: https://grafana.com/products/cloud/
[Loki]: https://grafana.com/oss/loki/
[Grafana]: https://grafana.com/
[plugin]: https://grafana.com/docs/grafana-cloud/send-data/logs/export/
[Grafana Alloy]: https://grafana.com/docs/alloy/latest/
[configuring Alloy to parse and format the logs]:
    https://github.com/systemcarl/folio/blob/v0.0.1/infra/config.alloy.tftpl
[Sentry]: https://sentry.io/
[the simple CLI I had created]: #following-the-script
[GitHub]: https://github.com/
[GitHub Actions]: https://docs.github.com/en/actions
[Vendor lock-in]: https://en.wikipedia.org/wiki/Vendor_lock-in
[Gitea]: https://about.gitea.com/
[YAML]: https://en.wikipedia.org/wiki/YAML
[repository events]:
    https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows
[hosted runners]:
    https://docs.github.com/en/actions/concepts/runners/github-hosted-runners
[designing a clean CLI interface]: #following-the-script
[Github Actions workflows]:
    https://github.com/systemcarl/folio/tree/v0.0.1/.github/workflows
[workflow dispatches]:
    https://docs.github.com/en/actions/how-tos/manage-workflow-runs/manually-run-a-workflow
[environment deployments]:
    https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments
[commit status]:
    https://docs.github.com/en/rest/commits/statuses
[an additional GitHub API integration]:
    https://github.com/systemcarl/folio/blob/v0.0.1/cli/status
[Act]: https://nektosact.com/
[an example]:
    https://dev.to/cicirello/how-to-test-a-github-action-with-github-actions-2hag
[workflow outputs]:
    https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows#using-outputs-from-a-reusable-workflow
[dry run]: https://en.wikipedia.org/wiki/Dry_run_(testing)
[The verification workflow]:
    https://github.com/systemcarl/folio/blob/v0.0.1/.github/workflows/verify.yaml
[limitations with Terraform]: #too-good-to-be-simple
[actions]:
    https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action
[minimal viable product (MVP)]:
    https://en.wikipedia.org/wiki/Minimum_viable_product
[as discussed earlier]: #putting-the-pieces-together
[initial iteration]: https://github.com/systemcarl/folio/tree/v0.0.1
[new versions of the application]: ./content-template.md
[Pulumi]: https://www.pulumi.com/
[personal portfolio]: https://carledwardlyons.ca
[Chris Adkins]: https://www.cjadkins.com/
