# DevLog: Exploring CI/CD for a Modern (SvelteKit) Web Application
Over the summer, I have been [working on a new web application] to host my
    [personal portfolio] and weblog.
The [template application] dynamically generates pages on demand,
[populated using static JSON files].
To achieve this, I've been using [SvelteKit]'s [server-side rendering (SSR)] to
    produce the HTML pages live.

To handle the non-trivial deployment process of building, bundling, and
    deploying a server-side application,
I decided to implement
    [a continuous integration and continuous deployment (CI/CD) pipeline],
automating the whole ordeal.
The goal of [this project] (and any CI/CD pipeline, I imagine) is to reduce
    common tasks into single, self-contained actions.
Implementing an automatic process to build, test, and deploy the application
    means there is no tedious, risky manual work discouraging regular updates.

## Putting the Pieces Together
True to its namesake, a CI/CD pipeline is made up many smaller pieces that
    deliver artifacts (*e.g.* code, configurations files, build outputs)
from one to the next, end-to-end.
Each piece is responsible for a specific task; every step brings the final
    product closer to delivery.
This project combines several tools and services to create a hands-off,
    repeatable process.

My goal was to make this project as self-contained and portable as possible,
so it could be easily shared and customized.
Choosing tools that were platform-agnostic and widely supported across operating
    systems was a priority.
Also minimizing external dependencies helped keep the project easier to manage,
    maintain, and replicate.
The less tools and services required, the fewer installation steps and
    environment configurations needed to get started.

Most projects choose to include the CI/CD configuration files directly within a
    main project repository that also includes the application source
    code.
However, I decided to keep the CI/CD pipeline in a separate repository to
    decouple the application from the specific CI/CD implementation I was using.
Since the application is just a template, the application could be re-deployed
    for a variety production environments,
each with their own configuration and CI/CD requirements.

This separation proved to be beneficial as both the application and CI/CD
    pipeline evolved.
Almost all of the operations were specific to the architecture prescribed by the
    CI/CD pipeline, not the application.
The only exception was the building of the SvelteKit application
— this ended up being more independent of customization than I had expected.
All the environment variables of the application are evaluated at runtime, after
    the build process,
and any template customization is fetched live from static resources.
This means that the application build artifacts can be effectively versioned
    with the [generic source code] without sacrificing flexibility.

### Following the Script
To balance portability with convenience, I chose to implement the CI/CD pipeline
    using [Bash].
The bulk of the pipeline is [defined using bash scripts] that can be run on any
    system with a compatible shell environment
(which is most systems at this point).

Bundling the the pipeline logic into a set of bash scripts clearly defines
    the pipeline actions.
Each script is responsible for a specific task in the process, combining
    shared functions, systems commands, and in many cases, other prerequisite
    scripts.
This modularity makes it easy to manage the individual steps of the pipeline
    for troubleshooting or maintenance,
while still providing a single entry point to run the whole process.

It took some trial, and a lot of error, to determine the best way to structure
    the scripts and breakdown the pipeline tasks.
The final [use-case diagram] illustrates the overall flow of the pipeline,
    describing the overall functionality provided.
The resulting command line interface (CLI) is fully documented in the project
    [README] file.

#### Bash With Bats
Since the vast majority of the pipeline logic is implemented in bash scripts,
continuing to maintain the pipeline would be a real nightmare without proper
    testing.
To properly deliver a working solution, each step must deliver a very specific
    outcome.
The smallest deviation from the expected result can cause a release to fail.

To test the bash scripts, I used [Bash Automated Testing System (Bats)], a
    testing framework for bash.
There are a few other, very similar testing frameworks for bash scripts,
    but Bats seemed like the easiest to get started with
— especially with its [easy Docker execution], which I'll come back to [later].
The pipeline Bats test suites are executed as part of the CI process to verify
    any changes to the scripts do not break the process as a whole.

Writing the tests was easy;
the only thing that Bats did not handle out-of-the-box was mocking system
    commands or other script calls.
I took a bit of time to implement [a simple mocking framework] that utilizes
    Bash function shadowing.
In addition to stubbing dependencies,
these helper functions also [store mock call counts and arguments] in a
    temporary file to [test mocked dependencies are invoked] as expected.

### I Could Hardly Contain Myself
For most of my software development career, I've been deploying my projects
    manually.
For statically-served sites made of pre-baked HTML and stand-alone JavaScript,
    this was easy enough.
When I needed a more complex setup, I would rely entirely on manually configured
    virtual environments, like [Python]'s [venv].
