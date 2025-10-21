# Abstract
- To accompany the first version of my personal portfolio and weblog, I have
    developed a continuous integration and continuous deployment (CI/CD)
    pipeline to deliver the application.
- This pipeline includes containerization, provisioning cloud infrastructure,
    automated testing, and deployment of observability solution.
- This is my first project exploring common DevOps tools and techniques.

# DevLog: Exploring CI/CD for a Modern (SvelteKit) Web Application
- I have been [building a SvelteKit application] for my [personal portfolio] and
    weblog.
    - The application is a [SvelteKit] template [server-side rendering (SSR)]
        web application that is [populated using static JSON files].
- I am also developing a [CI/CD pipeline] to automatically build, test, and
    deploy the application to a [cloud server].
    - The goal is to reduce common release and deployment tasks to a single
        action.
    - Scripting these tasks reduces personal effort and ensures operational
        consistency.

## Putting the Pieces Together
- My main goal was to create a self-contained project that could be easily
    shared and customized.
    - I took care to consider the best platform-agnostic tools to use.
    - Minimizing dependencies would make the project easier to maintain and
        environments easier to replicate.
- I chose to keep [this CI/CD pipeline] separate and decoupled from the
    [application code-base].
    - This will allow me to change the deployment process without needing to
        apply changes to the application code repository.
    - Separating concerns allows the application to only provide the necessary
        APIs for the deployment process.
- It proved beneficial to keep the two repositories separate.
    - Most operations were specific to the architecture prescribed by the
        CI/CD pipeline.
    - I was not sure if all template customization could be applied post-build,
        so a build process was included in the pipeline.

### Following the Script
- The pipeline logic was implement with [Bash] to make the CI/CD pipeline
    reasonably portable.
- Creating a set of Bash scripts entry points defines a clear interface for
    interacting with the pipeline.
    - Each script is responsible for a specific task.
    - Shared logic is abstracted into common functions; tasks may be composed of
        other tasks.
- Determining the correct interface and task breakdown took several iterations.
    - The [use-case diagram] illustrates the comprehensive functionality
        provided by the pipeline.
    - Documentation of the full CLI is provided in the repository [README].

#### Bash With Bats
- The Bash scripts implemented a significant amount of logic.
    - Maintaining the pipeline would not be feasible without automated tests.
- I chose to use [Bash Automated Testing System (Bats)] to implement unit tests.
    - The Bats test suites are executed as part of the CI/CD pipeline to verify
        pipeline functionality before deployment.
- Bats doesn't provide advanced mocking or stubbing functionality.
    - I took a bit of time to implement a simple mocking framework the using
        Bash function shadowing.
    - The framework also stores call arguments to a temporary file to test
        mocked dependencies are invoked correctly.

### I Could Hardly Contain Myself
- Previously, I had largely deployed applications as static files or within
   runtime virtual environments.
- [Docker] provides an alternative deployment strategy that packages the
    application and its dependencies into a single filesystem [image].
    - The image can be deployed to a sandbox [container] that runs in isolation
       from the rest of the host system.
- Aside from simplifying the deployment of the application, using Docker had
    other advantages:
    - Additional software dependencies could be easily installed on the host
        machine in separate containers.
    - Pipeline dependencies could also be easily installed without affecting the
        pipeline's environment,
        - e.g. Bats.
- In retrospect, it would be ideal to containerize the CI/CD pipeline itself to
    simplify setting up the pipeline environment.
    - However, to implement all the desired features of the pipeline the
        containerized pipeline would need to be able to run Docker containers
        itself.

### Terraform the Cloud
- Another technology I had not previously used was
    [Infrastructure as Code (IaC)].
    - In the past, all of my cloud infrastructure was manually configured using
        [SSH] connections.
    - IaC tools like [Terraform] provide a way to declarative define cloud
        infrastructure resources and provision them automatically.
- I chose to use Terraform because it is one of the most popular IaC tools with
    the broadest support for cloud providers.

#### Pie in the Sky
- Provisioning cloud infrastructure requires a cloud service provider.
    - I chose to use:
        - [DigitalOcean], for cloud virtual machines;
        - [Cloudflare], for DNS management;
        - [Google Cloud Services], for persistent storage.
    - These providers were chosen because I was already registered with each
        service.

#### Too Good to Be Simple
- Integrating a [Terraform] project into the CI/CD pipeline was overall very
    simple.
    - Since Terraform is a command line tool, it could be invoked directly from
        the pipeline scripts.
    - Terraform has a testing framework to verify infrastructure code.
    - Since cloud initialization can be done with a Bash script, the provisioned
        initialization script could be tested with [Bats] as well.
    - Separating state files made it easy to manage multiple concurrent
        environments.
- However, I did encounter a significant challenge provisioning configuration.
    - Terraform [does not fully support provisioning resources through remote
        connections].
        - *i.e.* transferring files and variables from the pipeline environment
           to the cloud resources is discouraged.
    - The recommended approach is to use cloud provider specific configuration
        management tools.
