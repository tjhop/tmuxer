# Tmuxer

A Tmux layout generator and command executor, similar to [tmuxinator](https://github.com/tmuxinator/tmuxinator), [tmux-xpanes](https://github.com/greymd/tmux-xpanes) or [i2ssh](https://github.com/mbruggmann/i2ssh).

## Overview

`tmuxer` is a small Bash script to automatically create multiple Tmux panes, and optionally run a command in each pane. Features include:

- The command to run can be global (ie, the same command is run in all panes and each pane provides a different target argument), or individual commands can be run per-pane.
- Input is synchronized across all panes by default.
- The ability to choose Tmux layout.
- The ability to create a new session or create a new window if attached to an existing Tmux session.

## Installation

#### Install with git

1. Install `tmux` (instructions will be specific to the distro you're running).
2. Clone this repo.
3. Symlink `tmuxer` script somewhere in your `$PATH`.

#### Install from AUR

`tmuxer` is now available in the [AUR for Archlinux](https://aur.archlinux.org/packages/tmuxer-git/)!

Basic AUR install:
```
git clone https://aur.archlinux.org/tmuxer-git.git aur/tmuxer-git
cd aur/tmuxer-git/
makepkg -si PKGBUILD
```

## Usage

`tmuxer` accepts several flags that you can use to configure it's behavior:

```
$ tmuxer -h
Usage:
    -h, --help              Show help
    -c, --command           Set command to be executed in each pane (default: `echo {}`). The command should contain `{}` where you
                            intend to substitute the pane's target argument. Multiple substituions can be made per pane.
                                For example: `dig {} +short`
    -d, --dump              Write out a tmuxer config file to stdout based on the provided command line arguments
    -f, --file [<file>]     Config file to read from. The following paths are checked for the specified file:
                                Absolute path, relative path to current directory, and `$XDG_CONFIG_HOME/tmuxer`
    -l, --layout [<layout>] Tmux layout to use
                                Valid options: `tiled`, `even-horizontal`, `even-vertical`, `main-horizontal`, `main-vertical`
                                Default: `tiled`
    -n, --new-session       Create new tmux session (default only if attached session not detected)
    -s, --ssh               "SSH Mode". Changes command to `ssh {}` (Same as: `--command 'ssh {}'`)
    -u, --unsync            Disable synchronization of panes (default: pane input is synchronized)
    -w, --window-name       What to name the tmux window created to hold panes
```

"Targets" for each Tmux pane can be provided as command line arguments, by piping/redirecting data to `tmuxer`, or by writing a config file. Checkout the examples below!

## Examples

#### Open new Tmux session and SSH to 10 hosts, without synchronizing input:

```shell
$ tmuxer --new-session --unsync --ssh user@host{01..10}

# same as:
$ tmuxer -n -u -s user@host{01..10}
```

#### Open new pane and check PTR records for reach A record returned by the `dig` command:

```shell
$ dig cloudfront.com +short | tmuxer --command 'dig -x {} +short' --layout 'even-vertical'

# same as:
$ dig cloudfront.com +short | tmuxer -c 'dig -x {} +short' -l 'even-vertical'
```

#### Open several panes and run different commands in each (_be careful to escape things!_):

```shell
$ tmuxer -c '{}' \
    "for i in {1..3}; do echo \"test #\$i\"; done" \
    "date" \
    "ls -lh \$HOME/github/tmuxer" \
    "cd \$HOME/github/tmuxer && git status"
```

#### Open up 4 blank panes in a new-session:

```shell
$ tmuxer -n -c '{}' '' '' '' ''
```

#### Use a config file to define targets:

```shell
$ cat<<EOF > /tmp/tmuxer-example
TMUXER_LAYOUT='even-vertical'
TMUXER_DISABLE_SYNC=1
TMUXER_NEW_SESSION=1
TMUXER_COMMAND='echo "{}": && dig {} +short'
TMUXER_PANES=(
    "google.com"
    "cloudflare.com"
    "packet.net"
    "linode.com"
)
EOF

$ tmuxer -f /tmp/tmuxer-example
```

#### Write a tmuxer config file based on command line options

Useful if you've created a `tmuxer` CLI command that you'd like to re-use in the future as a config file. Example: 

```shell
$ tmuxer --new-session --ssh user@host{01..04}
```

Using the `-d`/`--dump` flag, a `tmuxer` config file will be written to stdout (which can of course be redirected to a file of your choosing):

```shell
$ tmuxer --dump --new-session --ssh user@host{01..04}
TMUXER_LAYOUT="tiled"
TMUXER_DISABLE_SYNC=0
TMUXER_NEW_SESSION=1
TMUXER_COMMAND="ssh {}"
TMUXER_PANES=(
        user@host01
        user@host02
        user@host03
        user@host04
)
```

## Screencast Demos

Connecting to multiple hosts with `ssh`:
[![asciicast](https://asciinema.org/a/xP4HphFxGqzrtusVSxodVX9IV.svg)](https://asciinema.org/a/xP4HphFxGqzrtusVSxodVX9IV)

Using a config file with `tmuxer`:
[![asciicast](https://asciinema.org/a/IAqOAHDJAkqPqAMmccdKFEVdY.svg)](https://asciinema.org/a/IAqOAHDJAkqPqAMmccdKFEVdY)

Running arbitrary commands with data from stdin:
[![asciicast](https://asciinema.org/a/3tlGDk6BVhwMUoNFsWUzbBXiE.svg)](https://asciinema.org/a/3tlGDk6BVhwMUoNFsWUzbBXiE)

## Config File

Tmuxer will attempt to load configuration files in the following order:
1. By absolute path, if provided
2. By relative path to the working directory where `tmuxer` is invoked from
3. By relative path to `$XDG_CONFIG_HOME/tmuxer` as a fallback

#### Variables accepted and their usage:

| Variable Name | Variable Type | Description | Accepted Values | Default Value |
| --- | --- | --- | --- | --- |
| `TMUXER_COMMAND` | string | A command to be run in each Tmux pane opened. This will prefix any commands/arguments provided in that specific pane's command provided in the `PANES` array <br><br>**Be careful!** Commands are entered directly into the terminal and run. This can be dangerous. <br>*Note*: If providing shell variables, you will need to escape them. | _any_ | _empty string_ |
| `TMUXER_PANES` | array of strings | A list of command targets to be run. Each command/argument specified will open a new Tmux pane for the command to be run in. <br><br>**Be careful!** Commands are entered directly into the terminal and run. This can be dangerous. <br>*Note*: If providing shell variables, you will need to escape them.  | _any_ | _empty array_ |
| `TMUXER_LAYOUT` | string | The [Tmux layout](https://leanpub.com/the-tao-of-tmux/read#window-layouts) to apply to the panes | `even-vertical`, `even-horizontal`, `main-vertical`, `main-horizontal`, `tiled` | `tiled` |
| `TMUXER_DISABLE_SYNC` | integer | Whether or not to synchronize panes to broadcast input to all terminals | `1` (Disable sync)<br>`0`(Enable sync) | `0` |
| `TMUXER_NEW_SESSION` | string | Whether or not to create a new Tmux session to hold the panes.<br>This is the default if no attached session is detected. | `1` (Yes) <br>`0` (No) | `0`<br>(unless no attached session detected) |

## License
This project is licensed as [MIT](LICENSE).
