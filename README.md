# Daniel's Linting Configuration Guide (for JavaScript)

## What does linting do? Why do you need it?

Linters reduce mistakes in code by enforcing good practices. They encourage consistency and readability.

> When someone asks for help with a project, the first thing I do is check for a working linter. 

## What are the common issues with linters?

> When I check for a working linter, 80% of the time, there is no linting configuration in the project, or it isn't being used properly.

### Bad project configuration

* Linters should be configured on a per-project basis. When people rely on a **global lint configuration** on their machine, they end up with different settings from other team members.
* You should be able to run the linter from the command line after all the project dependencies are installed. Sometimes configurations have **peer-dependencies** that aren't automatically added to your `package.json`; when this happens your linter will often stop working.

### Conflicts with other formatters and editors

* Code formatters and linters cover similar issues and can sometimes have **conflicting rules** if not configured properly. If your code editor runs a linter and then a formatter, some linter rules may be undone by the formatter.
* Sometimes **code editors aren't configured to use the right plugins**. For example, `eslint`, `prettier`, and `beautify` plugins for `VS Code` will often cause conflicts. Often folks have all three plugins installed simultaneously and not really know why they need any specific one.
* Often I'll find that linting is properly configured and running as a step in a CI pipeline, but folks are relying on the results of the pipeline to see errors instead of running linting locally or seeing errors in their code editor. Often this is due to misconfiguration in the code editor (above).

### Pipeline configurations

* Running a linter as a step in your CI pipeline is essential. But see point above, about people not using linting locally.
