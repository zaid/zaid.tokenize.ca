---
title: "Tweaking the Copilot sandbox to make it work with ASDF"
date: 2026-11-23T17:45:32-05:00
tags: ["ai", "copilot"]
---

I set up Copilot CLI on my personal machine, and enabled the sandbox mode
to restrict what it can do (always a good practice in my opinion) and noticed
that Copilot couldn't run any `mix` commands.

I use ASDF (https://asdf-vm.com) to manage the different runtimes (Elixir, Ruby...etc)
and noticed that copilot can run `asdf` but it can't find any installed versions of any runtime.

{{<highlight bash>}}
(sandboxed)  No executable mix found for current version. Please select a different version
or install mix manually for the current version Failed with exit code 1.
{{< / highlight >}}

Copilot kept insisting that the issue is not with the sandbox but my own installation and since
`"allowDevToolAccess": true"` was enabled in Copilot's `settings.json` file, I
assumed the issue was either due to how copilot is launching the commands or some MacOS permission issue.

After digging through documentation and testing things with and without the sandbox, the solution was to
append the ASDF directory to the `readonlyPaths` list in Copilot settings.

{{< highlight json "linenos=inline">}}
{
    "readonlyPaths": [
        "/some/other/existing/path",
        "/Users/zaid/.asdf"
    ]
}
{{< / highlight >}}
