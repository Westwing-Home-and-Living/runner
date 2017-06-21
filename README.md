# Runner

[![Travis](https://travis-ci.org/stylemistake/runner.svg)][travis]
[![NPM](https://badge.fury.io/js/bash-task-runner.svg)][npm]
[![Gitter](https://badges.gitter.im/stylemistake/bash-task-runner.svg)][gitter]

> A simple, lightweight task runner for Bash.

Runner was made to replace Make in those few use cases, when all you need is a
bunch of `.PHONY` targets that are simple shell scripts. It uses the familiar
Bash syntax and only depends on `bash` and `coreutils`.

In addition, Runner provides tools to run tasks in parallel for improved
performance, nice logging facilities and error handling.


<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## Table of contents

- [1. Dependencies](#1-dependencies)
- [2. Installation](#2-installation)
  - [2.1. Simple (vendored)](#21-simple-vendored)
  - [2.2. Submodule (vendored)](#22-submodule-vendored)
  - [2.3. Homebrew](#23-homebrew)
  - [2.4. NPM](#24-npm)
  - [2.5. Git + PATH](#25-git--path)
- [3. CLI](#3-cli)
  - [3.1. Autocompletion](#31-autocompletion)
  - [3.2. Flag propagation](#32-flag-propagation)
- [4. Runnerfile](#4-runnerfile)
  - [4.1. Naming convention](#41-naming-convention)
  - [4.2. Default task](#42-default-task)
  - [4.3. Task chaining](#43-task-chaining)
  - [4.4. Error handling](#44-error-handling)
- [5. Function reference](#5-function-reference)
  - [`runner_log [message]`](#runner_log-message)
  - [`runner_colorize <color> [message]`](#runner_colorize-color-message)
  - [`runner_run [command]`](#runner_run-command)
  - [`runner_get_defined_tasks`](#runner_get_defined_tasks)
  - [`runner_is_defined <name>`](#runner_is_defined-name)
  - [`runner_is_task_defined [task ...]`](#runner_is_task_defined-task-)
  - [`runner_sequence [task ...]`](#runner_sequence-task-)
  - [`runner_parallel [task ...]`](#runner_parallel-task-)
  - [`runner_bootstrap`](#runner_bootstrap)
- [FAQ](#faq)
- [Contribution](#contribution)
- [License](#license)
- [Contacts](#contacts)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


## 1. Dependencies

Runner depends on:

- bash `>=4.2`
- coreutils `>=8.0`

For non-GNU environments, it also depends on:

- perl `>=5.0` (for resolving symlinks)

**Note for macOS users:**

Use [Homebrew] to install the dependencies:

```bash
brew install bash coreutils
```


## 2. Installation

Each of the below installation methods is differentiated along two properties:

- **Local to project**
  - Whether Runner will be installed locally to your project or globally on a
    system.
  - This is good for CI builds and spinning up multiple people on your project
- **CLI-enabled**
  - Whether you will be able to use the `runner` command from your prompt.
  - Useful for local development, tab completion, and convenience.

You may want to combine multiple installation methods in order to satisfy both
of these requirements. In particular, we recommend [**Simple
(vendored)**](#simple-vendored) with a method that gives you a CLI and is
compatible with your system.

|                      | Local to Project   | CLI-enabled        |
| ---                  | ---                | ---                |
| Simple (vendored)    | :white_check_mark: | :no_entry_sign:    |
| Submodule (vendored) | :white_check_mark: | :white_check_mark: |
| Homebrew             | :no_entry_sign:    | :white_check_mark: |
| NPM                  | :white_check_mark: | :white_check_mark: |
| Git + PATH           | :no_entry_sign:    | :white_check_mark: |

### 2.1. Simple (vendored)

Just drop `src/runner.sh` anywhere in your project folder:

```bash
wget https://raw.githubusercontent.com/stylemistake/runner/master/src/runner.sh
```

Then skip to [Runnerfile](#4-cli) for how to use a vendored Runner installation.

### 2.2. Submodule (vendored)

If you'd like a slightly better story around updating Runner when vendored, you
can use a Git submodule, if you're [familiar with submodules][submodules]:

```bash
git submodule add https://github.com/stylemistake/runner
```

> Note that if submodules are too heavy-handed, you can get the same effect
> (without the ease of updating) by just unzip'ing Runner's source into your
> project.

You should now be able to access `runner.sh` within the submodule. Additionally,
you can access the CLI with `./runner/bin/runner`. You can make this more
ergonomic by altering your PATH:

```bash
export PATH="$PATH:./runner/bin"
```

Then skip to [CLI](#3-cli) to learn how to use the CLI.

### 2.3. Homebrew

On OS X, installing Runner globally is simple if you have Homebrew:

```bash
brew install stylemistake/runner-brew/runner
```

Then skip to [CLI](#3-cli) to learn how to use the CLI.

### 2.4. NPM

If you don't mind the additional dependency on the NPM ecosystem, you can
install Runner with NPM:

```bash
# --- Local to Project --- #
npm install --save bash-task-runner

# to enable CLI:
export PATH="PATH:./node_modules/.bin"

# --- Global --- #
npm install -g bash-task-runner
```

Then skip to [CLI](#3-cli) to learn how to use the CLI.

### 2.5. Git + PATH

If Runner is not available in a package manager for your system, you can clone
Runner to your computer, and adjust your PATH to contain the installation
location:

```bash
git clone https://github.com/stylemistake/runner

export PATH="$PATH:$(pwd)/runner/bin"
```

Then skip to [CLI](#3-cli) to learn how to use the CLI.


## 3. CLI

NOTE: Please see `runner -h` for complete, up-to-date CLI usage information.

```
Usage: runner [options] [task] [task_options] ...
Options:
  -C <dir>, --directory=<dir>  Change to <dir> before doing anything.
  --completion=<shell>         Output code to activate task completions.
                               Supported shells: 'bash'.
  -f <file>, --file=<file>     Use <file> as a runnerfile.
  -l, --list-tasks             List available tasks.
  -h, --help                   Print this message and exit.
```

### 3.1. Autocompletion

The `runner` CLI supports autocompletion for task names and flags.
Add the following line your `~/.bashrc`:

```bash
eval $(runner --completion=bash)
```

### 3.2. Flag propagation

All flags you pass after the task name are passed to your tasks.

```bash
$ runner foo --production

task_foo() {
    echo ${@} # --production
}
```

To pass options to the `runner` CLI specifically, you must provide them
before any task names:

```bash
$ runner -f scripts/tasks.sh foo
```


## 4. Runnerfile

Runner works in conjunction with a `runnerfile.sh`. A basic runnerfile looks
like this:

```bash
task_foo() {
  ## Do something...
}

task_bar() {
  ## Do something...
}
```

Invoke Runner using `runner [task ...]`:

```bash
$ runner foo bar
[23:43:37.754] Starting 'foo'
[23:43:37.755] Finished 'foo' after 1 ms
[23:43:37.756] Starting 'bar'
[23:43:37.757] Finished 'bar' after 1 ms
```

**Optional**: If you want the `runnerfile.sh` to be a standalone script, add
this to the beginning (works best in conjunction with a vendored installation):

```bash
#!/usr/bin/env bash
cd "$(dirname "$0")" || exit
source <path_to>/runner.sh
```

To invoke such script, use `bash runnerfile.sh [task ...]`.

### 4.1. Naming convention

Your Runnerfile can be named any of the following. Using a `.sh` suffix helps
with editor syntax highlighting.

- `Runnerfile`
- `Runnerfile.sh`
- `runnerfile`
- `runnerfile.sh`

### 4.2. Default task

You can specify a default task in your Runnerfile. It will run when no arguments
are provided. There are two ways to do this:

```bash
task_default() {
  # do something ...
}
```

```bash
runner_default_task="foo"
task_foo() {
  # do something ...
}
```

### 4.3. Task chaining

Tasks can launch other tasks in two ways: *sequentially* and in *parallel*.
This way you can optimize the task flow for maximum concurrency.

To run tasks sequentially, use:

```bash
task_default() {
    runner_sequence foo bar
    ## [23:50:33.194] Starting 'foo'
    ## [23:50:33.195] Finished 'foo' after 1 ms
    ## [23:50:33.196] Starting 'bar'
    ## [23:50:33.198] Finished 'bar' after 2 ms
}
```

To run tasks in parallel, use:

```bash
task_default() {
    runner_parallel foo bar
    ## [23:50:33.194] Starting 'foo'
    ## [23:50:33.194] Starting 'bar'
    ## [23:50:33.196] Finished 'foo' after 2 ms
    ## [23:50:33.196] Finished 'bar' after 2 ms
}
```

### 4.4. Error handling

Sometimes you need to stop the task if one of the commands fails.
You can achieve this with a conditional return:

```bash
task_foo() {
    ...
    php composer.phar install || return
    ...
}
```

If a failed task was a part of a sequence, the whole sequence fails. Same
applies to the tasks running in parallel.

Notice that you should use this pattern for the whole sequence too to ensure
no further code is executed afterwards and the overall return code is
correctly set:

```bash
task_default() {
    ...
    runner_sequence foo bar || return
    ...
    echo "Won't show up on error above"
}
```


## 5. Function reference

### `runner_log [message]`

Prints a message with a timestamp. Variations of log with colors:

- `runner_log_error` (red)
- `runner_log_warning` (yellow)
- `runner_log_success` (green)
- `runner_log_notice` (gray)

### `runner_colorize <color> [message]`

Colorizes the message into the specified color. Here's a list of colors:

- `black`
- `red`
- `green`
- `yellow`
- `blue`
- `purple`
- `cyan`
- `light_gray`
- `gray`
- `light_red`
- `light_green`
- `light_yellow`
- `light_blue`
- `light_purple`
- `light_cyan`
- `white`
- `reset`

### `runner_run [command]`

`runner_run` command gives a way to run commands and have them outputted:

```bash
task_default() {
    runner_run composer install
    ## [12:19:17.170] Starting 'default'...
    ## [12:19:17.173] composer install
    ## Loading composer repositories with package information
    ## ...
    ## [12:19:17.932] Finished 'default' after 758 ms
}
```

### `runner_get_defined_tasks`

Lists all functions beginning with `task_`.

### `runner_is_defined <name>`

Checks if function is defined or program is accessible from current `$PATH`.

### `runner_is_task_defined [task ...]`

Checks if task name is defined.

### `runner_sequence [task ...]`

Runs tasks sequentially. If any task in the sequence fails, it stops execution
and returns an error code of a failed task.

### `runner_parallel [task ...]`

Runs tasks in parallel, and lets them finish even if any error occurs. In case
of an error, this command returns special error codes to determine how many
tasks have failed.

Error codes:

- `1` - one task failed
- `2` - some tasks failed
- `3` - all tasks failed

### `runner_bootstrap`

Bootstraps the task runner. This can be used to override the default mechanism
of startup.

By default, task runner starts up when it reaches the end of a Runnerfile.
By using `runner_bootstrap`, you can manually choose a point where it begins
to run tasks:

```bash
task_default() {
    ## Do things...
}

runner_bootstrap

if [[ ${?} -eq 0 ]]; then
    echo "Success! :)"
else
    echo "Failure! :("
fi
```

In example above, we used `runner_bootstrap` to handle the task runner's exit
code. You can use this to handle errors, do cleanup work or restart certain
tasks when needed.


## FAQ

Read the [FAQ]


## Contribution

Please provide pull requests in a separate branch (other than `master`), this
way it's easier for me to review and pull changes.

Before writing code, open an [issue][issues] to get initial feedback.


## License

This software is covered by GNU Lesser General Public License v3 (LGPL-3.0).
See [LICENSE.md].


## Contacts

Style Mistake <[stylemistake@gmail.com]>


[travis]: https://travis-ci.org/stylemistake/runner
[gitter]: https://gitter.im/stylemistake/bash-task-runner
[npm]: https://www.npmjs.com/package/bash-task-runner
[homebrew]: http://brew.sh/
[src/runner.sh]: https://raw.githubusercontent.com/stylemistake/runner/master/src/runner.sh
[issues]: https://github.com/stylemistake/runner/issues
[manuel]: https://github.com/ShaneKilkelly/manuel
[faq]: https://github.com/stylemistake/runner/wiki/FAQ
[LICENSE.md]: LICENSE.md
[stylemistake@gmail.com]: mailto:stylemistake@gmail.com
