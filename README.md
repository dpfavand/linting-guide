# Daniel's Linting Configuration Guide (for JavaScript)

## What does linting do? Why do you need it?

Linters reduce mistakes in code by enforcing good practices. They encourage consistency and readability.

> When someone asks for help with a project, the first thing I do is check for a working linter.

## What are the common issues with linters?

aka why did Daniel send me here?

> When I check for a working linter, 80% of the time, there is no linting configuration in the project, or it isn't being used properly. And you can tell in the quality of the codebase and the types of issues that folks find themselves tyring to untangle.

### Bad project configuration

- Linters should be configured on a per-project basis. When people rely on a **global lint configuration** on their machine, they end up with different settings from other team members.
- You should be able to run the linter from the command line after all the project dependencies are installed. Sometimes configurations have **peer-dependencies** that aren't automatically added to your `package.json`; when this happens your linter will often stop working.

### Conflicts with other formatters and editors

- Code formatters and linters cover similar issues and can sometimes have **conflicting rules** if not configured properly. If your code editor runs a linter and then a formatter, some linter rules may be undone by the formatter.
- Sometimes **code editors aren't configured to use the right plugins**. For example, `eslint`, `prettier`, and `beautify` plugins for `VS Code` will often cause conflicts. Often folks have all three plugins installed simultaneously and not really know why they need any specific one.
- Often I'll find that linting is properly configured and running as a step in a CI pipeline, but folks are relying on the results of the pipeline to see errors instead of running linting locally or seeing errors in their code editor. Often this is due to misconfiguration in the code editor (above).

### Pipeline configurations

- Formatting tool - often also a CLI or API

* Running a linter as a step in your CI pipeline is essential. But see point above, about people not using linting locally.

## Conceptual configuration guide

There are several parts to a working lint configuration, generally regardless of language, which all need to work together:

- Linting tool - generally a CLI or API that can read code files (and write fixes) and identify issues based on...
- Linting rules - collections of rules that can be used to identify problematic code
  - Linting configurations - linting rules are often bundled together to provide a common, consistent style guide
- Formatting tool - often also a CLI or API, similar to a linting tool but focused only on formatting
- Git repository - your project should exist in a repository, and the linter configurations should be stored as part of that repository to make sure everyone is working from the same rules
- CI pipeline - often attached to the Git repository, often runs the linter from the CLI before/alongside any other automated tests
- Git hooks - hooks are commands that can run on specific events, such as creating a commit
- Code editor - a tool that may automatically run the linter and formatter on save or during editing to provide feedback and auto-fixes. Must be careful it doesn't apply linting or formatting that is not defined in the repository settings

## JavaScript configuration guide

If you're setting up a new project or adding linting to an existing one:

### 1. Linting Tool: Eslint

At the highest level in your project, often at the root, add `eslint` to your `devDependencies`:

```shell
npm i -D eslint
```

### 2. Linting Rules: AirBnB

I recommend using the AirBnB eslint config unless you have a good reason not to. It provides a very common and comprehensive set of standards for JavaScript and React. Some frameworks may provide their own configurations, but usually I prefer to just go with AirBnB style.

This will install the eslint configurations, including all of the required peer dependencies:

```shell
npx install-peerdeps --dev eslint-config-airbnb
```

### 3. Linting Configuration

To tell `eslint` what configuration to use, create a `.eslintrc` file in the root of your project (at the same level as the `package.json`), with the following JSON:

```json
{
  "extends": ["airbnb", "airbnb/hooks"]
}
```

This sets up your project configuration to use the `airbnb` and `airbnb/hooks` rules as baselines. Any changes you make in `.eslintrc` will be applied on top of these configurations. Make sure this file is commited to the repository.

### 4. Formatter: Prettier

Prettier can be run on it's own, and it often is, especially when used as an automatic formatter within a code editor. However, some of its rules will conflict with the AirBnB eslint rules. What we will do instead is extend our configuration to disable the conflicting rules, and run `prettier` as part of the `eslint` process. So in the end, you will only run `eslint`, and then `eslint` will run `prettier` for you.

