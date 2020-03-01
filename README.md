# monorepo-lerna-yarn-workspaces

<p align="center">
  <a href="https://lerna.js.org/">
    <img  src="https://miro.medium.com/max/800/1*lgWw_inXlHN4dSgn4hxJSg.jpeg">
  </a>
</p>

## Introduction

This is a Proof of concepts of a monorepo using lerna and YARN workspaces.

> Lerna makes versioning and publishing packages easier by providing utility commands for handling the execution of tasks across multiple packages 
https://lerna.js.org/

> Yarn Workspaces manages our dependencies and optimizes the installing of dependencies, rather than have multiple node_modules. 
https://classic.yarnpkg.com/en/docs/workspaces/



## Quick Start


### Initial project config

> Init and set up the project with Lerna
```
$ npx lerna init
```

> Update the just created *"lerna.json"* to use Yarn Workspaces
```
{
    "packages": ["packages/*"],
    "npmClient": "yarn",
    "useWorkspaces": true,
    "version": "independent"
}
```

> Also modify *package.json* to define where the Yarn workspaces are located and set repository as private
```
{
    "name": "whatever",
    "private": true,
    "workspaces": ["packages/*"],
    "devDependencies": {
        "lerna": "^3.20.2"
    }
}
```
Setting private to true will prevent the root project from being published to NPM.

> Add node_modules directory to .gitignore


### Creating a new package

> Create a new directory inside *packages* directory.

```
$ cd packages
$ mkdir my-poc-core
$ cd my-poc-core

```

> Create a new package.json by running yarn init or npm init
```
$ yarn init
```
Is important to note that:
* Name the package following our NPM Org scope (example @my-scope-name)
* Start at version 0.0.0. Once we publish it using Lerna, it will be published at 0.1.0 or 1.0.0

If we have an NPM Org account that supports private packages, we can add restricted access:
```
"publishConfig": {
    "access": "restricted"
}
```

### Adding a local sibling dependency to a Specific Package

To add an existing package as dependency to another package and have Lerna symlink them:
```
$ cd my-poc-form
$ lerna add @my-poc-scope/my-poc-button --scope=@my-poc-scope/my-poc-form
```

This will update package.json of @my-poc-scope/my-poc-form:
```
// package.json
{
  "name": "@my-poc-scope/my-poc-form",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "@my-poc-scope/my-poc-button": "^0.0.0"
  }
}
```

Now we can reference the local dependency in index.js:
```
import Button from '@my-poc-scope/my-poc-button';
```

### Adding a common dependency to all packages
Doing this is similar to the previous command. This would be for /packages/*. 
It doesn’t matter if they’re local sibling dependencies or from NPM.

```
$ lerna add my-poc-validations
```

If we have common dev dependencies, is better to specify them in the worksapce root package.json.
Example wih Husky:
```
$ yarn add husky --dev -W
```
Adding -W flag instructs Yarn to install the given dependencies for the *entire workspace*

## Lerna features

### Removing dependencies
Lerna *exec* command runs the given command in each package.
```
$ lerna exec -- yarn remove dependency-name
```

### Running tests

Lerna *run* command run an npm script in each package that contains that script

```
$ lerna run test --stream
```
The stream flag just provides output from the child processes

### Publishing to NPM

We have to verify we are logged in npm:
```
$ npm whoami // myusername
```

Then we can publish with lerna:
```
$ lerna publish
```

### Automatic Publish with Conventional Commits

Lerna supports the use of the Conventional Commits Standard to automate Semantic Versioning in a CI environment.

We can do it by running the command:
```
$ lerna publish --conventional-commits --yes
```

Or modifying our lerna.json:
```
"command": {
    "publish": {
       "conventionalCommits": true, 
       "yes": true
    }
}
``` 

## Cross Project Local Development


### Symlink a local dependency
In order to create new components and test it before publishing, we can use yarn link command

```
$ cd ~/path/to/my-new-component
$ yarn link
```

Now our package is symlinked, we can go to the other package to use:
```
$ cd ~/path/to/my-other-component
$ yarn link  @my-scope-name/my-new-component
```

Any changes in packages/my-new-component will be reflected in my-other-component.

### Unlink a local dependency
```
$ cd ~/path/to/my-unlinked-component
$ yarn unlink
```

Now we can go to the other package to use:
```
$ cd ~/path/to/my-other-component
$ yarn link  @my-scope-name/my-unlinked-component
```