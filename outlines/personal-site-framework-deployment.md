# Abstract
- I am currently developing a my own personal website.
- To get started, I needed to set up the necessary application dependencies and
    environment.
- Before beginning to develop the application itself, I implemented a simple
    pipeline for automating application deployment.

# Personal Website Framework & Environment
- During [preliminary planning], I chose to use [*Svelte*] & [*SvelteKit*] for
    the application framework.
    - These core dependencies must be installed before any application
        development can begin.
    - Installing & developing with Svelte in turn requires a Node.js
        environment & a package manager.
- *SvelteKit* requires a [*Node.js*] environment to run.
    - *Node.js* must be installed in a development environment and a production
        server.
    - *Node.js* alone cannot handle Secure Sockets Layer (SSL) connections.
        - A reverse proxy server is required to handle SSL connections on the
            production server.
    - Ensuring a reliable production environment installation can be tedious
        and error-prone if not automated.
- Before starting development of the application, I implemented some basic
    automation for validating, building, & deploying the application.
    - Containing the deployment process in a comprehensive project helps define
        dependencies, such as:
        - environment variables,
        - authentication tokens,
        - external services & tools.
    - Automated deployment through a continuous integration & continuous
        deployment (CI/CD) pipeline ensures the application has been tested and
        is delivered in a consistent manner.
- I kept the dev-ops automation project separate and decoupled from the
    application code.
    - This will allow me to change the deployment process without needing to
        apply changes to the application code repository.
    - Separating concerns allows the application to only provide the necessary
        APIs for the deployment process.
    - This may not be necessary for the initial development, but will be
        valuable if alternate deployment implementations are required.
- The ultimate goal was to streamline development and operations.
    - Keeping the processes simple & concise will make automation easier to
        manage as the project evolves.
    - Making the process as portable will allow the application to be
        deployed from either a local development environment or *GitHub
        Actions*.
    - The less time spent on deployment, the more time can be spent on
        developing the application.
- Building a professional deployment solution gave me more opportunities to
    learn deployment operations concepts.
    - By building a solution using only the basic industry standard tools, I
        had no choice to learn more about developer operations (dev/ops)
        fundamentals.
    - The solution I created for this simple application may serve as a
        prototype for more complex projects in the future.
    - Sharing this may also help others learn about dev/ops automation and
        building their own CI/CD pipelines.

## Creating an Application
- The core of the project is the *SvelteKit* application, itself.
    - In order to implement a build & deployment process, we need to have a
        target to build and deploy.
    - It does not require any functionality, only the basic environment,
        structure, and configuration.

### A Local Node.js Environment
- First I installed a [*Node.js*] environment.
    - Instructions for downloading & installing Node.js can be found on the
        [download page].
        - I used the most recent long-term support (LTS) version, v22.16.0.
        - Any newer version should work, but I have not tested them.
    - It is generally assumed that a Node.js environment is available on the
        development machine.
    - When installing Node.js, I also installed the [*npm*] package manager.
    - This project likely will not require installing the tools for building
        native modules.
- Once Node.js is installed, I verified the installation via the command line.
    - I use a bash shell, but the commands should be the same in other shells.
    - Checking the versions confirms the installation was successful and the
        correct environment is available.
    ```bash
    node --version
    # v22.16.0
    npm --version
    # 10.9.2
    ```

### Setting Up A Svelte Project
- With a Node.js environment and package manager installed, I can now set up a
    *SvelteKit* project.
    - I used the *SvelteKit* project creation tool, *SV*, to create a new
        project, selecting *SvelteKit minimal* with TypeScript, and adding
        *eslint* & *vitest*.
    ```bash
    npx sv create .
    ```
    - When prompted, I selected the following options:
        - SvelteKit minimal;
        - TypeScript;
        - `eslint` & `vitest`;
        - & `npm`.
    - The application initialized properly with no warnings or errors.
    ```bash
    npm run dev -- --open
    ```
