# Abstract
- Monorepos easy to manage multiple software projects within a single
    repository.
- *npm* workspaces make it easy to manage multiple Node.js projects within a
    unified environment.

# Setting Up a Monorepo with *npm* Workspaces
- Projects often grow to include multiple related packages.
    - Combining these related packages into a single version-controlled
        repository is called a [monorepo].
    - Monorepos simplify:
        - dependency management,
        - code sharing,
        - package versioning.
- [*npm*] installs the JavaScript packages for [Node.js] projects.
    - Each project typically has its own `node_modules` directory.
        - This folder contains all the external dependencies required for that
            specific project.
    - Duplication and inconsistencies between projects can cause unexpected
        behavior.
- [*npm* workspaces] consolidate environments for multiple Node.js projects into
    a single parent project.
    - A single `node_modules` directory is shared across all workspace projects.
    - *npm* workspaces also:
        - [links local sub-project references],
        - [run scripts across multiple project contexts].

## Using Up NPM Workspaces
- *npm* workspaces just requires a root `package.json` file to define the
    workspaces.
    - *npm* workspaces are supported in *npm* version 7 and later.
    - `npm install` may need to be run from the root directory to reinstall
        dependencies if migrating an existing project to use workspaces.

### Defining Workspaces
- *npm* workspaces are defined in a root `package.json` file.
    - Each sub-project within the workspace has its own `package.json` file.
    - The sub-projects directory must be added to the `workspaces` field in the
        root `package.json`.
        - The workspace can be a list of directories or glob patterns.
    ```json
    // `package.json`
    {
      // ...
      "workspaces": [
        "projects/*", // includes all projects in the `./projects` folder
        "shared" // includes the `./sharded` folder
      ]
    }
    ```

### Cross-Referencing Projects
- Projects within the workspace can reference each other by their package names.
    - *npm* automatically links the workspace projects.
    - The referencing project only needs to specify the package name and version
        in its `package.json` dependencies.
    - The name must match the `name` and `version` fields in the referenced
        project's `package.json`.
    ```json
    // `projects/app/package.json`
    {
      "name": "app",
      "version": "1.0.0",
      "dependencies": {
        "shared": "^1.0.0" // references the `shared` project
      }
    }
    ```
    ```json
    // `shared/package.json`
    {
      "name": "shared",
      "version": "1.0.1",
    }
    ```
    ```javascript
    // `projects/app/index.js`
    import Shared from 'shared'; // imports from the `shared` project
    ```

### Running Workspace Commands
- *npm* scripts can be run across one or more workspace projects.
    - The [`--workspaces` flag] runs the command in all workspace projects.
    ```bash
    npm run start --workspaces
    ```
    - The [`--workspace=<name>` flag] runs the command in the specific
        workspace(s), identified by their workspace name(s).
    ```bash
    npm run start --workspace=app --workspace=server
    ```
    - The workspace scripts are run synchronously.
        - Starting multiple long-running processes may require using separate
            terminal sessions or the [`concurrently` package].

## A React + NestJS Example
- The [example React + NestJS monorepo template] demonstrates using *npm*
    workspaces to manage a full-stack application.
    - The template includes:
        - A React front-end project,
        - A NestJS back-end project,
        - A shared utilities project.
    - The shared project contains code used by both the front-end and back-end
        projects.
    - Base TypeScript configurations are shared across all projects.
- *npm* workspaces does not ensure compatibility between different projects.
    - Projects may use different module systems
        - (e.g., [ESM] vs. [CommonJS]).
    - The shared typescript must be compiled to be compatible with NestJS.
        - The React + Vite front-end can use ESM modules directly.
    ```json
    // `shared/package.json`
    {
      "name": "@react-nest/shared",
      // ...
      "module": "src/index.js", // ESM output for React
      "main": "dist/index.js", // CommonJS output for NestJS
      "types": "dist/index.d.ts",
      "scripts": {
        "build": "tsc -b",
        "watch": "tsc -b -w"
      }
    }
    - The development servers are managed via the root `package.json` scripts.
    {
      "name": "react-nest",
      // ...
      "workspaces": [
        "backend",
        "frontend",
        "shared"
      ],
      "scripts": {
        "dev:back": "npm run start:dev --workspace=backend",
        "dev:front": "npm run dev --workspace=frontend",
        "dev:shared": "npm run watch --workspace=shared",
        "build": "npm run build --workspace=shared --workspace=backend --workspace=frontend",
        "start:back": "npm run start:prod --workspace=backend",
        "start:front": "npm run preview --workspace=frontend"
      },
      // ...
    }
    ```

## Considerations
- *npm* workspaces provides basic monorepo functionality.
    - Using a monorepo provides more flexibility than a monolithic codebase.
    - There are many more advanced monorepo management tools.
        - The tools range in scope and complexity.
- The [example monorepo] works well for a simple full-stack application.
    - The example solution may not scale well.
        - The package versions must be manually updated across all projects.
        - Running scripts is a sequential process that may not be efficient
            or reliable enough for larger projects.

[monorepo]: https://en.wikipedia.org/wiki/Monorepo
[*npm*]: https://www.npmjs.com/
[Node.js]: https://nodejs.org/
[*npm* workspaces]: https://docs.npmjs.com/cli/v8/using-npm/workspaces
[links local sub-project references]:
    https://docs.npmjs.com/cli/v8/using-npm/workspaces#using-workspaces
[run scripts across multiple project contexts]:
    https://docs.npmjs.com/cli/v8/using-npm/workspaces#running-commands-in-the-context-of-workspaces
[`--workspaces` flag]:
    https://docs.npmjs.com/cli/v8/using-npm/workspaces#running-commands-in-the-context-of-workspaces
[`--workspace=<name>` flag]:
    https://docs.npmjs.com/cli/v8/using-npm/workspaces#running-commands-in-the-context-of-workspaces
[`concurrently` package]: https://www.npmjs.com/package/concurrently
[example React + NestJS monorepo template]:
    https://github.com/systemcarl/react-nest
[ESM]: https://nodejs.org/api/esm.html#introduction
[CommonJS]: https://nodejs.org/api/modules.html#modules-commonjs-modules
[example monorepo]: https://github.com/systemcarl/react-nest