- I chose to still directly provision configuration using Terraform's
    [file] and [remote-exec] provisioners.
    - Introducing a configuration management tool would have added significant
        overhead to the project.
- The solution itself is straightforward; the problem is reliability.
    - Remote execution is unreliable and there are no testing tools to verify
        the configuration.
- I added a [runtime verification] to the provisioned virtual machine to test
    that the configuration files were correctly deployed before finalizing the
    deployment.
    - The deployment could still fail unexpectedly, but this is this assures
        that a successful deployment is configured as expected.
- The only element of the infrastructure that could not be tested was the
    [trigger] that provisions and tests the configuration.
    - This limitation precludes this provisioning strategy from being a
        sufficiently reliable deployment solution.
    - Fully imaging the virtual machine with a pre-configured application would
        be a more robust solution.

### Stacking Containers
- The rest of the software stack were integrated together on the cloud server.
    - Most applications can be installed via the [Docker package registry].
    - The containerized applications can communicate with each other over a
        private Docker network.

#### Serving the Web
- A node server can directly serve the [SvelteKit] application.
    - However, handling [TLS termination] and static file serving is not
        straightforward with a [node server].
    - A [reverse proxy web server] can handle these tasks and forward
        application requests to the node server.
- When browsing web server options, I discovered [Caddy].
    - The major advantage of Caddy is its automatic TLS
        [certificate management].

#### Phoning It In
- One quickly learns that needing to [SSH] into a server to check logs is
    tedious.
    - Centralized log collection makes it easy to monitor application activity.
    - Offloading log storage reduces maintenance and ensures logs are not lost
        in the event of a server failure.
- I chose to use [Grafana Cloud] to collect and visualize application logs.
    - [Loki] is a ubiquitous log aggregation system that integrates naively with
        Grafana Cloud to immediately get managed visualization and archival.
    - The software is open-source and it's easy to export logs to persistent
        storage to store and migrate data.
    - Grafana also provides [Grafana Alloy] to collect and forward logs from the
        application to the Loki backend.
    - Alloy was easy to install as another Docker container on the server.
- The application also has error monitoring, built in with [Sentry].
    - So as long as the server environment is configured with the Sentry
        credentials, the application will automatically report errors.

### Taking Action
- The final element of the CI/CD pipeline is automation.
    - Ideally, the code repository should deliver itself to production with
        no manual intervention.
    - A centralized CI/CD service can monitor the automation to provide
        integration and deployment feedback.
- [GitHub Actions] is a popular CI/CD service because of its tight integration
    with [GitHub] repositories.
    - [YAML] configuration files in the repository define workflows that can
        be executed manually or triggered by repository events.
    - GitHub Actions provides hosted environments that can be configured with
        the dependencies and secrets to execute scripts.

#### Countering Compute Time
- I wanted to avoid dependency on a third-party CI/CD service.
    - [Vendor lock-in] can make make maintenance and migration difficult.
    - Running the CI/CD pipeline operations during development can quickly
        become costly with hosted CI/CD services.
- Abstracting as much logic as possible into the [Bash scripts] made it easy to
    run the pipeline both locally and in GitHub Actions.
    - Creating a simple GitHub Actions workflow wrapper around the Bash scripts
        minimized integration effort and maximized portability.
    - The GitHub Actions workflows are only responsible for setting up the
        environment and invoking the pipeline scripts.

#### Testing the Test Runner Tests
- It's difficult to know where to draw the line when testing a CI/CD pipeline.
    - Ideally, the automation that is responsible for deploying the application
        should be completely tested as well.
    - However, closing the loop to ensure test automation is tested often
        requires additional tooling.
- GitHub Actions do not provide an official testing framework and third-party
    solutions are limited.
    - [Act] is a popular open-source tool to run GitHub Actions workflows
        locally, but doesn't itself provide testing functionality.
- To solve this problem, I implemented a self-reflective testing strategy to
    verify the workflows call the correct scripts.
    - I added [an addition workflow] that runs each of the other workflows
        without actually executing any deployment steps (*i.e.* a [dry run]).
    - Instead of executing the deployment scripts, the scripts return a hash
        of their input arguments that can be verified against expected values.
- Similar to the problem of testing Terraform provisioning triggers, there's no
    way to fully verify the trigger conditions of the workflows.
    - Abstracting the workflow logic into portable [actions] might provide a way
        to fully test the workflow triggers.

## An Ambitious Task From End to End
- The process must deliver a deployed application from a change in source code
    to qualify as a pipeline.
    - Basic requirements include:
        - building the application,
        - automated testing,
        - and deployment to publicly accessible hosting.
    - My ambitions included:
        - automated code quality checks,
        - concurrent cloud pipeline execution,
        - concurrent environment deployment,
        - collecting and visualizing application logs.