- After after initializing the project, I created and initial project commit.
    - This will be the starting point for the project.
    - It is informative to have a record of the initial project template.
    - I will add all the files generated by the `sv` command, except for the
        exceptions included in the generated `.gitignore` file.
    ```bash
    git init
    git add .
    git commit
    ```
- To setup the project to be run in a self-hosted Node.js environment, I
    installed the necessary dependencies.
    - The *Node.js* adapter needed to be installed in the project.
    ```bash
    npm install --save-dev @sveltejs/adapter-node
    ```
    - After installing the adapter, I updated the `svelte.config.js` file to
        use the *Node.js* adapter.
- For convenience, I pushed the application code to a new repository on
    *GitHub*.
    - I created a new repository, `folio`, on *GitHub*.
    - I added the remote repository to the local repository and synchronized the
        initial commits.
    ```bash
    git remote add origin https://github.com/systemcarl/folio.git
    git push -u origin main
    ```

## Building & Deploying the Application
- Next I invested a significant amount of time implementing a set of build &
    deployment processes for the application.
    - Implementing the deployment pipeline was quite involved and too long
        to recount in detail.
    - The result and complete Git history is available in the
        [`folio-deploy` repository].
- The end result is a set of operational tools for managing application
    delivery.
    - The tools include:
        - a process for deploying a production build locally either locally
            or in a remote environment,
        - a process for cleaning up deployments and removing remote resources,
        - a process for testing the application and the deployment pipeline
            itself.
    - The processes needed to be automated via the *GitHub Actions* platform,
        requiring:
        - a means of tracking the current state of the integration & deployment.
- I created a separate repository for continuous integration & continuous
    deployment (CI/CD) automation.
    - It will use submodules to link the application code repository.
    - A submodule for automation code will not be necessary.
        - The automation code will be tightly coupled to the process itself.
    - In the new repository, I added the application code repository as a
        submodule.
    ```bash
    git submodule add https://github.com/systemcarl/folio.git
    git submodule update --init --recursive
    ```
    - This created the submodule folder, `folio`, and generated the
        `.gitmodules` file.

### Docker
- To simplify the deployment process, I containerized the application using
    *Docker*.
    - *Docker* is the standard for containerization.
    - It is a common practice to use a reverse proxy server to serve a
        containerized application on a cloud-hosted virtual machine (VM).
        - Alternates include:
            - using orchestration to automate container deployment,
            - or deploying serverless applications that instead abstract away
                the underlying infrastructure as a service.
        - A simple container deployed to a VM can easily be extended to be used
            in most alternate strategies.
