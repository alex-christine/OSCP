---
description: Focuses on nano and vi for CLI file editing
---

# Editing Files From Command Line

## `nano`

nano is one of the simplest text editors. It is invoked with the `nano <file>` command and options for cut, paste, format, save, etc. are stored across the bottom.

One of the primary benefits of nano is that it works in almost any situation because of its simplicity.

## `vi`

vi is an extremely powerful text editor, capable of blazing speed especially when it comes to automating repetitive tasks. However, it has a relatively steep learning curve and is nowhere near as simple to use as Nano.

1. Once the file is opened, enable _insert-text mode_ to begin typing. To do this, press the `i` key and start typing away
2. To disable _insert-text mode_ and go back to _command mode_, press the `esc` key
3. While in _command mode_, use:
   * `dd` to delete the current line
   * `yy` to copy the current line
   * `p` to paste the clipboard contents
   * `x` to delete the current character
   * `:w` to write the current file to disk and stay in vi
   * `:q!` to quit without writing the file to disk
   * `:wq` to save and quit

Because _vi_ seems so awkward to use, many users avoid it. However, from a penetration tester's point of view, _vi_ can save a great deal of time in the hands of an experienced user and _vi_ is installed on every POSIX-compliant system.
