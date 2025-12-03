# Abstract
- Testing CI/CD automation ensures reliable software delivery.
- Github Actions does not provide tools for testing workflows.
- Output validation and dry-run techniques can be used to test workflows.

# Testing Github Workflows
- Workflows define GitHub Actions tasks for CI/CD automation.
    - They are defined in YAML files stored in the `.github/workflows` folder.
- Unfortunately, there are no options to test workflows directly in GitHub.
    - Act is a popular tool for running GitHub Actions locally.
        - Act doesn't have any tools for testing either.

## Validating Workflow Output
- One way to test workflows is to validate their outputs.
    - Comparing an expected output to actual workflow results can validate the
        workflow.
    - Using "dry-run" and "verbose" options allow testing intent without
        executing CI/CD tasks.
    - There are examples in my personal website CI/CD pipeline.

### Simplifying Workflows
- Testing workflows is not quick or easy.
    - Complex workflows with many steps are tedious to test.
    - It is easier to test logic outside of GitHub Actions.
- Ideally, workflows should only define the minimum setup required to run tasks.
    - The workflow should:
        - Set up the environment.
        - Execute a single script or command.
```yaml
name: Run Script

jobs:
  run-script:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Make script executable
        run: chmod +x ./script

      - name: Run bash script
        run: ./script
          ENVIRONMENT: 'production'
```

### Re-Using Workflows
- Re-usable workflows allow workflows to be called from other workflows.
    - This allows a workflow to be tested using another workflow.
- Defining an `workflow_call` trigger allows a workflow to be called from
    another workflow.
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

      - name: Run bash script
        run: ./script --message "$INPUT_MESSAGE"
        env:
          # pass inputs as environment variables to avoid embedding value in
          #    the run command
          INPUT_MESSAGE: ${{ inputs.message }}
          ENVIRONMENT: ${{ inputs.environment }}
```
- Data can be passed back to the calling workflow via the workflow outputs.
    - The script output can be recorded to a workflow output variable.
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

      - name: Run bash script
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
- Remember not to expose sensitive data in workflow outputs.
    - GitHub Actions will attempt to mask secrets.

### Validating the Output
- A calling workflow can compare the output of a re-usable workflow to the
    expected output.
    - A redundant execution can be used if the command or script can be run
        without side effects.
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
        # compare expected output to an actual execution
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
- The test workflow will fail if the outputs do not match.
    - The test provides redundancy and prevents regression.

### Overriding Behavior
- Allowing additional arguments to be passed to the script can all for more
    flexible testing.
    - Passing test dry-run or verbose flags can allow testing without side
        effects.
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

      - name: Run bash script
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
- Artifacts can also be used to validate workflow outputs instead of direct
    output comparison.
    - Requires persisting artifacts between jobs and workflows.
    - Artifacts can be downloaded and inspected after workflow execution.
    - Ideal if the CI/CD tasks generate files as part of their operation.
- The strategy is similar to output validation
    - Artifacts are stored and retrieved instead of returning output variables.
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

      - name: Run bash script
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
- More examples of testing GitHub workflows can be found in my personal
    website repository.
- I also use a testing workflow to validate the GitHub Actions environment
    variables and runner configuration.
    - This helps troubleshoot configuration issues when setting up new
        environments.
    - Verifying the "verification" environments ensures test workflow runs do
        not have access to production credentials.
