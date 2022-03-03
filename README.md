# reviewdog-action-prettier

![Maintained](https://img.shields.io/badge/maintained-yes-brightgreen) 
[![Test](https://github.com/EPMatt/reviewdog-action-prettier/workflows/Tests/badge.svg)](https://github.com/EPMatt/reviewdog-action-prettier/actions?query=workflow%3ATest)
[![Code Quality](https://github.com/EPMatt/reviewdog-action-prettier/workflows/Code%20Quality/badge.svg)](https://github.com/EPMatt/reviewdog-action-prettier/actions?query=workflow%3Areviewdog)
[![Deps auto update](https://github.com/EPMatt/reviewdog-action-prettier/workflows/Deps%20auto%20update/badge.svg)](https://github.com/EPMatt/reviewdog-action-prettier/actions?query=workflow%3Adepup)
[![Release](https://github.com/EPMatt/reviewdog-action-prettier/workflows/Release/badge.svg)](https://github.com/EPMatt/reviewdog-action-prettier/actions?query=workflow%3Arelease)
[![GitHub release (latest SemVer)](https://img.shields.io/github/v/release/EPMatt/reviewdog-action-prettier?logo=github&sort=semver)](https://github.com/EPMatt/reviewdog-action-prettier/releases)
[![action-bumpr supported](https://img.shields.io/badge/bumpr-supported-ff69b4?logo=github&link=https://github.com/haya14busa/action-bumpr)](https://github.com/haya14busa/action-bumpr)

<a href="https://www.buymeacoffee.com/epmatt"><img width="150" alt="yellow-button" src="https://user-images.githubusercontent.com/30753195/133942263-5fef0166-4ab5-4529-b931-37b5d14f02bf.png"></a>

This GitHub Action runs [prettier](https://prettier.io/) with [reviewdog](https://github.com/reviewdog/reviewdog) to improve code checking, formatting and review experience for your codebase. :dog:

The action will first run Prettier, then passing the tool's output to reviewdog for further processing. Depending on the action configuration, reviewdog will then provide a GitHub check either with code annotations or with a Pull Request review.

For full documentation regarding reviewdog, its features and configuration options, please visit the [reviewdog repository](https://github.com/reviewdog/reviewdog).

## See it in action!

Check out the [Sample PR](https://github.com/EPMatt/reviewdog-action-prettier/pull/6) for an example usage of this action.

### With `github-check` or `github-pr-check` reporters

![reviewdog-action-prettier-github-check](https://user-images.githubusercontent.com/30753195/156574262-ea7f7969-d5c4-4fcc-8727-dcbe08fdb611.png)

If configured with `github-check` and `github-pr-check` reporters, the action will report Prettier errors and warnings about unformatted files.

As an example, see what the action reported for the Sample PR [here](https://github.com/EPMatt/reviewdog-action-prettier/pull/6/checks?check_run_id=5399599766).

### With `github-pr-review` reporter

![reviewdog-action-prettier-github-pr-review](https://user-images.githubusercontent.com/30753195/156574285-f6fd599d-af5c-470d-b23b-c7daa92d316f.png)

If configured with the `github-pr-review` reporter, the action submits a code review including any errors reported by Prettier; moreover, any formatting changes provided by Prettier are attached as code suggestions.

Check out the [Sample PR _Conversation_ tab](https://github.com/EPMatt/reviewdog-action-prettier/pull/6) to see how the action submitted a code review, including both error reports and formatting suggestions.
## Input

```yaml
inputs:
  github_token:
    description: 'GITHUB_TOKEN'
    default: '${{ github.token }}'
    required: false
  workdir:
    description: |
      Working directory relative to the root directory.
      This is where the action will look for a
      package.json which declares Prettier as a dependency.
      Please note that this is different from the directory
      Prettier will run on, which is defined in the prettier_flags input.
      Default is `.`.
    default: '.'
    required: false
  ### Flags for reviewdog ###
  level:
    description: |
      Report level for reviewdog [info,warning,error].
      Default is `error`.
    default: 'error'
    required: false
  reporter:
    description: |
      Reporter of reviewdog command [github-check,github-pr-check,github-pr-review].
      Default is `github-pr-check`.
    default: 'github-pr-check'
    required: false
  filter_mode:
    description: |
      Filtering mode for the reviewdog command [added,diff_context,file,nofilter].
      Default is `added`.
    default: 'added'
    required: false
  fail_on_error:
    description: |
      Exit code for reviewdog when errors are found [true,false].
      Default is `false`.
    default: 'false'
    required: false
  reviewdog_flags:
    description: |
      Additional reviewdog flags.
      Default is ``.
    default: ''
    required: false
  tool_name:
    description: 'Tool name to use for reviewdog reporter'
    default: 'prettier'
    required: false
  ### Flags for prettier ###
  prettier_flags:
    description: |
      Flags and args to pass to Prettier.
      If you override this input, please make sure to append to it the directory
      which Prettier will run on.
      The path provided here is relative to the workdir path, provided in the workdir input.
      Default is `.`, which makes Prettier run on the path provided in the workdir input.
    default: '.'
    required: false
```

## Usage

This example shows how to configure the action to run on any event occurring on a Pull Request. Reviewdog will report Prettier output messages by opening a code review on the Pull Request which triggered the workflow. Moreover, any formatting changes provided by Prettier will be attached as code suggestions.

```yaml
name: reviewdog
on: [pull_request]
jobs:
  prettier:
    name: runner / prettier
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: EPMatt/reviewdog-action-prettier@v1
        with:
          github_token: ${{ secrets.github_token }}
          # Change reviewdog reporter if you need
          # [github-pr-check,github-check,github-pr-review].
          # More about reviewdog reporters at
          # https://github.com/reviewdog/reviewdog#reporters
          reporter: github-pr-review
          # Change reporter level if you need
          # [info,warning,error].
          # More about reviewdog reporter level at
          # https://github.com/reviewdog/reviewdog#reporters
          level: warning
```

## FAQs

### How do I run the action on a module in a subfolder?

To run the action on a module in a subfolder, you can change the path where the action will run with the `workdir` input.

Please note that this setup is relevant only if your repository includes a module in a subfolder, declaring `prettier` as a dependency in its `package.json`. If you just want to run Prettier in a subfolder, read [this FAQ](#how-do-i-include-or-exclude-certain-files-and-folders) to learn how to include or exclude certain files and folders.

```yaml
name: reviewdog
on: [pull_request]
jobs:
  prettier:
    name: runner / prettier
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: EPMatt/reviewdog-action-prettier@v1
        with:
          github_token: ${{ secrets.github_token }}
          reporter: github-pr-review
          level: warning
          # The action will look for a package.json file with prettier
          # declared as a dependency located in the "foo" subfolder.
          workdir: foo
```

### How do I run the action on multiple modules?

For more complex setups which require to run the action on multiple modules, run the action once for each single module, changing the `workdir` and `prettier_flags` inputs accordingly.

Please note that this setup is relevant only if you have multiple submodules in your repository, each of them declaring `prettier` as a dependency in their `package.json`. If you just want to include or exclude certain files and folders from prettier, read [this FAQ](#how-do-i-include-or-exclude-certain-files-and-folders).

```yaml
name: reviewdog
on: [pull_request]
jobs:
  prettier:
    name: runner / prettier
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      # Run action for the "submodule1" module
      - uses: EPMatt/reviewdog-action-prettier@v1
        with:
          github_token: ${{ secrets.github_token }}
          reporter: github-pr-review
          level: warning
          # The action will look for a package.json file with prettier
          # declared as a dependency located in the "submodule1" subfolder.
          workdir: submodule1
      # Run action for the "submodule2" module
      - uses: EPMatt/reviewdog-action-prettier@v1
        with:
          github_token: ${{ secrets.github_token }}
          reporter: github-pr-review
          level: warning
          # The action will look for a package.json file with prettier
          # declared as a dependency located in the "submodule2" subfolder.
          workdir: submodule2
```

### How do I include or exclude certain files and folders?

You can select which files Prettier will run on by either configuring it with a [configuration file](https://prettier.io/docs/en/configuration.html) or by providing the appropriate [CLI flags](https://prettier.io/docs/en/cli.html).

When using this action, custom flags can be supplied to Prettier via the `prettier_flags` input.
Please note that the `file/dir/glob` parameter provided in `prettier_flags` input is relative to the path provided in the `workdir` input.

The following example workflow runs when a push is performed on the `main` branch. The `prettier` job runs the tool only on `.jsx` files included in the repository.

```yaml
name: reviewdog
on:
  push:
    branches:
      - main
jobs:
  prettier:
    name: runner / prettier
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: EPMatt/reviewdog-action-prettier@v1
        with:
          github_token: ${{ secrets.github_token }}
          reporter: github-pr-review
          level: warning
          # The action will look for a package.json file with prettier
          # declared as a dependency located in the repo root directory.
          workdir: .
          # Run only on files with the .jsx extension
          prettier_flags: *.jsx
```


### Why can't I see the results?

Try looking into the `filter_mode` options explained [here](https://github.com/reviewdog/reviewdog#filter-mode). Prettier errors and warnings will sometimes appear in lines or files that weren't modified by the commit the workflow run is associated with, which instead get filtered with the default `added` option.

### Why does the action require a `package.json` with `prettier` declared as a dependency?

The action will automatically install Prettier from a `package.json` in the `workdir` path. However, it will not choose a Prettier version and install it for you if `package.json` is missing in the provided `workdir` or it does not declare Prettier as a dependency.

Different Prettier versions might result in different formatting; indeed, [the official documentation](https://prettier.io/docs/en/install.html#summary) suggests to install an exact version in your project to avoid any formatting changes and issues due to non-identical Prettier versions. Therefore, the action leaves you to decide which version of Prettier should be executed. 

The action does not provide a custom input for specifying the Prettier version; instead, you should use a `package.json` file, and add `prettier` as a dependency. Please read the [Prettier installation guide](https://prettier.io/docs/en/install.html) for more.

Using a `package.json` file is the most practical and standardized way to declare Node dependencies, and it will also allow you to automate dependency updates and local installation with other tools.

## Contributing

Want to improve this action? Cool! :rocket: Please make sure to read the [Contribution Guidelines](CONTRIBUTING.md) prior submitting your work.

Any feedback, suggestion or improvement is highly appreciated!

## Sponsoring

If you want to show your appreciation and support maintenance and future development of this action, please consider **making a small donation [here](https://www.buymeacoffee.com/epmatt)**. :coffee:

Moreover, if you like this project don't forget to **leave a star on [GitHub](https://github.com/EPMatt/reviewdog-action-prettier)**. Such a quick and zero-cost act will allow the action to get more visibility across the community, resulting in more people getting to know and using it. :star:
