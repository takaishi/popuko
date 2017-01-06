# popuko

[![Build Status (master)](https://travis-ci.org/karen-irc/popuko.svg?branch=master)](https://travis-ci.org/karen-irc/popuko)

## What is this?

- This is an operation bot to do these things automatically.
    - GitHub
        - merge a pull request automatically.
        - assign a pull request to a reviewer.
        - patrol a pull request which are newly unmergeable by others.
- Almost reimplementation of [homu](https://github.com/barosl/homu).


## Motivation

[homu](https://github.com/barosl/homu) is the super great operation bot for development on GitHub
and it supports a lot of valuable features: merge pull request into the latest upstream, try to testing on TravisCI,
and more. But its development is not in active now. And also Mozilla's servo team maintains
[their forked version of homu](https://github.com/servo/homu). But it is developed for their specific usecase.
Not for other projects.

This project intent to re-implement homu with minimum features to support our projects for work including non-public activities,
and to simplify to deploy this bot, and challenge to deploy this bot easily.

These are why we have developed this project.


## Features

These features are inspired by [homu](https://github.com/barosl/homu) and [highfive](https://github.com/servo/highfive).

- __Change the labels, the assignees of the pull request by comments__
    - By a reviewer's comment, this bot changes the labels, the assignees of the pull request.
- __Patrol pull requests which cannot merge into it after the upstream has been updated__
    - This bot patrols automatically by hooking GitHub's push events.
    - And change the label for the unmergeable pull request.
- __Try the pull request with the latest `master` branch, and merge into it automatically__
    - We call this feature as "Auto-Merging".
- __Specify a reviewer by a file committed to the repository__
    - This feature is not implemented by homu.
    - You can manage a reviewer by normal pull request process for open governance.


### Command

You can use these command as the comment for pull request.

- `r? @<reviewer>` or `@<reviewer> r?`
    - Assign the reviewer to the pull request with labeling `S-awaiting-review`.
- `@<botname> r+` or `@<botname> r=<reviewer>`
    - Merge this pull request by `<botname>` with labeling `S-awaiting-merge`.
    - If you enable Auto-Merging, this bot queues the pull request into the approved queue.
      And this bot
- `@<botname> r-`
    - Cancel the approved by `@<botname> r+`.
    - This set back the label to `S-awaiting-review`


### Auto-Merging

This bot provides a powerful feature we called as "Auto-Merging".
Auto-Merging behaves like this:

1. Accept the pull request by the review's approved comment (e.g. `@bot r+`)
2. This bot queues its pull request into the approved queue.
3. If there is no active item, try to merge it into the latest upstream and run CI on the special "auto" branch.
4. If the result of step 3 is success, this bot merge its pull request into the upstream actually.
   Otherwise, this bot marks it as failed.
5. This bot redo step 3 until the approved queue will be empty.



## Setup Instructions

### Build & Launch the Application

1. Build from source file
    - You can do `go get`.
2. Create the config directory.
    - By default, this app uses `$XDG_CONFIG_HOME/popuko/` as the config dir.
      (If you don't set `$XDG_CONFIG_HOME` environment variable, this use `~/.config/popuko/`.)
    - You can configure the config directory by `--config-base-dir`
3. Set `config.toml` to the config directory.
    - Let's copy from [`./example.config.toml`](./example.config.toml)
4. Start the exec binary.
    - This app dumps all logs into stdout & stderr.

#### Set up for your repository in GitHub.

1. Set the account (or the team which it belonging to) which this app uses as a collaborator
   for your repository (requires __write__ priviledge).
2. Add `OWNERS.json` file to the root of your repository.
    - Please see [`OwnersFile`](./setting/ownersfile.go) about the detail.
    - The example is [here](./OWNERS.json).
3. Set `http://<your_server_with_port>/github` for the webhook to your repository with these events:
    - `Issue comment`
    - `Push`
    - `Status` (required to use Auto-Merge feature).
4. Create these labels to make the status visible.
    - `S-awaiting-review`
        - for a pull request assigned to some reviewer.
    - `S-awaiting-merge`
        - for a pull request queued to this bot.
    - `S-needs-rebase`
        - for an unmergeable pull request.
    - `S-fails-tests-with-upstream`
        - for a pull request which fails tests after try to merge into upstream (used by Auto-Merge feature).
6. Enable to start the build on creating the branch named `auto` for your CI service (e.g. TravisCI).
7. Done!


## Why there is no released version?

- __This project always lives in canary__.
- We only support the latest revision.
- All of `master` branch is equal to our released version.
- The base revision and build date are embedded to the exec binary. You can see them by checking stdout on start it.


## The current limitations

- The upstream branch should be named as `master`.
- This bot cannot handle the pull request which aims to merge into not `master` correctly.
- This bot uses `auto` branch for the origin repository. You cannot configure the name.


## TODO

- Intelligent cooperation with TravisCI.
- [See more...](https://github.com/karen-irc/popuko/issues)
