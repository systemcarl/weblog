# Testing Github Workflows
> [!NOTE]
This article discusses the philosophy and general strategies for testing GitHub
    Actions workflows
to give appropriate context to the problem and the solution being proposed.
Testing [continuous integration and continuous deployment (CI/CD)] workflows
    can be complex,
and an effective solution often requires more than any one single approach.
>
> However, you can always skip ahead the tutorials on validating GitHub Actions
    workflows by [validating workflow output] or
    [by inspecting workflow-generated artifacts].
There is also an [example project] with self-testing GitHub Actions workflows
    that provides a working demonstration of the strategies discussed here.

For a while now, access to a CI/CD platform has been as easy as registering a
    [GitHub] account.
GitHub provides a free CI/CD automation platform, [GitHub Actions], for public
    repositories.
To automate any process involving your git repository (containing code or any
    other files),
simply [define a workflow file] in the `.github/workflows` folder of your
    repository
that delegates tasks to other pre-defined actions or custom shell commands.
Written in YAML,
these workflow files are easy to create and modify for anyone with basic
    information technology skills.

In my experience, however, a comprehensive CI/CD system is still seen as a
    luxury.
I think most developers still aspire to have any automation in place to remove
    some of the manual steps
— building, testing, and deploying software — from their daily routines.
It's also rare to consider testing the CI/CD system themselves.
Often, CI/CD testing is disregarded,
because CI/CD system requirements change rapidly to keep up with the software
    being deployed,
and there's little value beyond mitigating risk of catastrophic deployment
    failures.
Even when testing is considered, there is seldom any support for testing the
    automation directly,
further discouraging developers from actually testing their CI/CD solutions.

GitHub Actions is no exception.
Since workflow files are simply [YAML definitions] to configure the CI/CD jobs
    and their composing steps,
there is little functionality to be tested;
workflow files are just rules to guide the functionality of GitHub Actions.
However, the implementation being largely declarative
    (defining *what* to do, rather than *how* to do it)