Run these commands to install all the required modules:

```shell
npm i -D eslint-plugin-prettier eslint-config-prettier
npm i -D --save-exact prettier
```

And update your `.eslintrc` file so it looks like this:

```json
{
  "extends": [
    "airbnb",
    "airbnb/hooks",
    "plugin:prettier/recommended",
    "prettier/react"
  ]
}
```

### 5. NPM Scripts

Now, we'll create a couple NPM scripts that make it easy to run the linter. Update your `scripts` object in `package.json` to add these:

```json
{
  "scripts" {
    (other scripts),
    "lint": "eslint . --ext .jsx,.js",
    "lint:fix": "eslint . --ext .jsx,.js --fix",
    "lint:githook": "eslint --ext .jsx,.js --fix --ignore-path .gitignore"
  }
}
```

Now from the command line you can run the following to automatically fix some errors, and see reports about the others:

```shell
npm run lint:fix
```

### 6. Git Hooks

Because different people's code editors may not apply the same rules if they aren't configured properly, we configure `eslint` to run when we do a `git commit`. That way it is difficult to commit unlinted. This is important even if you do configure your code editor to auto fix on save: you won't always use the code editor and you can't guarantee that everyone else will have the right configurations.

Install Husky:

```shell
npm i -D husky lint-staged
```

Add `husky` and `lint-staged` configurations to your `package.json`:

```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.{js,jsx}": ["npm run lint:githook"]
  }
}
```

Now, when you commit, the linter will run. If you do need to escape this, you can add `--no-verify` to the `git commit` command:

```shell
git commit --no-verify -m "Commit Message"
```

### 7. CI Pipeline

Using whatever CI pipeline you have available, configure a step like the following:

```shell
npm i
npm run lint
```

### 8. Code Editor Configuration

Code editor configuration can be a matter of preference. For example,

- Some people dont' want squiggly lines in their editor (too distracting)
- Some turn on auto save (which, if you have auto-fix on, can easily result in garbled code)

If you've configured git hooks and the CI pipeline above, you should be set to enforce the code rules regardless of the editor preferences. But I do recommend the following for `VS Code`:

- Remove the Prettier and Beautify plugins if you have them(two common JS linters), or at least disable them in your project's workspace.
- Install the `Eslint` plugin
- Add the following to your `.vscode/settings.json` file (or create if not present):

```json
{
  "editor.codeActionsOnSsave": {
    "source.fixAll.eslint": true
  }
}
```

You can also set `"source.fixAll": true` if you want to turn on autofix for other formatters (particularly useful if you're using a CSS/SCSS formatter).

These settings will apply to all VS Code users who open this repository, so this is a great way to share configuration. It isn't foolproof, though - if you have different plugins installed you can still end up with different autofix results. That's why having commit hooks enabled above is so important.

### 9. Follow the rules!

Now that you have eslint working, make sure to follow the rules. Sometimes rules can be overridden, but you should only do so when you understand why.

## Finally, all the configurations in one step:

Run these:

```shell
npm i -D eslint husky eslint-plugin-prettier eslint-config-prettier
npx install-peerdeps --dev eslint-config-airbnb
npm i -D --save-exact prettier
```

Create/update `.eslintrc`:

```json
{
  "extends": [
    "airbnb",
    "airbnb/hooks",
    "plugin:prettier/recommended",
    "prettier/react"
  ]
}
```

Update `package.json`:

```json
{
  "scripts" {
    (other scripts),
    "lint": "eslint . --ext .jsx,.js",
    "lint:fix": "eslint . --ext .jsx,.js --fix",
    "lint:githook": "eslint --ext .jsx,.js --fix --ignore-path .gitignore"
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.{js,jsx}": ["npm run lint:githook"]
  }
}
```

Create/update `.vscode/settings.json`:

```json
{
  "editor.codeActionsOnSsave": {
    "source.fixAll.eslint": true
  }
}
```
