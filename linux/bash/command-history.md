# Command History

By default, the command history is saved to the `.bash_history` file in the user home directory. Two environment variables control the history size: `HISTSIZE` and `HISTFILESIZE`.

* `HISTSIZE` controls the number of commands stored in memory for the current session
* `HISTFILESIZE` configures how many commands are kept in the history file

These variables can be edited according to our needs and saved to the Bash configuration file (`.bashrc`)

## Accessing History

`history` command will display a numbered list of previously commands. Commands can be run from this list with `!<NUM>` where NUM is number displayed next to the desired command in the output of `history`.

* `!!` will re-run the last command run

Users can also scroll through command history via the command line with the UP and DOWN arrow buttons.

`Ctrl+R` typed at the command line will open a command search prompt that attempts to find the best match to your search input from the previously-run commands.
