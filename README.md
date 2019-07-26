# Tmuxer
A Tmux layout generator and command executor, similar to [tmuxinator](https://github.com/tmuxinator/tmuxinator) or [i2ssh](https://github.com/mbruggmann/i2ssh).

## Overview
`tmuxer` is a small Bash script to create Tmux sessions of a predefined layout, and optionally seed each pane with a command to run. Commands can be specified on a per-pane basis, or a global command can be provided which each pane containing a different target argument. Optionally, it allows specifying a Tmux layout for the panes, as well as whether or not to synchronize input across all panes (ie, broadcast input to all Tmux panes at once).

`tmuxer` config files just contains Bash variables that get sourced by the script. Config files are currently expected to live under `$HOME/.tmuxer/` and can be named whatever you want. Example configs can be found in the `examples/` folder in this repo. There are also `asciinema` cast examples under [Examples](#Examples).

## Installation
1. Install `tmux` (instructions will be specific to the distro you're running).
2. Clone this repo.
3. Symlink `tmuxer` script somewhere in your `$PATH`.

## Usage
Create a config file under `$HOME/.tmuxer/`. Variables accepted and their usage:

| Variable Name | Variable Type | Description | Accepted Values | Default Value |
| --- | --- | --- | --- | --- |
| `TMUX_LAYOUT` | string | The [Tmux layout](https://leanpub.com/the-tao-of-tmux/read#window-layouts) to apply to the panes | `even-vertical`, `even-horizontal`, `main-vertical`, `main-horizontal`, `tiled` | `tiled` |
| `TMUX_SYNCHRONIZE` | string | Whether or not to synchronize panes to broadcast input to all terminals | `on`, `off` | `on` |
| `PANES` | array of strings | A list of command targets to be run. Each command/argument specified will open a new tmux pane for the command to be run in. <br><br>**Be careful!** Commands are entered directly into the terminal and run. This can be dangerous. <br>*Note*: If providing shell variables, you will need to escape them.  | _any_ | _empty array_ |
| `GLOBAL_COMMAND` | string | A command to be run in each Tmux pane opened. This will prefix any commands/arguments provided in that specific pane's command provided in the `PANES` array <br><br>**Be careful!** Commands are entered directly into the terminal and run. This can be dangerous. <br>*Note*: If providing shell variables, you will need to escape them. | _any_ | _empty string_ |

## Examples
Example with a global command (`dig`):
[![asciicast](https://asciinema.org/a/zpvTnqtBpyiFYauARunD0VZCj.png)](https://asciinema.org/a/zpvTnqtBpyiFYauARunD0VZCj)

Example with no global command and individual pane commands:
[![asciicast](https://asciinema.org/a/FM3GkAduszLvvDyTVAFOhqoq7.png)](https://asciinema.org/a/FM3GkAduszLvvDyTVAFOhqoq7)

## License
This project is licensed as [MIT](LICENSE).