This was always a horrific experience filled with unexpected issues, often
    stemming from malformed commands.
Often times servers and environments were reused indefinitely to reduce setup.
This just led to more problems down the line:
strange dependency conflicts
and stale system packages with major security vulnerabilities.

It was about time I started deploying applications within containerized
    environments.
[Docker] is now a ubiquitous tool for packaging applications and something any
   modern full-stack developer should be familiar with.
By bundling up the application and its dependencies into a single filesystem
    [image],
I no longer have to worry about environment inconsistencies and every
    installation is the same command on any system with Docker installed.
Each Docker [container] runs an application (or sometimes multiple applications)
    in isolated environments, preventing conflicts with other applications
    running on the same host.

I used Docker extensively throughout this project.
Aside from containerizing and deploying my application to a production server,
Docker also provides a standardized development environment to test deployment
    configurations on my local machine.
On the production server, I also used Docker to install and run other services
    required by the application.
Docker was also useful for running pipeline dependencies without worrying about
    installation or compatibility issues (*e.g.,* [running Bats tests]).

If anything, I don't think I used Docker (or an alternate containerization
    tools) enough in this project.
In retrospect, it would be ideal to containerize the entire CI/CD pipeline
    itself to simplify the CI/CD environment.
However, to achieve this for my specific use-cases, would require Docker
    executions to themselves run other Docker containers.
This is possible using [Docker-in-Docker (DinD)] to run Docker containers
    within a running Docker container,
or by exposing the host's Docker socket to the container to run
    Docker-out-of-Docker (DooD).
Both approaches have their own drawbacks that would require careful
    consideration.
At this point, it was probably best that I didn't attempt any of this as
    things were already [too complicated].

### Terraform the Cloud
Something else I had little experience with before this project was using
    [Terraform], or any [infrastructure as code (IaC)] tools for that matter.
Setup and maintenance of web servers has always required long stressful [SSH]
    sessions,
copying and pasting artisanal shell commands to craft the perfect machine.
Again, this manual process discouraged frequent updates.
Scaling was out of the question.
Terraform instead allows infrastructure to be defined using simple,
    declarative configuration files that are magically spawned into existence
    in exchange for your cloud provider credentials.

There are a few different IaC tools available,
but Terraform has a history of wide support across many cloud providers
    and services.
It also has a certain reputation for being easy to get started with, and
    may discussions about IaC tend to revolve around Terraform.
Despite having my reservations about declarative languages and recent changes
    to the Terraform's licensing model, I decided it was at least worth a try.

#### Pie in the Sky
Terraform itself doesn't actually create or manage any infrastructure, it
    generally just converts the configuration files into API calls to your cloud provider.
So to use Terraform, you first need to setup an account with the cloud providers
    you want to use for each element of your infrastructure.
in my case, I used:
- [DigitalOcean] for hosting the application server;
- [Cloudflare] to manage DNS records and provide a public facing proxy;
- and [Google Cloud Services (GCS)] for persistent storage.

