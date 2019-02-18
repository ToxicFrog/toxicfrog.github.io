---
layout: post
title: Adding subcommands to arbitrary programs
---

Git -- like many other Linux programs -- has a bunch of subcommands. You never run `git` on its own; you always `git <verb>`, e.g. `git commit` or `git log`. Under the hood these are handled by hyphenated commands: `git-commit`, `git-log`, etc. This is fairly standard; it's a UI paradigm used by other version control systems, backup software, the ZFS and mdraid tools, taskwarrior, beets, and so forth.

What git has that the others don't is the ability to easily add new subcommands. If you have something in `$PATH` named `git-foo`, you can then run `git foo ...`. This isn't something I've seen in other programs, including ones that generally have much better thought out UIs than git.

So, I wrote a function for my `.zshrc` that lets me add this capability to _any_ command. It's more general than the git version; in addition to binaries in `$PATH` it also supports shell functions and aliases.

Doing this for any one command is easy; you just do something like this:

    function hg-subcommand-wrapper {
      local subcommand="$1"; shift
      if silently type "hg-$subcommand"; then
        "hg-$subcommand" "$@"
      else
        command hg "$subcommand" "$@"
      fi
    }

(`silently`, by the way, is just `function silently { &>/dev/null "$@"; }`). But then you have to write one of these for each command you want to wrap. It gets tedious pretty fast.

So, once I started wanting to do this for more than one command, I wrote a generic version, that looks like this:

    function subcommand-wrapper {
      local command="$1"
      local subcommand="$2"
      shift 2
      if silently type "$command-$subcommand"; then
        "$command-$subcommand" "$@"
      else
        command "$command" "$subcommand" "$@"
      fi
    }

This one expects to get the original command from its caller. Then we just need something to make sure it gets invoked appropriately:

    function enable-subcommands-for {
      while [[ $1 ]]; do
        alias "$1"="subcommand-wrapper $1"
        shift
      done
    }

And then:

    enable-subcommands-for hg beet borg zfs zpool ipfs

Now I can define new subcommands for any of those by dropping scripts in `~/bin`, or just defining shell functions or aliases.
