# tmux-init

`tmux-init` is a bash shell script to (re-)initialize tmux sessions.

Instead of writing a config file, you organize the scripts you want tmux to run in a directory structure. Calling `tmux-init` on that directory creates the appropriate sessions, windows and panes; and runs the scripts in them.

Here's an example:
```
~/
├── .config
│   └── tmux-init  # default root directory for tmux-init, change with -d
│       ├── session1          # session name
│       │   ├── window1.sh    # This creates a window named "window1" and runs window1.sh in it
│       │   ├── window2       # This creates a window named "window2", and splits it into two panes.
│       │   │   ├── pane1.sh  # This runs in the first pane.
│       │   │   └── pane2.sh  # This runs in the second pane.
│       │   └── window3 -> ../../../shared_scripts/window3_executable  # Symlinks work.
│       └── session2
│           ├── window1
│           │   ├── pane1.py # Any executable file works.
│           │   └── pane2.sh
│           └── window2.pl
└── shared_scripts
    └── window3_executable (executable script)
```


## What is this good for?

Some examples and recipes:

### Running long-lived processes

I run a bunch of daemons that require user session in tmux, which is maybe not advisable, but works quite well, specially if you need to occasionally check logs for these jobs or restart them. tmux-init makes it easy to start them all at once. 

```
.
└── background
    ├── emacs-daemon.sh
    ├── sketchybar.sh
    ├── sync-email.sh
    └── sync-notes.sh
```

This works for multiple machines as well. Here's an example setup:

```bash
~/.config/tmux-init
│
├── laptop.hostname
│   └── background
│       ├── sync-email.sh
│       └── sync-notes.sh
├── server.hostname
│   └── background
│       └── backup.sh
│
└── all
    ├── background
    │   └── emacs-daemon.sh
    └── monitoring
        └── splitter
            ├── top -> /usr/bin/top
            └── monitor_sensors.sh

$ tmux-init -d ~/.config/tmux-init/all
$ tmux-init -d ~/.config/tmux-init/"$(hostname)"
```

### Setting up a development environment

You can have tmux-init set up dev environments. The setup scripts can live in the same repository as your code to keep things simple.

```
~/
└── src
    └── my-project
        └── tmux-init
            ├── backend
            │   ├── start_api_server.sh  # Script to start the Python backend server.
            │   └── background_jobs
            │       ├── job1.py          # Python script for the first background job.
            │       └── job2.py          # Python script for the second background job.
            └── frontend
                ├── launch_dev_server.sh # Script to start the frontend development server.
                └── tasks
                    ├── build_ui.sh      # Build the UI components.
                    └── test_ui.sh       # Execute UI-related tests.
                    
$ tmux-init -d src/my-project/tmux-init
```

## What is this not good for?

Fine-grained control over splits, nested splits, and other advanced window/pane management. Its goal is to have a simple, config-less way to initialize tmux sessions.

## Installation

You can clone the repository and link to the script from a directory in your `PATH`:

```bash
git clone https://github.com/sam33r/tmux-init.git
ln -s "$(pwd)/tmux-init/tmux-init" /usr/local/bin/tmux-init # Might need sudo
```

## Usage

Optional arguments:

- `-d | --dir` `<directory>`: Set the root directory to traverse (default: `$HOME/.config/tmux-init`).
- `-l | --layout` `<layout>`: Set the layout for window panes (default: `even-horizontal`).
- `-c | --conflicts` `<strategy>`: Define the conflict resolution strategy for windows (`abort`, `replace`, `skip`, `append`).
- `--dry-run`: Print the sessions, windows and panes that will be created, but don't actually create them.

### Conflict Resolution

When existing windows with conflicting names are encountered, you can specify how to handle the conflict with the `--conflict` option:
- **abort**: Stop execution.
- **replace**: Replace the conflicting window with the new one (killing the existing window).
- **skip**: Skip conflicting windows.
- **append**: Create a new window and append a numerical suffix.