A GCS bucket wasn't necessary for the deployment itself, but to store
    [Terraform's state] files in a remote location that could be accessed by
    any pipeline execution.
There was no real reason for these specific choices beyond the fact that I
    already had accounts with these providers.

#### Too Good to Be Simple
Using [Terraform] was quite straightforward overall.
Since Terraform deployments are executed by the command line,
I could invoke Terraform commands directly from the [pipeline bash scripts].
Most of the tests were quite simple to write with Terraform's
    [testing framework] (if you can call it that).
And since cloud initialization can be done with [a Bash script],
    the provisioned initialization script could be [tested with Bats] as if
    it were part of the pipeline itself.
I was also very happy with how easy it was to
    [separate state files for different environments]
    (e.g., staging or production).

Despite the simplicity Terraform provides, I did encounter a significant
    challenge provisioning my server configuration
— copying environment files to the provisioned server.
Terraform
    [does not fully support provisioning resources through remote connections].
While possible, documented, and maintained, transferring files and variables
    from the pipeline environment to the cloud resources is discouraged.
Instead, the idea is to use a cloud provider's native secret management system
    to handle sensitive data.

In my case, this wasn't a straightforward option.
DigitalOcean doesn't have a secret management system that naively integrates
    with there virtual machines
and subscribing to a third-party secret manager would have added unnecessary
    complexity and cost to the project
(and some how I would have had to configure the provisioned server to
    authenticate with the server manager
— begging the question of how to securely configure the server).
So, I just used the [file] and [remote-exec] provisioners to copy over the
    files anyway.

It's not hard to configure these remote connection provisioners,
the problem is reliability.
Since these operations depend on the remote server being reachable over [SSH],
    any network hiccup can cause the provisioning to fail.
Moreover, since these operations are not seen as first-class citizens in
    Terraform, there isn't official support for testing a connection
    configuration.

In the end, instead of unit testing the file provisioners ahead of time,
I implemented a [runtime verification] step within the Terraform configuration
    to confirm the files were copied successfully through [a remote execution]
    before proceeding with the rest of the deployment.
On the cloud-side, I also had to add some [checks] to the initialization script
    to prevent a race condition between the file transfer and script execution.
Since this test happens during the deployment step, this doesn't prevent
    the deployment from failing unexpectedly.
However, it at least provides a clear indication of what went wrong
and assures that a successful deployment means the files were copied correctly.

The only part of the whole pipeline that I could not entirely test in some way
    was the deprovisioning of the configuration between releases.
The [trigger] argument that causes the configuration to be re-applied if the
    server a new server has been provisioned cannot be tested without running
    two full deployments.
While I understand the reasoning and necessity of this limitation with
    Terraform's plan and apply workflow,
I can't justify using a provisioning strategy I can't thoroughly test for a
    production project.
The only alternative I can imagine would be to fully image the virtual machine
    with a pre-configured application.
— something I will carefully consider before using Terraform for future
    projects.

### Stacking Containers
To fully implement a modern web application deployment, it was necessary to
    install more than just my [SvelteKit] application on the server.
Fortunately, since I was [already using Docker], I could containerize the other
    required services as well.
The [Docker package registry] makes it easy to install most applications as
    pre-configured Docker from any system with Docker installed and internet access.
These containerized applications have the benefit of being isolated from each
    other,
still with the ability to communicate over a [private Docker network].

#### Serving the Web
Technically, a SvelteKit application, or any [Node.js] application really, can
    directly serve HTTP requests.
However, this is not ideal; It's much easier and more secure to use a dedicated
    web server application as a [reverse proxy].
In this case, I needed to handle [TLS termination] for
    [hypertext transfer secure (HTTPS) connections]
 — an encrypted HTTP protocol.
SvelteKit applications are also not designed to directly serve static assets.
It's expected that static files will be bundled with the application during
    the build process,
and my website's configuration was not something I wanted to include in the
    application artifacts.

When exploring web server options, I discoverd [Caddy], which
    [automatically provisions and renews TLS certificates] using
    [Let's Encrypt].
The simple configuration made it an easy choice for setting up a reverse
    proxy with minimal configuration overhead.

#### Phoning It In
It doesn't take long to realize that keeping application logs locally on the
    server is not a great idea.
More than anything, it's horribly tedious to SSH into the server to browse
    through log files.
Exporting logs automatically from the server to a centralized logging service
    makes it much less painful to actively monitor an application.
Having a remote copy of the logs (and ideally another backup) is also a very
    good idea.
Logs can often go missing due to disk failures, application crashes, or a sneaky
    hacker helping you hide all your security breaches.

[Grafana Cloud] was a product I had been meaning to try for some time.
With a free account, you can get a hosted instance of [Loki], an open-source log
    aggregation system,
with a managed [Grafana] dashboards (also open-source) for easy visualization.
It also comes with a [plugin] to export logs to persistent storage (like GCS)
    for long-term archival, or to migrate logs to another service.
Integrating it was also quite easy using [Grafana Alloy], a lightweight log
    forwarder that I could run as another Docker container on the server.
The only effort was [configuring Alloy] to parse and format the logs to be
    optimized for Loki
— though most of this might be optional if you're not picky about how your logs
    are labelled and organized.

While system errors from the SvelteKit server are included in the aggregated
    application logs,
the real error monitoring is handled by [Sentry].
This happens within the application itself,
so as long the correct DSN is set in the environment variables,
error reports are automatically sent to Sentry's servers.

### Taking Action
The last big piece of the CI/CD pipeline is automation.
For this project, working only by myself with fairly infrequent updates,
it would have been fine to just manually trigger the pipeline using
    [the simple CLI I had created].
But since this is a project to explore CI/CD practices,
I wanted to implement some form of automated triggering.
In an ideal system, any change to the application source code should
    automatically deliver itself to production without requiring any manual
    intervention.
And having a more centralized CI/CD environment (instead of just running scripts
    locally) provides a way to monitor and manage deployments at scale.

Since [GitHub] is by far the most popular code hosting platform,
most people have at least heard [GitHub Actions].
At first I had some concerns about using a platform-specific CI/CD solution,
    worrying about portability and [vendor lock-in].
However, I did find that there are now many open-source solutions (like [Gitea])
    that can be used to run GitHub Actions workflows outside of GitHub platform
    itself.
Otherwise GitHub Actions workflow are nice because of their simplicity and
    tight integration with other Git services.
Readable [YAML] configuration files define CI/CD steps that can be triggered
    manually with control over environment variables and secrets,
or automatically on [repository events], like updates to specific files or
    branches, or in response to a pull requests.
Github also provides free hosted ["runners"] to execute workflows within a
    preconfigured, on-demand environment, which is incredibly convenient.

#### Countering Compute Time
While Github Actions can be used for free, it's not without limitations.
Only public repositories can use Github Actions without restrictions,
and even then, there are limits on the amount of compute time available
    per month before billing starts.
And while workflows can be run self-hosted runners, or through an alternate
    CI/CD platform,
these options require more setup and maintenance overhead that are ideally
    avoided.

Avoiding dependency on a specific platform, or any platform at all, was the
    primary motivation for [designing a clean CLI interface].
This way, the pipeline can be run anywhere with a Bash-compatible shell,
    including any GitHub Actions runner.
The [Github Actions workflows] for the project are therefore simply just
    wrappers for invoking the pipeline scripts.
These workflow files simply define the conditions for execution:
defining the environment variables to pass into the scripts,
and configuring the triggering events.
Some addition structure to the workflows did help simplify the workflows;
by generalizing the pipeline script workflows, chaining [workflow dispatches]
    made it easy to reuse setup between different [environment deployments].

Since the Terraform state files are stored remotely in GCS, both local and
    remote executions of the pipeline can share state, synchronizing deployment
    processes.
Otherwise it would be either very difficult (or stupidly risky) to locally
    execute commands against automated deployment environments
— resources could easily be duplicate, destroyed, or orphaned.
But with this setup, it's easy to follow up automated deployments with local
    commands to troubleshoot issues or manage resources between releases.

One other benefit of using GitHub Actions is the automatic [commit status]
    integration;
the execution status of GitHub Action runs are automatically stored within the
    repository's commit history.
To provide this feedback during local executions, I also added some simple
    [GitHub API integration] to manually update commit statuses.
While this doesn't provide the same resolution as the GitHub Actions runs
    themselves,
it standardizes the most important information between execution environments.
With a consistent way to monitor deployment status, it's easy to diagnose
    deployment issues
and other integration with other GitHub features (like pull requests) will be
    more manageable in the future.

#### Testing the Test Runner Tests
I always struggle to decide how much testing is enough, and designing a CI/CD
    pipeline has probably been the most challenging example.
It stands to reason, that if you are reliant on an automated process to test
    your solution, then the automation should also be tested thoroughly.
However, the cycle of testing the tests can potentially go on forever.
To provide a complete testing strategy, the solution often requires a clever
    way to 'close the loop', and create a test that effectively verifies itself.

Unfortunately, there are no official, off-the-shelf testing tools for GitHub
    Actions workflows.
Even third-party tools that help with local development, like [Act], don't
    provide any testing frameworks to verify workflow functionality without
    a real workflow execution.

With all the limitations and general lack of standard practices, this seemed
    like an interesting problem to explore.
If I planned to ever develop a serious CI/CD pipeline for running production
    deployments at any sort of scale,
I would need a reliable way validate a GitHub Actions workflow (or whatever
    other CI/CD platform equivalent).
After some thought and research, I found [an example] that used the artifacts
    produced by a workflow run to verify the expected outcomes of the workflow.
While my workflows don't produce any meaningful artifacts, I realized that the
    workflows can be configured to return [workflow outputs].
Since all I was concerned about was validating the CLI input and environment,
    I could [dry run] each script with a verbose flag to output a summary of
    the evaluated configuration.
To make sure secret values were not accidentally exposed in the outputs,
    the scripts returned truncated hashes of the combined variables.
The [verification workflow] dry run outputs could then be verified against the
    outputs from the expected CLI invocations.
As a bonus, this verification workflow also first verified all the environments
    (including the verification environment itself) were configured correctly,
to make troubleshooting variable misconfigurations much easier
and prevent tests from accidentally acting against production resources.

Similar to testing the Terraform triggers, I still haven't found a way to test
    the GitHub Actions workflows are dispatched correctly based on repository
    events.
This isn't as much of a concern for me as the [limitations with Terraform]
    since I'll still be checking the deployment results after each release.
No news is bad news; if the workflow fails to trigger, a missing "in progress"
    status will make it obvious the deployment was not initiated.
This could be possibly be mitigated by abstracting the workflows into reusable
    [actions] that are tested in their own isolated repositories with controlled
    triggering events.

## An Ambitious Task From End to End
Now, even if I were to say that the project only qualifies as a pipeline when it
    delivers the application from end to end — from code changes to a live
    deployment —
I still grossly overshot the idea of a [minimal viable product (MVP)].
If I were to describe the bare minimum requirements of a CI/CD pipeline, it
    would only need to include:
- building the application (though [as discussed earlier], this would better
    fall under the concern of the application template itself);
- running application and pipeline tests;
- and deploying the application to a publicly accessible host.

My [initial iteration] of this project also included:
- automated code quality checks via GitHub commit statuses;
- support for concurrent pipeline executions over multiple environments;
- and log aggregation with monitoring dashboards.

Clearly, my ambition to create a fully-featured, modern CI/CD pipeline
    *clouded* my judgment (sorry),
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
Despite all the doubt and reservations I had while working on the
    [first iteration] of the project,
the result has been truly satisfying.
Continuing to release and deploy [new versions of the application] has been a
    been trivial compared to my past experiences publishing web applications
— every update so far has gone out without a problem.
But most importantly, I learned a ton about common DevOps tools and practices
    that I had little experience with before.
I have a much better understanding at this point of what makes a good CI/CD
    solution,
and what core problems that need to be considered when deciding on a design.

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

However, by far the biggest issue with this CI/CD project, is simply it's scope.
The solution is very specifically tailored to this particular SvelteKit
    application template, and the exact tools and services I chose to
    use.[^softyy]
The pipeline logic (the abstracted processes to define the CI/CD steps) is
    mixed in with all these specific implementation details.
While this may be acceptable for certain projects where the application
    architecture generates long-term value and is unlikely to change,
it's not ideal for a small project meant for learning and experimentation.
The time invested into manage this very specific deployment process can't be
    justified when it cannot be easily adapted to other projects.

In the future, I would like to explore creating a more generalized CI/CD
    framework (probably more of a micro-framework) that can be easily extended
    to support common application architectures and deployment strategies.
This would provide a more solid foundation to build CI/CD pipelines on top of,
    reducing the amount of boilerplate and duplicated effort required
    between projects.
I think exploring alternate IaC tools that can integrate directly with
    higher-level languages (like [Pulumi]) would also be worthwhile.
After this experience, I think there may be an elegant way to create a
    generalize CI/CD library that can be containerized to provide more robust
    tools without sacrificing portability.

In the meantime, I am quite happy with the current state of the project;
I'm sure this solution will save me plenty of time and stress while I continue
    developing and improving my [personal portfolio] and weblog.
After some more time using the pipeline and better understanding its
    limitations, I will definitely revisit theses problems.
Hopefully, by the time I am ready to begin deploying more applications, with
    more complex requirements and at a larger scales,
I will be well prepared to try this again.

[^softyy]: Discussion of this projects strict coupling to the application and
    deployment architecture
came out of a conversation with fellow software developer, [Chris Adkins].

[working on a new web application]: ./sveltekit.md
[personal portfolio]: https://carledwardlyons.ca
[template application]: https://github.com/systemcarl/blank
[populated using static JSON files]: ./sveltekit.md#knobs-and-dials
[SvelteKit]: https://svelte.dev/docs/kit/introduction
[server-side rendering (SSR)]:
    https://developer.mozilla.org/en-US/docs/Glossary/SSR
[a continuous integration and continuous deployment (CI/CD) pipeline]:
    https://en.wikipedia.org/wiki/CI/CD
[this project]: https://github.com/systemcarl/folio
[generic source code]: https://github.com/systemcarl/blank
[Bash]: https://en.wikipedia.org/wiki/Bash_(Unix_shell)
[defined using bash scripts]:
    https://github.com/systemcarl/folio/tree/v0.0.1/cli
[use-case diagram]:
    https://www.figma.com/board/KlVlC2x59WcyfeWcGDwh5A/Portfolio-Plans?t=VNSCcSgmN5HuSPOm-6
[README]: https://github.com/systemcarl/folio/tree/v0.0.1#readme
[Bash Automated Testing System (Bats)]:
    https://bats-core.readthedocs.io/en/stable/
[easy Docker execution]:
    https://bats-core.readthedocs.io/en/stable/installation.html#running-bats-in-docker
[later]: #i-could-hardly-contain-myself
[a simple mocking framework]:
    https://github.com/systemcarl/folio/blob/v0.0.1/cli/tests/mocks
[store mock call counts and arguments]:
    https://github.com/systemcarl/folio/blob/v0.0.1/cli/tests/deploy.bats#L16-L43
[test mocked dependencies are invoked]:
    https://github.com/systemcarl/folio/blob/v0.0.1/cli/tests/deploy.bats#L315-L317
[Python]: https://www.python.org/
[venv]: https://docs.python.org/3/library/venv.html
[Docker]: https://www.docker.com/
[image]:
    https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/
[container]:
    https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/
[running Bats tests]: #bash-with-bats
[Docker-in-Docker (DinD)]:
    https://www.docker.com/resources/docker-in-docker-containerized-ci-workflows-dockercon-2023/
[too complicated]: #an-ambitious-task-from-end-to-end
[Terraform]: https://developer.hashicorp.com/terraform
[Infrastructure as Code (IaC)]:
    https://en.wikipedia.org/wiki/Infrastructure_as_code
[SSH]: https://en.wikipedia.org/wiki/Secure_Shell
[DigitalOcean]: https://www.digitalocean.com/
[Cloudflare]: https://www.cloudflare.com/
[Google Cloud Services (GCS)]: https://cloud.google.com/
[Terraform's state]: https://developer.hashicorp.com/terraform/language/state
[pipeline bash scripts]: #following-the-script
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
[runtime verification]: https://en.wikipedia.org/wiki/Runtime_verification
[a remote execution]:
    https://github.com/systemcarl/folio/blob/v0.0.1/infra/main.tf#L129-L137
[checks]:
    https://github.com/systemcarl/folio/blob/v0.0.1/infra/cloud-init.tftpl#L75-L91
[trigger]:
    https://registry.terraform.io/providers/hashicorp/null/latest/docs/resources/resource#triggers-1
[already using Docker]: #i-could-hardly-contain-myself
[Docker package registry]: https://www.docker.com/products/docker-hub/
[private Docker network]:
    https://docs.docker.com/network/network-tutorial-standalone/
[Node.js]: https://en.wikipedia.org/wiki/Node.js
[reverse proxy]: https://en.wikipedia.org/wiki/Reverse_proxy
[TLS termination]:
    https://en.wikipedia.org/wiki/Transport_Layer_Security#TLS_termination
[hypertext transfer secure (HTTPS) connections]:
    https://en.wikipedia.org/wiki/HTTPS
[Caddy]: https://caddyserver.com/
[automatically provisions and renews TLS certificates]:
    https://caddyserver.com/docs/automatic-https
[Let's Encrypt]: https://letsencrypt.org/
[Grafana Cloud]: https://grafana.com/products/cloud/
[Loki]: https://grafana.com/oss/loki/
[Grafana]: https://grafana.com/
[plugin]: https://grafana.com/docs/grafana-cloud/send-data/logs/export/
[Grafana Alloy]: https://grafana.com/docs/alloy/latest/
[configuring Alloy]:
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
["runners"]:
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
[GitHub API integration]:
    https://github.com/systemcarl/folio/blob/v0.0.1/cli/status
[Act]: https://nektosact.com/
[an example]:
    https://dev.to/cicirello/how-to-test-a-github-action-with-github-actions-2hag
[workflow outputs]:
    https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows#using-outputs-from-a-reusable-workflow
[dry run]: https://en.wikipedia.org/wiki/Dry_run_(testing)
[verification workflow]:
    https://github.com/systemcarl/folio/blob/v0.0.1/.github/workflows/verify.yaml
[limitations with Terraform]: #too-good-to-be-simple
[actions]:
    https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action
[minimal viable product (MVP)]:
    https://en.wikipedia.org/wiki/Minimum_viable_product
[as discussed earlier]: #putting-the-pieces-together
[initial iteration]: https://github.com/systemcarl/folio/tree/v0.0.1
[first iteration]: https://github.com/systemcarl/folio/tree/v0.0.1
[new versions of the application]: ./content-template.md
[Pulumi]: https://www.pulumi.com/
[personal portfolio]: https://carledwardlyons.ca
[Chris Adkins]: https://www.cjadkins.com/
