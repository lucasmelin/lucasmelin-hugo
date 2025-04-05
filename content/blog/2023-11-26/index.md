---
date: "2023-11-26"
title: "Automatically detecting and resolving deprecations using Semgrep"
tags: ["semgrep", "go", "bash"]
summary: "Managing deprecations oftentimes requires a lot of manual work and communication. But what if we could automate this process instead?"
---

Working on internal tools for developers, it's common to deprecate unused features or outdated workflows.
Communicating these deprecations and ensuring that all teams are successfully migrated off of the deprecated functionality can be a challenge.

Normally, you might add a warning to your tooling that is displayed whenever the deprecated feature is used, but that requires someone to:

1. **Notice** the deprecation message.
2. **Understand** the change that needs to be made.
3. Have the **time** and **motivation** to make the required change across one or more files.

And these 3 steps are required across _all_ affected repositories.

You could also take the approach of creating the pull requests yourself,
but that might involve manually searching and making changes across many unfamiliar repositories, which can make it easy to miss something.

An alternative is to automate this process, using a tool like [Semgrep](https://semgrep.dev/).

# Using Semgrep

Semgrep allows you to write patterns and rules to enforce specific practices in your codebase.
It's oftentimes marketed as a security tool, since it can be used to detect vulnerable coding practices, but it's also a great choice for our specific use case.

Let's take a look at a concrete example.
The code for these next steps [is on GitHub](https://github.com/lucasmelin/semgrep-deprecation-demo), so you can follow along using the [Semgrep CLI](https://github.com/semgrep/semgrep?tab=readme-ov-file#option-2-getting-started-from-the-cli) on your local machine, or in the [Semgrep playground](https://semgrep.dev/playground/new) if you prefer not to install anything.

Imagine you have an internal tool called `code-buddy`, and you're deprecating the `--please` flag from the `run <name>` command.
You know that references to this tool are found in several different types of places - in READMEs, in GitHub Actions workflows, in bash scripts, and even in some Go code.
Rather than take the old-school manual search-and-replace approach, let's automate this whole process.

We'll start by creating a rule to detect the deprecated flag in bash scripts.
Create a `rule.yaml` file and add the following content:

```yaml
rules:
  - id: bash-deprecated-please-flag
    languages:
      - bash
    message: >
      code-buddy[🤖]: The --please flag is deprecated.
    severity: WARNING
```

This creates a new Semgrep rule that will run on all `bash` scripts, and will print out our custom warning message.
All that's missing is a `pattern` to detect usage of our deprecated flag.

Looking at [one of the bash scripts](https://github.com/lucasmelin/semgrep-deprecation-demo/blob/main/run-app.sh) that uses our `code-buddy` script, we see the following line:

```bash
code-buddy run mycoolapp --please
```

So we want our pattern to match `code-buddy run`, plus some string, followed by our `--please` flag.

After adding the pattern, our rules file should look something like this:

```yaml {hl_lines=[7]}
rules:
  - id: bash-deprecated-please-flag
    languages:
      - bash
    message: >
      code-buddy[🤖]: The --please flag is deprecated.
    pattern: code-buddy run $X --please
    severity: WARNING
```

We can then test out our new rule using the Semgrep CLI by running `semgrep scan --config rule.yaml`:

```sh
$ semgrep scan --config rule.yaml

┌────────────────┐
│ 1 Code Finding │
└────────────────┘

    run-app.sh
       bash-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

            3┆ code-buddy run mycoolapp --please

Ran 1 rule on 1 file: 1 finding.
```

With that, we've just written our first successful Semgrep rule which can detect our deprecated `--please` flag in bash scripts!

But we don't have to stop at _detecting_ the issue, we can also use Semgrep to _fix_ this issue as well.

## Autofix

Now that we can find uses of the `--please` flag, we can use Semgrep to also remove the deprecated flag.
To do this, we need to add a `fix` to our existing rule:

```yaml {hl_lines=["8"]}
rules:
  - id: bash-deprecated-please-flag
    languages:
      - bash
    message: >
      code-buddy[🤖]: The --please flag is deprecated.
    pattern: code-buddy run $X --please
    fix: code-buddy run $X
    severity: WARNING
```

In this case, the change we're appling is just dropping the `--please` flag from the end of the command, and keeping the rest.
We can run Semgrep with the `--autofix` flag to automatically apply the fix, or we can also include the `--dry-run` flag to simply preview the proposed changes.

```sh
$ semgrep scan --config rule.yaml --autofix --dryrun

┌────────────────┐
│ 1 Code Finding │
└────────────────┘

    run-app.sh
       bash-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ code-buddy run mycoolapp
            3┆ code-buddy run mycoolapp

Ran 1 rule on 1 file: 1 finding.
```

Now our rule can automatically fix this issue in any bash scripts it encounters.

Let's move on to detecting the `--please` flag in our documentation.

## Extracting bash from Markdown

[In our README](https://github.com/lucasmelin/semgrep-deprecation-demo/blob/main/README.md), there's an example in a bash code block showing how to call the `code-buddy` CLI from the command-line.
In order to detect usage of the `--please` flag in a Markdown document, we could either create a rule using the `generic` language type, or we could tell Semgrep how to identify the bash script in the code block and reuse our existing rule.

Let's try out option two.

We'll start by creating a new Semgrep rule, but this time, we'll set the mode to `extract`. Since Semgrep doesn't currently support Markdown, we'll set the language to `generic`.

```yaml
rules:
  - id: extract-markdown-code-block
    mode: extract
    languages:
      - generic
```

Next, we need to tell Semgrep how to identify a bash code block.
In Markdown, a bash codeblock starts with <code>\`\`\`bash</code> or <code>\`\`\`sh</code>, and ends with <code>\`\`\`</code>.

So our extract rule needs to extract everything between those delimiters so that we can treat it as bash and use our bash rule on it.

```yaml
rules:
  - id: extract-markdown-code-block
    mode: extract
    languages:
      - generic
    pattern-either:
      - pattern: |
          ```sh$...CMD```
      - pattern: |
          ```bash$...CMD```
    extract: $...CMD
    dest-language: bash
```

This creates a new variable using the [elipsis operator](https://semgrep.dev/docs/writing-rules/pattern-syntax/#ellipsis-metavariables) to capture all the content between our delimiters. Our new variable is then treated as bash code for the purpose of running other rules.

When we run the Semgrep CLI this time, we can see that our bash rule including the fix now also applies to our README.

```sh
$ semgrep scan --config rule.yaml --autofix --dryrun

┌─────────────────┐
│ 2 Code Findings │
└─────────────────┘

    README.md
       bash-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ code-buddy run YOURAPP
           13┆ code-buddy run YOURAPP

    run-app.sh
       bash-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ code-buddy run mycoolapp
            3┆ code-buddy run mycoolapp

Ran 2 rules on 9 files: 2 findings.
```

Let's try writing a rule for Go code next.

## Parsing Go expressions

Looking at [our example Go code](https://github.com/lucasmelin/semgrep-deprecation-demo/blob/main/main.go), there's a small hiccup.
Our `code-buddy` tool is being run with the `--please` flag, but the `os/exec` import in our `main.go` file is aliased to `runner`.

We could write two rules, one for the regular `exec` import used in `main_test.go`, and one for this newly found `runner` import.
But what we'd really like is to write a **generic** rule that can handle both cases.

Thankfully, Semgrep doesn't just parse lines, but it understands the syntax of Go code.
This means that we can write just one rule using the **original** import name, and Semgrep will understand that this applies to the **alias** form as well.

```yaml
rules:
  - id: go-deprecated-please-flag
    languages:
      - golang
    message: >
      code-buddy[🤖]: The --please flag is deprecated.
    pattern: |
      exec.Command("code-buddy", "run", $X, "--please")
    severity: WARNING
```

Running this using the Semgrep CLI, we can confirm that the rule detects `runner.Command` as being an alias for `exec.Command`:

```sh {hl_lines=["14-24"]}
$ semgrep scan --config rule.yaml

┌─────────────────┐
│ 4 Code Findings │
└─────────────────┘

    README.md
       bash-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ code-buddy run YOURAPP
           13┆ code-buddy run YOURAPP --please

    main.go
       go-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           18┆ cmd := runner.Command("code-buddy", "run", program, "--please")

    main_test.go
       go-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           10┆ cmd := exec.Command("code-buddy", "run", "demo", "--please")

    run-app.sh
       bash-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ code-buddy run mycoolapp
            3┆ code-buddy run mycoolapp --please

Ran 3 rules on 9 files: 4 findings.
```

But what about the fix?
Unfortunately, Semgrep's autofix currently operates on strings instead of the syntax tree, so it doesn't understand the alias.

In this particular instance, we'll need to create a regex fix.

```yaml {hl_lines=["10-11"]}
rules:
  - id: go-deprecated-please-flag
    languages:
      - golang
    message: >
      code-buddy[🤖]: The --please flag is deprecated.
    pattern-either:
      - pattern: |
          exec.Command("code-buddy", "run", $X, "--please")
    fix-regex:
      regex: 'Command\("code-buddy", "run", (.*), "--please"\)'
    severity: WARNING
```

Since our regex operates only on the part of the code that matched our pattern, we can omit the `exec` alias.
Our regex says to find the literal string `Command("code-buddy", "run", ` and then save the next characters up until the first `,` into a capture group, and finally our string must end with `"--please")` to be considered a match.

The `\` characters ensure that the parentheses are treated as literal parentheses, and not as the start of a capture group.

With this, we can write the replacement we want to make:

```yaml {hl_lines=["12"]}
rules:
  - id: go-deprecated-please-flag
    languages:
      - golang
    message: >
      code-buddy[🤖]: The --please flag is deprecated.
    pattern-either:
      - pattern: |
          exec.Command("code-buddy", "run", $X, "--please")
    fix-regex:
      regex: 'Command\("code-buddy", "run", (.*), "--please"\)'
      replacement: 'Command("code-buddy", "run", \1)'
    severity: WARNING
```

Our replacement writes the literal string `Command("code-buddy", "run", ` then pastes in the contents of our capture group, and then finishes the string by adding `)` to the end.

Running the Semgrep CLI using the `--autofix` and `--dryrun` flags, we can see how this fix will be applied:

```sh {hl_lines=["14-26"]}
$ semgrep scan --config rule.yaml --autofix --dryrun

┌─────────────────┐
│ 4 Code Findings │
└─────────────────┘

    README.md
       bash-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ code-buddy run YOURAPP
           13┆ code-buddy run YOURAPP

    main.go
       go-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ s/Command\("code-buddy", "run", (.*), "--please"\)/Command("code-buddy", "run", \1)/g
           18┆ cmd := runner.Command("code-buddy", "run", program)

    main_test.go
       go-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ s/Command\("code-buddy", "run", (.*), "--please"\)/Command("code-buddy", "run", \1)/g
           10┆ cmd := exec.Command("code-buddy", "run", "demo")

    run-app.sh
       bash-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ code-buddy run mycoolapp
            3┆ code-buddy run mycoolapp

Ran 3 rules on 9 files: 4 findings.
```

With that, we can move on to our last task, adding a rule for detecting our deprecated flag inside of a GitHub Actions workflow.

## Using nested patterns for GitHub Actions workflows

For this task, we're looking for [a bash command nested inside of a YAML file](https://github.com/lucasmelin/semgrep-deprecation-demo/blob/6e82638b88e109c2d8709a654f123542e6ebcf1c/.github/workflows/test.yml#L12-L13).
We could write another `extract` rule to tell Semgrep where the bash script is nested inside our YAML file, or we could teach Semgrep about the structure of GitHub Actions workflow files so that we can precisely target our bash command.

Again, let's go with option two.

We'll start with laying out the basic structure of a new YAML rule:

```yaml
rules:
  - id: github-actions-deprecated-please-flag
    languages:
      - yaml
    message: >
      code-buddy[🤖]: The --please flag is deprecated.
    severity: WARNING
```

Taking a look at [our GitHub Actions workflow](https://github.com/lucasmelin/semgrep-deprecation-demo/blob/main/.github/workflows/test.yml), the first thing to notice is that all of the actual commands are nested in `steps`.
So let's add this structure to our rule as a pattern:

```yaml {hl_lines=["7-8"]}
rules:
  - id: github-actions-deprecated-please-flag
    languages:
      - yaml
    message: >
      code-buddy[🤖]: The --please flag is deprecated.
    patterns:
      - pattern-inside: "steps: [...]"
    severity: WARNING
```

Notice that we use `pattern-inside` because we don't want to just match on the `steps`, but we want to scope the other patterns we'll define next to only be found within `steps`.

Next, we can see that individual steps have a `name`, some optional fields like `id` or `env`, and then in the case of bash steps, a `run` field.

We can capture this structure in a new pattern:

```yaml {hl_lines=["9-12"]}
rules:
  - id: github-actions-deprecated-please-flag
    languages:
      - yaml
    message: >
      code-buddy[🤖]: The --please flag is deprecated.
    patterns:
      - pattern-inside: "steps: [...]"
      - pattern-inside: |
        - name: ...
          ...
          run: ...
    severity: WARNING
```

We're starting to narrow in on the specific code we want to match.
Now that we're focused on a single step, we want to focus closer on the `run` field,
since that's the field that will contain our bash code.

We'll add this as a pattern as well:

```yaml {hl_lines=["13"]}
rules:
  - id: github-actions-deprecated-please-flag
    languages:
      - yaml
    message: >
      code-buddy[🤖]: The --please flag is deprecated.
    patterns:
      - pattern-inside: "steps: [...]"
      - pattern-inside: |
        - name: ...
          ...
          run: ...
      - pattern-inside: "run: $SHELL"
    severity: WARNING
```

With these three patterns, any bash scripts used in our GitHub Actions workflow will be captured in the `$SHELL` variable.

We can now write a test for this variable to see if it contains a call to our `code-buddy` CLI along with the deprecated `--please` flag.

```yaml {hl_lines=["14-17"]}
rules:
  - id: github-actions-deprecated-please-flag
    languages:
      - yaml
    message: >
      code-buddy[🤖]: The --please flag is deprecated.
    patterns:
      - pattern-inside: "steps: [...]"
      - pattern-inside: |
        - name: ...
          ...
          run: ...
      - pattern-inside: "run: $SHELL"
      - metavariable-pattern:
          language: bash
          metavariable: $SHELL
          pattern: code-buddy run $X --please
    severity: WARNING
```

Let's run this new rule against our codebase to confirm that it detects the `--please` flag in our GitHub Actions workflows.

```sh {hl_lines=["7-11"]}
$ semgrep scan --config rule.yaml

┌─────────────────┐
│ 5 Code Findings │
└─────────────────┘

    .github/workflows/test.yml
       github-actions-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           13┆ run: code-buddy run someapp --please

    README.md
       bash-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ code-buddy run YOURAPP
           13┆ code-buddy run YOURAPP --please

    main.go
       go-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ s/Command\("code-buddy", "run", (.*), "--please"\)/Command("code-buddy", "run", \1)/g
           18┆ cmd := runner.Command("code-buddy", "run", program, "--please")

    main_test.go
       go-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ s/Command\("code-buddy", "run", (.*), "--please"\)/Command("code-buddy", "run", \1)/g
           10┆ cmd := exec.Command("code-buddy", "run", "demo", "--please")

    run-app.sh
       bash-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ code-buddy run mycoolapp
            3┆ code-buddy run mycoolapp --please

Ran 4 rules on 9 files: 5 findings.
```

One last thing before we move on to the fix - we want to make sure that our fix is applied just to the bash script, and not to any of the surrounding YAML.

To do this, we can [specify a `focus-metavariable`](https://semgrep.dev/docs/writing-rules/rule-syntax/#focus-metavariable) which tells Semgrep that we're only interested in the code matched by the `$SHELL` variable, and not the `run: ` portion that precedes it.

```yaml {hl_lines=["18"]}
rules:
  - id: github-actions-deprecated-please-flag
    languages:
      - yaml
    message: >
      code-buddy[🤖]: The --please flag is deprecated.
    patterns:
      - pattern-inside: "steps: [...]"
      - pattern-inside: |
          - name: ...
            ...
            run: ...
      - pattern-inside: "run: $SHELL"
      - metavariable-pattern:
          language: bash
          metavariable: $SHELL
          pattern: code-buddy run $X --please
      - focus-metavariable: $SHELL
    severity: WARNING
```

If we create a regular `fix` borrowing from our bash rule, watch what happens when we attempt the dry run:

```yaml {hl_lines=["19-20"]}
rules:
  - id: github-actions-deprecated-please-flag
    languages:
      - yaml
    message: >
      code-buddy[🤖]: The --please flag is deprecated.
    patterns:
      - pattern-inside: "steps: [...]"
      - pattern-inside: |
          - name: ...
            ...
            run: ...
      - pattern-inside: "run: $SHELL"
      - metavariable-pattern:
          language: bash
          metavariable: $SHELL
          pattern: code-buddy run $X --please
      - focus-metavariable: $SHELL
    # This won't work the way we want!
    fix: code-buddy run $X
    severity: WARNING
```

When we run our rules with Semgrep, it shows that it's going to replace our line `run: code-buddy run someapp --please` with `run: code-buddy run $X`.

```sh {hl_lines=["7-12"]}
$ semgrep scan --config rule.yaml --autofix --dryrun

┌─────────────────┐
│ 5 Code Findings │
└─────────────────┘

    .github/workflows/test.yml
       github-actions-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ code-buddy run $X
           13┆ run: code-buddy run $X

    README.md
       bash-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ code-buddy run YOURAPP
           13┆ code-buddy run YOURAPP

    main.go
       go-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ s/Command\("code-buddy", "run", (.*), "--please"\)/Command("code-buddy", "run", \1)/g
           18┆ cmd := runner.Command("code-buddy", "run", program)

    main_test.go
       go-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ s/Command\("code-buddy", "run", (.*), "--please"\)/Command("code-buddy", "run", \1)/g
           10┆ cmd := exec.Command("code-buddy", "run", "demo")

    run-app.sh
       bash-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ code-buddy run mycoolapp
            3┆ code-buddy run mycoolapp

Ran 4 rules on 9 files: 5 findings.
```

That's not what we want - why isn't Semgrep replacing the `$X` variable with `someapp`?

Our `$SHELL` metavariable is propagating up to the `fix`, but we don't actually have a metavariable pattern defined for `$X`.
So this nested variable is not in scope for the `fix` to be able to use.

Earlier however, we figured out how to create regex rules, so this doesn't prevent us from creating the fix:

```yaml {hl_lines=["19-21"]}
rules:
- id: github-actions-deprecated-please-flag
    languages:
      - yaml
    message: >
      code-buddy[🤖]: The --please flag is deprecated.
    patterns:
      - pattern-inside: "steps: [...]"
      - pattern-inside: |
          - name: ...
            ...
            run: ...
      - pattern-inside: "run: $SHELL"
      - metavariable-pattern:
          language: bash
          metavariable: $SHELL
          pattern: code-buddy run $X --please
      - focus-metavariable: $SHELL
    fix-regex:
      regex: 'code-buddy run (.*) --please'
      replacement: 'code-buddy run \1'
    severity: WARNING
```

With that, we can verify our final rule using Semgrep.

```sh {hl_lines=["7-12"]}
$ semgrep scan --config rule.yaml --autofix --dryrun

┌─────────────────┐
│ 5 Code Findings │
└─────────────────┘

    .github/workflows/test.yml
       github-actions-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ s/code-buddy run (.*) --please/code-buddy run \1/g
           13┆ run: code-buddy run someapp

    README.md
       bash-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ code-buddy run YOURAPP
           13┆ code-buddy run YOURAPP

    main.go
       go-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ s/Command\("code-buddy", "run", (.*), "--please"\)/Command("code-buddy", "run", \1)/g
           18┆ cmd := runner.Command("code-buddy", "run", program)

    main_test.go
       go-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ s/Command\("code-buddy", "run", (.*), "--please"\)/Command("code-buddy", "run", \1)/g
           10┆ cmd := exec.Command("code-buddy", "run", "demo")

    run-app.sh
       bash-deprecated-please-flag
          code-buddy[🤖]: The --please flag is deprecated.

           ▶▶┆ Autofix ▶ code-buddy run mycoolapp
            3┆ code-buddy run mycoolapp

Ran 4 rules on 9 files: 5 findings.
```

We can now run all of these fixes without the `--dryrun` flag to update all of our files in one go.

These new rules can also be reused across any number of repositories to automate detecting and resolving all usages of our deprecated `--please` flag.

## Wrap-up

We've learned how to create Semgrep rules to find and fix deprecated usages of our `code-buddy` CLI across bash scripts, Markdown files, Go code and GitHub Actions workflows.

We now have an automated way to apply these fixes to as many internal repositories as we need.
We could also add this as a CI check to notify teams about the deprecation, offering them an easy way to perform the migration when they're ready by just running `semgrep scan --config rule.yaml --autofix`.

Although getting used to a new tool can take some effort, hopefully you can see how this technique can be very powerful for automating complex changes across multiple repositories.
