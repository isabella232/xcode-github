# xcode-github

A macOS app that creates new Xcode bots when a new PR is created on an observed GitHub repo.

There are projects for a full Mac app and another for a command line utility.

* [To Do](#to-do)
* [Overview](#overview)
* [Command Line and Help](#command-line-and-help)
* [Program Flow](#program-flow)
* [References](#references)
* [Xcode Bot Documentation](#xcode-bot-documentation)
* [GitHub Documentation](#github-documentation)
* [Note Pad](#note-pad)

## To Do
```
* [x] Parse command line options.
* [x] Update Xcode server status.
* [x] Create Xcode bots from new PRs.
* [x] Start rough version of the Mac app.
* [ ] Update integration status in GitHub.
* [ ] Clean up command line app logging and messages.
* [ ] Polish Documentation.
```

## Overview

This command line tool creates a new Xcode test bot when a new PR on a GitHub repo is created.

When a new PR is created, this app creates a new Xcode test bot on an Xcode server based on an existing template bot and schedules it to run. The bot will report the test status back on GitHub.

## Command Line and Help
[Standard command line parsers.](https://stackoverflow.com/questions/9642732/parsing-command-line-arguments)

```
xcode-github - Creates an Xcode test bots for new GitHub PRs.

usage: xcode-github [-dhsVv] -g <github-auth-token>
                 -t <bot-template> -x <xcode-server-domain-name>


  -d, --dryrun
      Dry run. Print what would be done.

  -g, --github <github-auth-token>
      A GitHub auth token that allows checking the status of a repo
      and change a PR's status.

  -h, --help
      Print this help information.

  -s, --status
      Only print the status of the xcode server bots and quit.

  -t --template <bot-template>
      An existing bot on the xcode server that is used as a template
      for the new GitHub PR bots.

  -V, --version
      Show version and exit.

  -v, --verbose
      Verbose. Extra 'v' increases the verbosity.

  -x, --xcodeserver <xcode-server-domain-name>
      The network name of the xcode server.
```

## Program Flow

```
GitHub "New PR" Event -> xcode-github clones a new Xcode bot based on an existing bot
    Changes GitHub status to pending
    Xcode runs the test bot
    Changes GitHub status to success or fail.
GitHub "Close PR" Event -> xcode-github deletes Xcode Bot
```

`xcode-github` Flow

1. Get the latest bots from Xcode server.
    * Make sure the template bot exists.
1. Git the latest PRs for the repo.
1. Loop through PRs:
    * If PR doesn't have a bot, create bot based on the template bot.
1. Loop through bots:
    * If a bot is one of 'our' bots, and the PR is not open, delete it.

### Xcode Bot Flow
* Test start: Change GitHub PR status to 'Pending'.
* Run tests.
* Test end: Change GitHub PR status to 'Passed' or 'Failed'.

## References

### Xcode Bot Documentation

[Xcode Bot Documentation](https://developer.apple.com/library/content/documentation/Xcode/Conceptual/XcodeServerAPIReference/Bots.html)

#### List All Bots

        GET https://server.mycompany.com:20343/api/bots
        curl -k https://esmith.local:20343/api/bots

#### Copy Bot from Template

        POST https://server.mycompany.com:20343/api/bots/{bot-id}/duplicate
        Headers
            Content-Type: application/json
        Body
             The properties to be set after the bot has been duplicated.

```
curl --insecure --request POST \
    --header 'Content-Type: application/json' \
    --data '{ "name": "My new bot" }' \
    https://esmith.local:20343/api/bots/1a023fbac7f749ede507153bb43e75e3
```


#### Delete Bot
    DELETE https://server.mycompany.com:20343/api/bots/{bot-id}
    curl --insecure --request DELETE https://esmith.local:20343/api/bots/1a023fbac7f749ede507153bb43e75e3`

#### Get Bot Status
    GET https://esmith.local:20343/api/bots/{bot-id}/integrations?last=1
    curl -k https://esmith.local:20343/api/bots/1a023fbac7f749ede507153bb43d6878/integrations?last=1

#### Start Integration
    POST https://esmith.local:20343/api/bots/{bot-id}/integrations
```
    {
        shouldClean: true
    }
```

### GitHub Documentation
* [GitHub API Documentation](https://developer.github.com/v3/)
* [GitHub CI Server Documentation](https://developer.github.com/v3/guides/building-a-ci-server/)

#### Get Pull Requests for a Repo
```
    curl \
        --header 'Accept: application/vnd.github.v3+json' \
        --header 'Authorization: token 13e499f7d9ba4fca42e4715558d1e5bc30a6a4e9' \
        https://api.github.com/repos/BranchMetrics/ios-branch-deep-linking/pulls?state=open\&sort=created\&direction=desc \
        | prettyjson
```

#### Update Pull Request Status
[Pull Request Status Documentation](https://developer.github.com/v3/repos/statuses/)

    POST /repos/:owner/:repo/statuses/:sha
```
    {
      "state": "success",
      "target_url": "https://example.com/build/status",
      "description": "The build succeeded!",
      "context": "continuous-integration/jenkins"
    }
```

#### List Pull Request Status
    GET /repos/:owner/:repo/commits/:ref/statuses
```
    curl \
        --header 'Accept: application/vnd.github.v3+json' \
        --header 'Authorization: token 13e499f7d9ba4fca42e4715558d1e5bc30a6a4e9' \
        https://api.github.com/repos/BranchMetrics/ios-branch-deep-linking/commits/push-notifications/statuses
```

## Note Pad
```
curl -k https://esmith.local:20343/api/bots | prettyjson
curl -k https://esmith.local:20343/api/bots/1a023fbac7f749ede507153bb43d6878/integrations?last=1 | prettyjson

Integration Link in Status Message:
  https://stlt.herokuapp.com/v1/xcs_deeplink/qabot.stage.branch.io/f58eba9902ec5f7f8dd96c518f88b617/fe902cbba44b59ff95b81a640158bc6f
    =>
  xcbot://qabot.stage.branch.io/botID/f58eba9902ec5f7f8dd96c518f88b617/integrationID/fe902cbba44b59ff95b81a640158bc6f
```