does not preclude workflows from still needing testing.
These files still define the logic and sequence of operations to be performed
    (even if they don't themselves implement it),
    and ideally, the outcome should still be validated against the intent.
There are tools to run GitHub Actions workflows locally,
like [Act],
that can make debugging and troubleshooting workflows faster
— speeding up the most common "run it and see what happens" approach to CI/CD
    testing.
But these local tools do not provide any additional utilities or frameworks to
    make testing any simpler,
leaving developers to devise their own strategies for testing workflows.

## Deployment Testing
The most straightforward way to test [GitHub Actions] workflows
    (or any other CI/CD workflows) is to run them in a staging or test environment,
executing all the same steps as they would in production.
This is typically referred to as deployment testing,
    a very thorough form of [end-to-end] testing,
where the entire deployment process is validated from start to finish and
    the final state and functionality of a live system is verified.
Generally, this is the simplest and most reliable approach to testing GitHub
    Actions workflows.
However, there are some caveats to consider.
Primarily, there is often a cost (either monetary or time) to maintaining
    production-like environments for testing.
Sometimes, simplified environments can be used to reduce costs,
    but this may not reflect the actual deployment process accurately
— especially if performance or security is a concern.
This also assumes that any state or functionality being tested is observable
    from the outside,
which may not always be the case
— *especially if performance or security is a concern*.

Regardless of the project size or complexity,
deployment testing is always useful for validating larger infrastructure
    changes.
For example, if a new resource is being added to the system
    (*e.g.,* a server, database, or cache),
deployment testing can ensure that the new resource is properly configured
    and integrated with the existing system.
Testing completely in isolation (without actually attempting to provision the
    resources) may overlook critical nuances of the infrastructure deployment.
Many problems only manifest when networking, access controls, or other external
    dependencies are involved.
Therefore, testing is most effective when combining deployment tests with
    other isolated tests — like the later suggested here —
to balance reliability and cost.

## Validating Workflow Output
Since GitHub Actions workflows either directly or indirectly execute scripts
    and commands,
the resulting command line output can be used inspected and validate the
    behavior of the workflow.
This is especially useful for workflows that do not perform deployments
    directly,
but rather prepare environments, run tests, or generate reports.
If the workflow doesn't take any potentially destructive actions,
    the commands can be continually re-executed for testing purposes, without
    consequence.

However, even deployment workflows can be tested this way
    if the commands can be executed in a non-destructive manner.
A common pattern is to have a command line flags allow the command
    to describe the intended actions without actually performing them.
This is often referred to as a "dry-run", and may sometimes be combined with a
    "verbose" flag to output the necessary details to validate the workflow
    behavior.
With the right options, a deployment workflow can be effectively transformed
    into a non-destructive, reporting workflow,
making it safe to test repeatedly without side effects.
For and examples of self-reporting scripts, you can check out
    [the scripts used to build and deploy my personal website]
(the one you're likely reading this article on right now).

### Simplifying Workflows
No matter which way you look at it, testing GitHub Actions workflows directly
    is not usually easy and definitely not quick.
The difficulty only gets worse as workflows become more complex,
    with many steps and dependencies.
Therefore, it's almost always better to test the underlying logic outside of
    GitHub Actions workflows,
using a more suitable testing framework to verify the functionality or the
    scripts and commands being executed.
This not only makes testing easier and faster,
    but also encourages better separation of concerns and modularity in the
    workflow design.
When [I built the CI/CD solution for my personal website],
encapsulating all the deployment logic in [standalone scripts] made it simple to
    also run CI/CD tasks locally,
in-sync with the GitHub Actions workflows.

As a general rule, I suggest keeping workflows as simple as possible,
    delegating the actual work as much as possible to external scripts or
    commands.
Most of the time, the workflow should just be a thin wrapper around the actual
    logic.
The workflow's primary purpose is to set up the [environment]
(install dependencies, set environment variables, etc.),
and then execute the necessary command(s) or script(s)
— but ideally just one.
If more than one command or script is needed,
    consider combining them into a single script that can be executed in a
    single step.
Wrapping commands in a script can also allow dry-run or verbose modes to be
    implemented when the commands lack such functionality.
The most basic GitHub Actions workflow looks something like this:
> [!IMPORTANT]
On many runners (the virtual machines that execute the workflows),
scripts do not allow execution by default.
Often, a step to set the [executable permission] on the script file is needed
    before it can be run.
```yaml
name: Run Script

jobs:
  run-script:
    runs-on: ubuntu-latest
    steps:
      # make the source repository available
      - name: Checkout repository
        uses: actions/checkout@v4

      # allow script execution
      - name: Make script executable
        run: chmod +x ./script

      - name: Run script
        # execute the script
        run: ./script
        # define environment variables as needed for the script
        env:
          ENVIRONMENT: 'production'
```

### Re-Using Workflows
In the example above, the script output is output directly to the
    [workflow logs],
which can be inspected by the user via the GitHub web interface.
However, GitHub Actions does not provide ubiquitous access to the workflow logs
    during execution beyond the I/O streams of the hosting runner;
it is not easy to capture or extract workflow logs as variables or files during
    the workflow execution.
The output could be captured by the runner operating system,
but since each workflow job has its own isolated runner environment,
sharing outputs between jobs — and by extension, between workflows — requires
    some additional configuration.
One way to achieve this is by defining [job outputs],
which expose environment variables from a job to other jobs within the same
    workflow.
These job outputs can then also be exposed as [workflow outputs],
when configured as an output of a [re-usable workflow], which can be called as a
    job in another workflow.
> [!NOTE]
Alternately, artifacts can also be used to share data between jobs and
    workflows.
Artifacts are (generally) files that are uploaded to GitHub storage during
    workflow execution
and can be later downloaded by other jobs and workflows.
This is discussed later in the [Validating Generated Artifacts] section.
Aside from using artifacts instead of workflow outputs,
the proposed strategy for validating workflow via artifacts is otherwise the
    same
— and therefore, refers back to the rest of the [Validating Workflow Output]
    tutorial here for general guidance.

To make any workflow re-usable, add a `workflow_call` trigger to the workflow
    definition.
The `workflow_call` trigger provides an interface to define the inputs of the
    workflow,
which are expected to be provided by the calling workflow.
These inputs can be defined like any GitHub Actions workflow inputs,
with support for required and optional inputs, default values, and type
    annotations.
As any input, these values can be accessed as environment variables in the
    workflow steps,
using the `${{ inputs.<input_name> }}` to embed the input value into the
    environment variable definition (or command directly, if needed).
```yaml
name: Run Script

on:
  workflow_call:
    inputs:
      message:
        description: 'Message to print'
        required: true
        type: string
      environment:
        description: 'Environment name'
        required: false
        type: string
        default: 'development'

jobs:
  run-script:
    runs-on: ubuntu-latest
    steps:
      # previous steps ...

      - name: Run script
        run: ./script --message "$INPUT_MESSAGE"
        env:
          # pass inputs as environment variables to avoid embedding value in
          #    the run command
          INPUT_MESSAGE: ${{ inputs.message }}
          ENVIRONMENT: ${{ inputs.environment }}
```
Outputs can also be defined within the `workflow_call` trigger.
The outputs definition specifies the job outputs that should be exposed,
    respectively,
    as workflow outputs.
To reference a job output, the job must have an `id` defined.
The output value(s) is defined as variables,
written to the designated `$GITHUB_OUTPUT` file during the job execution.
The job output can then be referenced in the workflow output definition
    using the `${{ jobs.<job_id>.outputs.<output_variable> }}` syntax.
```yaml
name: Run Script

on:
  workflow_call:
    inputs:
      # input definitions ...
    outputs:
      # expose job output as workflow output
      script_output:
        description: 'Output from the script'
        value: ${{ jobs.run-script.outputs.script_output }}

jobs:
  run-script:
    runs-on: ubuntu-latest
    # define job outputs
    outputs:
      script_output: ${{ steps.run_script.outputs.script_output }}
    steps:
      # previous steps ...

      - name: Run script
        id: run_script
        # send script output to job output variable
        run: |
          OUTPUT=$(./script --message "$INPUT_MESSAGE")
          echo "$OUTPUT"
          echo "script_output<<EOF" >> $GITHUB_OUTPUT
          echo "$OUTPUT" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT
        env:
          INPUT_MESSAGE: ${{ inputs.message }}
          ENVIRONMENT: ${{ inputs.environment }}
```
> [!NOTE]
Many script outputs are more than one line.
To capture multi-line outputs and preserve formatting,
    the [here document] syntax can be used to mark the beginning and end of the
    respective output value.

> [!IMPORTANT]
It's also important to not expose sensitive information in workflow logs or
    as job/workflow outputs.
GitHub Actions [attempts to mask and warn about sensitive information] if
    detected,
but there is no guarantee that all sensitive information will be caught.

### Validating the Output
With the outputs of a re-usable workflow exposed,
a calling workflow can now execute the re-usable workflow and inspect the
    result.
If the workflow is executing a predictable command or script with a known
    output,
the output can be compared to the expected output to validate the workflow
    behavior.
If the output is too complex to predict or changes frequently,
a redundant execution of the command or script can be performed to generate the
    expected output for comparison
— assuming the command or script can be executed without detrimental side
    effects or expensive resource consumption.
```yaml
name: Test Run Script Workflow

on:
  push:
    branches:
      - main

jobs:
  run-workflow:
    uses: ./.github/workflows/script.yml
    with:
      message: 'Test Message'
      environment: 'test-env'

  verify-output:
    needs: run-workflow
    runs-on: ubuntu-latest
    steps:
      # repeat the prerequisite steps
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Make script executable
        run: chmod +x ./script

      - name: Compare Output
        # compare expected output to an explicit execution
        run: |
          EXPECTED="${{ needs.run-workflow.outputs.script_output }}"
          ACTUAL="$(./script --message "Test Message")"
          echo "$ACTUAL"
          echo ""
          if [ "$ACTUAL" == "$EXPECTED" ]; then
            echo "[x] Output matches"
          else
            echo "[ ] Output does not match"
            exit 1
          fi
        env:
          ENVIRONMENT: 'test-env'
```
In this example, a redundant execution of the `./script` command is performed
(along with all the prerequisite steps to set up the environment)
to generate the actual output for comparison.
Comparing the outputs ensures that the script is executed with the correct
    arguments and environment variables within the workflow under test.
Since the output is compared directly,
any discrepancy between the expected and actual output indicates a problem with
    the script invocation, not the script itself.
The script can be tested in it's own native environment separately,
    using a faster more focused testing framework.
The redundant setup and execution can be tedious to implement and maintain,
    but it is effective in preventing regressive workflow misconfigurations.
Keeping the command interface (or script) simple and stable can help reduce the
    burden of maintaining these tests.

### Overriding Behavior
Sometimes, the command or script being executed may need to behave differently
    when being tested.
Many CI/CD tasks are not free of side effects,
    they often involve deploying resources or making changes to existing
    systems,
which may not be safe or desirable during testing.
Fortunately, many commands and scripts provide options to modify their behavior,
    allowing them to be executed in a non-destructive manner.
A "dry-run" option is a common feature that simulates the actions of the command
    without actually performing them.
Additionally, a "verbose" option can provide more detailed output,
    which can be useful for validating the command behavior during testing.
Allowing the workflow under test to accept additional arguments to pass to the
    command or script
can allow a calling workflow control over invocation behavior during testing.
This can be achieved by adding an optional input to the re-usable workflow
    definition,
which is appended to the script arguments during execution.
A testing workflow can then provide the necessary "dry-run" or "verbose"
    arguments to ensure safe, verifiable execution during testing.
```yaml
# script.yaml
name: Run Script

on:
  workflow_call:
    inputs:
      # other inputs ...
      additional_args:
        description: 'Additional arguments for the script'
        required: false
        type: string
        default: ''
    outputs:
      # define outputs ...

jobs:
  run-script:
    runs-on: ubuntu-latest
    outputs:
      script_output: ${{ steps.run_script.outputs.script_output }}
    steps:
      # previous steps ...

      - name: Run script
        id: run_script
        run: |
          OUTPUT=$(./script --message "$INPUT_MESSAGE" $ADDITIONAL_ARGS)
          echo "$OUTPUT"
          echo "script_output<<EOF" >> $GITHUB_OUTPUT
          echo "$OUTPUT" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT
        env:
          INPUT_MESSAGE: ${{ inputs.message }}
          ADDITIONAL_ARGS: ${{ inputs.additional_args }}
          ENVIRONMENT: ${{ inputs.environment }}
```
```yaml
# test.yaml
name: Test Run Script Workflow

# trigger definitions ...

jobs:
  run-workflow:
    uses: ./.github/workflows/script.yml
    with:
      message: 'Test Message'
      # run the script in verbose mode for testing
      additional_args: '--verbose'
      environment: 'test-env'

  verify-output:
    needs: run-workflow
    runs-on: ubuntu-latest
    steps:
      # prerequisite steps ...

      - name: Compare Output
        # remember to compare with expected verbose output
        run: |
          EXPECTED="${{ needs.run-workflow.outputs.script_output }}"
          ACTUAL="$(./script --message "Test Message" --verbose)"
          # ...
        env:
          ENVIRONMENT: 'test-env'
```

## Validating Generated Artifacts
Instead of capturing command outputs as a workflow output variable,
the output can also be saved as an [artifact]
— a semi-permanent file stored by GitHub.
GitHub Actions artifacts are typically used to share files between jobs and
    workflows when direct inter-job communication is not practical.
It also provides a convenient way to persist files generated during workflow
    execution for later consumption or inspection.
If the CI/CD tasks generate files as part of their operation,
these files can be uploaded as artifacts during the workflow execution
and downloaded to another job or workflow for validation.

Aside from providing an alternative, indirect way to pass script outputs
    between workflows,
the testing strategy is the same as
    [the proposed strategy for validating workflow outputs].
The workflow under test instead uploads the generated output file as an
    artifact,
and the testing workflow can then download the artifact and read the contents
    for comparison against the expected output
(or [a redundant execution of the command or script]).
```yaml
# script.yaml
name: Run Script

# trigger definitions ...

jobs:
  run-script:
    runs-on: ubuntu-latest
    outputs:
      # define outputs ...
    steps:
      # previous steps ...

      - name: Run script
        id: run_script
        # Save output to a file for artifact upload
        run: |
          OUTPUT=$(./script --message "$INPUT_MESSAGE" $ADDITIONAL_ARGS)
          echo "$OUTPUT"
          echo "$OUTPUT" > output.txt
          # ...
        env:
          INPUT_MESSAGE: ${{ inputs.message }}
          ADDITIONAL_ARGS: ${{ inputs.additional_args }}
          ENVIRONMENT: ${{ inputs.environment }}

      # Upload the output file using the upload-artifact action
      - name: Upload script artifact
        uses: actions/upload-artifact@v4
        with:
          name: output
          path: output.txt
```
```yaml
# test.yaml
name: Test Run Script Workflow

# trigger definitions ...

jobs:
  run-workflow:
    uses: ./.github/workflows/script.yml
    with:
      message: 'Test Message'
      additional_args: '--verbose'
      environment: 'test-env'

  verify-artifact:
    needs: run-workflow
    runs-on: ubuntu-latest
    steps:
      # prerequisite steps ...

      - name: Download script artifact
        uses: actions/download-artifact@v4
        with:
          name: output

      - name: Compare Artifact Output
        run: |
          EXPECTED="$(cat output.txt)"
          ACTUAL="$(./script --message "Test Message" --verbose)"
          echo "$ACTUAL"
          echo ""
          if [ "$ACTUAL" == "$EXPECTED" ]; then
            echo "[x] Artifact output matches"
          else
            echo "[ ] Artifact output does not match"
            exit 1
          fi
        env:
          ENVIRONMENT: 'test-env'
```

## More Examples
For simplicity, the above examples test a trivial script execution.
Since any number commands or scripts can be wrapped into a single command
    line execution,
any complex CI/CD task can be tested using the same strategies with minimal
    refactoring.
In addition to the example project for this tutorial,
[the CI/CD workflows for my personal website] also use these strategies
    extensively to thoroughly [test all of the testing and deployment workflows]
— including [the workflow responsible for automatically testing] all
    [the underlying shell scripts] used by the deployment workflows.

The self-testing workflows in my personal website CI/CD solution also include
    some [additional testing to verify the GitHub Actions environments]
    themselves.
The workflows require several environment variables and secret credentials to
    be set up correctly in order to function properly.
Dedicated jobs are invoked to validate these variables have been set in the
    GitHub Actions environment
and provides a detailed report of any environment misconfiguration.
This helps catch configuration issues early,
and provides the feedback necessary to troubleshoot problems quickly.
The environment validation jobs also checks that the environment used for test
    execution does not have the credentials necessary to make changes to
    live services,
further protecting against detrimental workflow executions.

[continuous integration and continuous deployment (CI/CD)]:
    https://en.wikipedia.org/wiki/CI/CD
[validating workflow output]: #validating-workflow-output
[by inspecting workflow-generated artifacts]: #validating-generated-artifacts
[example project]: https://github.com/systemcarl/github-workflow-test
[GitHub]: https://github.com
[GitHub Actions]: https://docs.github.com/en/actions
[define a workflow file]:
    https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax
[YAML definitions]:
    https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#about-yaml-syntax-for-workflows
[Act]: https://github.com/nektos/act
[end-to-end]: https://en.wikipedia.org/wiki/System_testing
[the scripts used to build and deploy my personal website]:
    https://github.com/systemcarl/folio
[I built the CI/CD solution for my personal website]:
    ../../devlogs/blank/ci-cd.md
[standalone scripts]: ../../devlogs/blank/ci-cd.md#following-the-script
[environment]:
    https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-variables#defining-environment-variables-for-a-single-workflow
[executable permission]: https://en.wikipedia.org/wiki/File-system_permissions
[workflow logs]:
    https://docs.github.com/en/actions/how-tos/monitor-workflows/use-workflow-run-logs?versionId=free-pro-team%40latest&productId=actions&restPage=how-tos%2Cwrite-workflows%2Cchoose-what-workflows-do%2Cuse-variables
[job outputs]:
    https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/pass-job-outputs?versionId=free-pro-team%40latest&productId=actions&restPage=how-tos%2Cmonitor-workflows%2Cuse-workflow-run-logs#defining-and-using-job-outputs
[workflow outputs]:
    https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows#using-outputs-from-a-reusable-workflow
[re-usable workflow]:
    https://docs.github.com/en/actions/using-workflows/reusing-workflows
[Validating Generated Artifacts]: #validating-generated-artifacts
[Validating Workflow Output]: #validating-workflow-output
[here document]: https://en.wikipedia.org/wiki/Here_document
[attempts to mask and warn about sensitive information]:
    https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets#using-secrets-in-a-workflow
[artifact]: https://docs.github.com/en/actions/tutorials/store-and-share-data
[the proposed strategy for validating workflow outputs]:
    #validating-workflow-output
[a redundant execution of the command or script]: #overriding-behavior
[the CI/CD workflows for my personal website]:
    https://github.com/systemcarl/folio/tree/v0.0.5/.github/workflows
[test all of the testing and deployment workflows]:
    https://github.com/systemcarl/folio/blob/v0.0.5/.github/workflows/verify.yaml
[the workflow responsible for automatically testing]:
    https://github.com/systemcarl/folio/blob/v0.0.5/.github/workflows/test.yaml
[the underlying shell scripts]:
    https://github.com/systemcarl/folio/tree/v0.0.5/cli
[additional testing to verify the GitHub Actions environments]:
    https://github.com/systemcarl/folio/blob/v0.0.5/.github/workflows/verify.yaml#L20-L158