- My ambition to create a robust and modern continuous integration CI/CD
    pipeline distracted me from focusing on a [minimal viable product (MVP)].
    - The anxiety of solving all the design challenges up-front let me to
        extend the initial version of the pipeline.
    - Exploring these problems in a throw-away prototype would have let me
        defer less essential features.

### From One Project to the Next
- Generally, this [first iteration] of the project was a success.
    - The CI/CD pipeline is functional and handles all the necessary tasks to
        develop and maintain the application.
    - Most importantly, I learned a lot about tools and techniques that I had
        not been comfortable using.
- The outstanding issues encountered during development can be addressed in
    future iterations.
    - Generating a virtual machine image with a pre-configured application could
        improve reliability and remove the need for remote provisioning.
    - Abstracting the GitHub Actions workflows into a more reusable format would
        at least abstract away the testing issues.
- The biggest problem with this solution is its lack of generality.
    - The collection of tools used in the deployment are fixed withing the
        pipeline code.
    - The time investment to develop this pipeline is very high for a single
        application.
- This has inspired me to consider developing a more general CI/CD framework to
    use for future projects.
    - I would like to explore alternative tools to Terraform that can better
        respond to dynamic infrastructure needs.
    - A more abstract scripting language might provide easier development and
        maintenance.
        - Containerizing the pipeline would remove portability concerns that
            influenced my choice to use Bash.
- In the meantime, further development of this project will continue to expand
    my understanding of dev/ops problems.

[building a SvelteKit application]: ./sveltekit.md
[personal portfolio]: https://carledwardlyons.ca
[SvelteKit]: https://svelte.dev/docs/kit/introduction
[server-side rendering (SSR)]:
    https://developer.mozilla.org/en-US/docs/Glossary/SSR
[populated using static JSON files]: ./sveltekit.md#knobs-and-dials
[CI/CD pipeline]: https://en.wikipedia.org/wiki/CI/CD
[cloud server]: https://en.wikipedia.org/wiki/Cloud_computing
[this CI/CD pipeline]: https://github.com/systemcarl/folio
[application code-base]: https://github.com/systemcarl/blank
[Bash]: https://en.wikipedia.org/wiki/Bash_(Unix_shell)
[use-case diagram]:
    https://www.figma.com/board/KlVlC2x59WcyfeWcGDwh5A/Portfolio-Plans?t=VNSCcSgmN5HuSPOm-6
[README]: https://github.com/systemcarl/folio/tree/v0.0.1#readme
[Bash Automated Testing System (Bats)]:
    https://bats-core.readthedocs.io/en/stable/
[Bats]: https://bats-core.readthedocs.io/en/stable/
[Docker]: https://www.docker.com/
[image]:
    https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/
[container]:
    https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/
[Infrastructure as Code (IaC)]:
    https://en.wikipedia.org/wiki/Infrastructure_as_code
[SSH]: https://en.wikipedia.org/wiki/Secure_Shell
[Terraform]: https://developer.hashicorp.com/terraform
[DigitalOcean]: https://www.digitalocean.com/
[Cloudflare]: https://www.cloudflare.com/
[Google Cloud Services]: https://cloud.google.com/
[does not fully support provisioning resources through remote connections]:
    https://developer.hashicorp.com/terraform/language/resources/provisioners/file
[file]:
    https://developer.hashicorp.com/terraform/language/resources/provisioners/file
[remote-exec]:
    https://developer.hashicorp.com/terraform/language/resources/provisioners/remote-exec
[runtime verification]: https://en.wikipedia.org/wiki/Runtime_verification
[trigger]:
    https://registry.terraform.io/providers/hashicorp/null/latest/docs/resources/resource#triggers-1
[Docker package registry]: https://hub.docker.com/
[TLS termination]:
    https://en.wikipedia.org/wiki/Transport_Layer_Security#TLS_termination
[node server]: https://en.wikipedia.org/wiki/Node.js
[reverse proxy web server]: https://en.wikipedia.org/wiki/Reverse_proxy
[Caddy]: https://caddyserver.com/
[certificate management]: https://caddyserver.com/docs/automatic-https
[Grafana Cloud]: https://grafana.com/products/cloud/
[Loki]: https://grafana.com/oss/loki/
[Grafana Alloy]: https://grafana.com/docs/alloy/latest/
[Sentry]: https://sentry.io/
[GitHub Actions]: https://docs.github.com/en/actions
[GitHub]: https://github.com/
[YAML]: https://en.wikipedia.org/wiki/YAML
[Vendor lock-in]: https://en.wikipedia.org/wiki/Vendor_lock-in
[Bash scripts]: #following-the-script
[Act]: https://nektosact.com/
[an addition workflow]:
    https://github.com/systemcarl/folio/blob/v0.0.1/.github/workflows/verify.yaml
[dry run]: https://en.wikipedia.org/wiki/Dry_run_(testing)
[actions]:
    https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action
[minimal viable product (MVP)]:
    https://en.wikipedia.org/wiki/Minimum_viable_product