- I installed *Docker*.
    - I downloaded the latest version of *Docker Desktop* (v4.42.0) from the
        [official download page](
        https://www.docker.com/products/docker-desktop).
    - The installation was configured using the recommended settings.
    - Once installed, I verified the installation via the command line.
    ```bash
    docker --version
    # Docker version 28.1.1, build 4eba377
    ```

#### Containerizing the Application
- With docker available, I could create a simple [Dockerfile] to build the
    application.
    - The *Dockerfile* uses the base image, copies in the build output from the
        application submodule, and configures the container to run the built
        Node.js application.
    - Additionally, the *Dockerfile* ensures the file permissions are exclusive
        to the `node` user which will be executing the application.
- To build the container, the application must be ready for deployment.
    - The `npm` dependencies must be installed, and the server application
        must be built.
    ```bash
    npm install
    npm run build
    ```
    - With the application build files in the `./build` directory, the
        *Dockerfile* can be followed.
    ```bash
    docker build -t folio:latest -f ./Dockerfile .
    ```
- To test the container is working correctly, I ran it locally.
    - I used the `docker run` command to start the container, exposing the
        connection port.
    ```bash
    docker run -d --name folio -p 3000:3000 folio:latest
    ```
- Later, this image was uploaded to the *GitHub Packages* registry for
    distribution.
    - Publishing the container image to the registry creates a source of truth
        for the application version.
    - When the container is deployed, the image can be pulled from the registry
        to the remote environment.

#### Running Development Dependencies
- To make it easier to run pipeline processes, I used containerized dependencies
    when possible.
    - Since the dependency images are readily available through the Docker
        registry, no additional installation steps will be required for these
        tools.
    - This makes one less thing to consider when porting to a cloud CI/CD
        environment.
> - In retrospect, I would have considered using a *Docker* container to run
    all of the pipeline processes.
>   - This would have made completely removed the need for any other
        environment dependencies, like *Node.js* or a *BASH* shell.
>   - May have been overkill for this project but could have been an
        interesting exercise.

### BASH
- At first I thought a simple set of independent bash scripts would be
    sufficient for automating the build & deployment process.
    - The use cases for local deployment seemed like they could be easily
        compartmentalized.
    - To create a production-ready deployment process, automated via
        GitHub Actions, required a more comprehensive solution design.
- After some iteration, I eventually determined a reasonable set of use cases
    that satisfied the desired functionality.
    - The use case diagram is available on Figma:
        [`folio-deploy` Activity Diagram].
    - Structuring my *BASH* scripts into functions around these use cases
        made the implementation match the intended use of the pipeline,
        regardless of environment.
    - The full implementation is available in the [`folio-deploy` repository].
- The BASH scripts serve as the central point for managing the application
    process.
    - To keep the project (reasonably) portable, as much logic as possible is
        kept within the *BASH* scripts.
    - The scripts provide an entry point for both local execution and automation
        via GitHub Actions.
    - The BASH scripts in turn execute the necessary deployment processes via
        the appropriate deployment tools.

#### Entry Points
- To organize the BASH logic into approachable commands, functions are wrapped
    in entry-point scripts.
    - The entry points are intended to be run from a BASH command line, or via
        *GitHub Actions*.
    - These scripts load the necessary BASH function files and help control the
        execution environment.
    - Keeping these entry points simple and concise reduces the amount of script
        to a minimum, in favour of function execution.
- For example, the `deploy` entry point is a simple script that loads the
    `deploy` function and executes it with the provided arguments.
    - The [`deploy` script] ensures the working directory is the root of the
        project, in case the script is called from another location.
    - To simplify implementation, the [`deploy` function] assumes it is being
        called from the root of the project.
- The project is structured around a set of *BASH* script entry points:
    - There are 3 main entry points for managing the application, as a whole:
        - `deploy` - deploys the application to a remote environment;
        - `destroy` - removes the application from a remote environment;
        - `test` - runs the application tests.
    - There are additional entry points for managing specific aspects of the
        deployment independently, such as:
        - `validate` - validates the application code;
        - `containerize` - builds the application container image;
        - `smoke` - runs a smoke test against a deployed application;
        - `status` - checks or updates the status of the application integration
            and deployment.
    - The remaining *GitHub Actions* use-cases for publishing and staging the
        application will be handled by extending the deployment workflow.

#### BATS
- BASH scripting is also more convenient to test than declarative configuration
    files, like *GitHub Actions* workflows.
    - I used the [BATS] testing framework to unit test the *BASH* scripts.
    - To make the tests portable, I used a *Docker* container to run the tests.
        - This was the most significant reason I chose to use BATS over other
            BASH testing frameworks.
    - The *Docker* container provides additional [BATS helper libraries], such
        as `bats-support` and `bats-assert`.
- Compartmentalizing the BASH scripts into functions also made it easy to test
    each function independently.
    - The number of test cases can scale exponentially with the complexity of an
        individual function or script.
    - Pre-loaded function definitions can be easily tested in isolation.
- For complete coverage, I tested both the entry-point scripts and the
    individual functions.
    - The [`deploy` test file] tests the `deploy` function mocking system
        dependencies and other function calls.
    - The [`cli` test file] mocks the function and the `source` command to
        prevent the entry-point scripts from re-loading the functions.

### Cloud Configuration
- Before deploying to a remote environment, the cloud accounts and credentials
    must be configured.
    - I used *Cloudflare* for DNS management.
    - *DigitalOcean* was used to provide the virtual machine for hosting the
        application.
    - To use the necessary *GitHub* features, including commit statuses and the
        *GitHub Packages* registry, appropriate permissions must be set.
    - Lastly, to keep deployment state across CI/CD environments, I created
        a *Google Cloud Services* bucket to store state files.

#### Cloudflare
- [*Cloudflare*] is a great service for managing DNS records and providing
    security features.
    - When using Cloudflare DNS as a proxy, Cloudflare acts as a reverse
        proxy for the application.
    - Cloudflare's proxy provides additional features for managing external
        traffic, including:
        - DDoS protection,
        - performance analytics.
    - Hosting behind Cloudflare's proxied DNS has no effect on development or
        operations, aside from some considerations when testing.
- To automate DNS management for my application, I created [*Cloudflare* API
    token].
    - To make the necessary changes, the API token must be able to:
        - create, update, and delete DNS records.
- To ensure the application is accessible, I update the Cloudflare configuration
    rules.
    - We want the settings to take full advantage of Cloudflare's security
        features, but still function properly.
    - In production, I will set the SSL/TSL to "Full (strict)" to ensure
        the connection is as secure as possible.
    - For testing, I set the [DNS SSL/TSL mode] to "Full" in order to not raise
        errors when using a staging SSL certificate for all subdomains.
- I also added the necessary [redirect rules] for any domain variants.
    - This will ensure that all traffic is directed to the primary domain,
        and the application, with a consistent URL.
    - I added proxied CNAME records as a placeholder to catch any traffic.
    - I used Cloudflare's page rules to respond to any request, on any
        subdomain, with a 301 redirect to the primary domain, maintaining the
        path.

#### DigitalOcean
- I used [*DigitalOcean*] to provide a virtual machine for hosting the
    application.
    - *DigitalOcean* is a popular cloud provider that's very easy to use.
    - It provides a simple interface for managing resources, including virtual
        machines, called "[droplets]".
- I created a [DigitalOcean API token] to allow the *Terraform* module to
    provision the necessary resources.
    - The API token must have the following permissions:
        - create, read, update, and delete droplets,
        - create, read, update, and delete reserved IPs,
        - read SSH keys,
        - create, read, delete tags.
- Since all digital ocean resources are configured individually, the automated
    provisioning will handle all the necessary configuration.

#### GitHub
- To use the all the *GitHub* required features, I needed two
    [*GitHub* API tokens].
    - The first [fine-grained token] is used to manage commit statuses and
        needed:
        - read repository contents,
        - read and write commit statuses.
    - Another [classic token] is necessary to publish the application container
        image.
    - Since the container image is not sensitive, I published all packages
        publicly to remove the need to authenticate the application server
        during installation.

#### Google Cloud Services
- I created a [*Google Cloud Services*] bucket to store any additional state
    files.
    - This will be necessary for coordinating deployment state across
        development environments.
    - *Terraform* will use this bucket to store deployment state.
    - This can be done using any compatible storage service that Terraform
        supports.
        - I used *Google Cloud Services* because I already had a Google
            account.
        - I would recommend using a more user-friendly service for a smaller
            project like this; [DigitalOcean Spaces] can be used.
- *Terraform* requires credentials to access the storage bucket.
    - I set up a [service account] and [principal] with the admin permissions
        for the target bucket.
    - A [JSON key] for accessing the bucket could then be generated for the
        service account.

### Terraform
- I configured a *Terraform* project to deploy the application to a production
     environment.
    - To host the application, I will require:
        - a virtual machine, running a web server & our application,
        - reserve a IP address to keep DNS records consistent between
            deployments,
        - update DNS records to point to the reserved IP address.
    - Terraform will automatically manage the infrastructure provisioning via
        Cloudflare and DigitalOcean, using their respective APIs.
- The Terraform module is simple.
    - The [main] module provisions the resources.
    - I defined the necessary and optional variables in the [variables] file.
- The [Terraform module] provisions the virtual machine with a customer
    cloud-init script.
    - The [cloud-init] script:
        - ensures the latest updates are installed,
        - installs the necessary dependencies:
            - `ufw` ,
            - `docker`.
        - configures a non-root, non-sudo user for running the application,
        - loads * configures a Caddy container to reverse proxy the
            application,
        - loads the application container image.
    - The cloud-init script is generated from a template to inject the
        necessary environment variables.

#### Applying Changes
- To deploy or re-deploy the application, a plan is created and applied.
    - The plan is created using the `terraform plan` command, to first review
        the necessary changes.
    - The plan is then executed using the `terraform apply` command.
    - I combined these two commands in the [`deploy` function] to automatically
        configure the terraform variables from the environment or command
        line arguments,
        - including the option to automatically approve the plan.
- The [`destroy` function] executes the same commands, except with the
    `-destroy` flag to remove the resources.
    - To avoid unexpected variable/state conflicts, all variables are again
        used to generate the plan, even if they are not used in the destroy
        resources.

#### Parallel Environments
- *Terraform* keeps track of the current state of the deployment.
    - The state is stored in a file, `terraform.tfstate`, by default in the
        current module directory.
    - To share the state between multiple environments, the state can be
        stored in a remote storage service.
    - I configured the *Terraform* module to use a remote
        [*Google Cloud Services* bucket] to store the state.
- When configuring terraform, the different state storage can be specified.
    - With the *Google Cloud Services* backend, a `PREFIX` configuration
        variable can be used to specify the state file to use.
    - By setting the `PREFIX` variable to the environment name, multiple
        environments can be deployed and tracked independently.
- Multiple environments are necessary for thorough testing.
    - I mainly use this to concurrently deploy a `staging` or `test` environment
        while the `production` environment is running.
    - Any number of environments can be deployed concurrently, so long as
        they do not share the same name.
    - To make sure there is no conflict, the [`deploy` function] automatically
        sets the host domain to be prefixed with the environment name, *e.g.*
        `staging.example.com`.

#### Testing Terraform
- Terraform comes with a built-in framework for [testing Terraform modules].
    - These tests consist of commands executions that can be asserted against.
- To test the *Terraform* module, I included unit and contract tests.
    - The simple [`main` module tests] evaluate the plan output without
        attempting to apply the changes.
    - In some cases, the plan cannot be evaluated until dependencies have been
        provisioned.
        - If so, the tests must apply the plan using a [mock provider].
    - To test [variable validation], invalid values can be refuted during the
        plan generation.

#### Testing Cloud Initialization
- I also managed to achieve test coverage of the cloud-init script.
    - Since the cloud-init script is a *BASH* script template, it can be
        tested like any other *BASH* script,
        - so long as there are no logical template expressions.
    - I used the *BATS* testing framework to test the cloud-init script,
        mocking all system dependencies.
    - This required some consideration to ensure the script could be tested
        without *Terraform* and all command calls could be effectively mocked.

### GitHub Actions
- Up to this point, the pipeline is completely functional and complete.
    - There's no functional requirement in this case to run the pipeline
        in a cloud environment.
    - The cloud environment will provide a persistent, consistent environment
        for executing pipeline processes.
    - A remote environment also makes it trivial to share the pipeline between
        multiple developers.
    - [*GitHub Actions*] can also automatically trigger pipeline processes, making
        sure integration and deployment is immediate and removing redundant
        manual tasks.
- I created a set [*GitHub Actions* workflows] to simply wrap the *BASH* command
    scripts.
    - Most of the workflows are simple, single-job workflows that run the
        entry-point scripts after setting up the environment.
    - The core workflows can be dispatched manually:
        - [`deploy.yaml`],
        - [`destroy.yaml`],
        - [`validate.yaml`],
        - [`test.yaml`].
    - Some workflows are triggered by events:
        - [`publish.yaml`] is triggered by a push to the `main` branch,
            deploying the `production` environment.
        - [`stage.yaml`] is triggered by a push to the `staging` branch,
            deploying the application to a `staging` environment.
        - [`test.yaml`] is triggered by a push to any branch that changes the
            CI/CD code, running all pipeline tests.

#### Testing GitHub Actions
- To completely automate testing of the CI/CD pipeline, the configuration of the
    *GitHub Actions* workflows must be tested.
    - There are no integrated testing tools or frameworks for testing *GitHub
        Actions* workflows.
    - Workflows can be run in an isolated environment using a *GitHub Actions*
        runner or using [`act`].
        - Neither provide much in terms of testing tools.
    - I attempted to instead use *GitHub Actions* itself to test the workflows.
- One additional workflow was needed to verify the *GitHub Actions* integration
    itself.
    - The [`verify.yaml`] workflow is also triggered by a push to any branch
        that changes the CI/CD code.
    - This workflow runs a basic integration test to check the required
        environments and each workflow.
- Separate from the integration test itself, each expected environment is
    verified.
    - The *GitHub Actions* environment variables and secrets are checked
        to ensure all necessary values are set and valid.
    - The workflows expect the environments:
        - `production`
        - `staging`
        - `test`
- A separate `verification` environment is used for the integration test.
    - This environment expects the authentication tokens to not be set.
    - Revoking the authentication tokens from this environment acts as a
        secondary safeguard against accidentally running process against real
        deployment environments.
- The integration test runs each workflow.
    - By making each workflow reusable, the integration test can trigger each
        job.
    - Injecting the `verification` environment and arguments to run the workflow
        commands with the `--dry-run` & `--verbose` flag simulates the command
        call without actually executing it.
    - The output of the command is then returned as a [job output] that can be
        later inspected.
    - Any necessary installation steps are also reflected in the job output.
- The integration test compares the output of each workflow to the expected
    output.
    - The expected dependency versions are asserted.
    - To check the environment variables are as expected, each command returns
        a hash of all relevant environment variables.
    - The hash of the workflow output is compared to a hash generated in the
        integration test itself, confirming the workflow was run in the expected
        environment.

## Retrospective
- While likely not finished, the deployment pipeline is functional and
    establishes a foundation for future development.
    - Some features may not be flexible enough for all situations.
        - Deployment requires the target application image to be public, which
            has implications for privacy and security.
    - Planned application features will require additional resources, such as:
        - a database,
        - a content delivery network (CDN).
- For now, this does everything I need to get started.
    - Deployment is reliable and consistent.
    - Application re-deployment requires little to no manual intervention.
    - The pipeline is portable and easy to run from an new machine or  in cloud.

### The Importance of Dev/Ops Platforms
- The most important lesson I learned from this process is the value of good
    dev/ops tools.
    - This project took significantly more time than I imagined.
    - The most difficult aspects were related to a lack of support in the tools
        I chose to use.
- By far, the most frustrating aspect of this project was the lack of testing
    tools for *GitHub Actions* workflows.
    - While devising a solution to test the workflows was an interesting
        exercise, I would not recommend it as a solution.
    - To get all of the integration tests working, I repeated over 400 workflow
        runs.
    - In the future, I intend to explore other options for cloud CI/CD
        environments.
        - Ideally, it would be possible to develop against a local environment
            in a [*KVM*] or *Docker* container, with sufficient testing support.
    - Otherwise, integration with *GitHub* was a good experience.
        - It was convenient to have the repository, package registry, and
            CI/CD status all in one place.
- I hope in the future to be able to generalize some features of this
    deployment pipeline.
    - This simple application delivery pipeline already has over 400 unit tests.
    - Now that I have more experience and a better understanding of the problem,
        hopefully I find additional tools to simplify similar solutions.
    - I would like to explore developing a more general-purpose, lightweight
        deployment pipeline solutions that can be easily adapted to different
        projects and use cases.

[preliminary planning]: /outlines/planning-a-personal-site.md#step-2-sveltekit
[*Svelte*]: https://svelte.dev/
[*SvelteKit*]: https://svelte.dev/docs/kit/introduction
[*Node.js*]: https://nodejs.org/en/
[download page]: https://nodejs.org/en/download
[*npm*]: https://www.npmjs.com/
[`folio-deploy` Activity Diagram]:
    https://www.figma.com/board/KlVlC2x59WcyfeWcGDwh5A/Portfolio-Plans?t=wbhkjfqd4uXKsdsu-6
[`folio-deploy` repository]: https://github.com/systemcarl/folio-deploy
[Dockerfile]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/Dockerfile
[`deploy` script]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/deploy
[`deploy` function]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/cli/deploy
[BATS]: https://bats-core.readthedocs.io/en/stable/
[BATS helper libraries]:
    https://bats-core.readthedocs.io/en/stable/writing-tests.html#libraries-and-add-ons
[`deploy` test file]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/cli/tests/deploy.bats
[`cli` test file]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/cli/tests/cli.bats
[*Cloudflare*]: https://www.cloudflare.com/
[*Cloudflare* API token]:
    https://developers.cloudflare.com/fundamentals/api/get-started/create-token/
[DNS SSL/TSL mode]:
    https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/
[redirect rules]:
    https://developers.cloudflare.com/rules/url-forwarding/single-redirects/create-dashboard/
[*DigitalOcean*]: https://www.digitalocean.com/
[droplets]: https://www.digitalocean.com/docs/droplets/
[DigitalOcean API token]:
    https://docs.digitalocean.com/reference/api/create-personal-access-token/
[*Google Cloud Services*]: https://cloud.google.com/
[service account]: https://cloud.google.com/iam/docs/service-account-overview
[principal]:
    https://cloud.google.com/iam/docs/service-account-overview#service-accounts-identities
[JSON key]:
    https://cloud.google.com/iam/docs/service-account-overview#credentials
[DigitalOcean Spaces]: https://www.digitalocean.com/products/spaces/
[*GitHub* API tokens]:
    https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens
[fine-grained token]:
    https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token
[classic token]:
    https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic
[Terraform module]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/infra/
[main]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/infra/main.tf
[variables]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/infra/variables.tf
[cloud-init]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/infra/cloud-init.tftpl
[`destroy` function]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/cli/destroy
[Terraform module tests]:
    https://developer.hashicorp.com/terraform/language/tests
[`main` module tests]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/infra/tests/main.tftest.hcl
[mock provider]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/infra/tests/mocks.tftest.hcl
[variable validation]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/infra/tests/variables.tftest.hcl
[*GitHub Actions*]: https://docs.github.com/en/actions
[*GitHub Actions* workflows]:
    https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions
[`deploy.yaml`]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/.github/workflows/deploy.yaml
[`destroy.yaml`]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/.github/workflows/destroy.yaml
[`validate.yaml`]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/.github/workflows/validate.yaml
[`test.yaml`]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/.github/workflows/test.yaml
[`publish.yaml`]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/.github/workflows/publish.yaml
[`stage.yaml`]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/.github/workflows/stage.yaml
[`verify.yaml`]:
    https://github.com/systemcarl/folio-deploy/blob/log/folio-framework/.github/workflows/verify.yaml
[`act`]: https://github.com/nektos/act
[job output]:
    https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows#using-outputs-from-a-reusable-workflow
[*KVM*]: https://linux-kvm.org/page/Main_Page
