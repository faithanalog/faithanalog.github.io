---
layout: post
title: Defining Custom Actions
---

This post covers defining custom actions. Custom actions allow you to create a single voice binding for an action, while providing multiple context-specific implementations of that action. For example, many applications have different key shortcuts on macOS than they do on Linux. Creating actions for your commands allows you to specify these shortcuts for each OS separately.

As a demonstration, let's define an action to trigger the file finder in Visual Studio Code. The default binding for this on Linux is `ctrl-p`, while on macOS it's `cmd-p`.

We're going to create the following file structure:

```
apps/
├── code
│   └── vscode.py
├── linux
│   └── vscode.talon
├── mac
│   └── vscode.talon
└── vscode.talon
```

Let's start with `apps/code/vscode.py`:

```python
from talon import Module

# create a new Module
mod = Module()

# @mod.action_class causes all methods defined on the annotated class to be
# defined as talon actions
@mod.action_class
class Actions:

    # Define an action. Actions will be available under the `user` namespace.
    # talonscript code can refer to this action as `user.vscode_go_to_file`.
    def vscode_go_to_file():
        # All actions must define documentation in triple quotes. This is
        # displayed in help contexts.
        """Show a dialog to search the active workspace for a file."""

        # Actions may also provide a default implementation to be used if no
        # other overriding implementation exists. We don't do that in this case.
```

This file defines a new Module, and creates an Action Class. All methods defined on an Action Class are available under the `user` namespace. With this file loaded, we can provide OS specific implementations for `user.vscode_go_to_file`.

`apps/linux/vscode.talon`:

```
os: linux
app: Code
-
action(user.vscode_go_to_file):
    key(ctrl-p)
```

`apps/mac/vscode.talon`:

```
os: mac
app: Code
-
action(user.vscode_go_to_file):
    key(cmd-p)
```

Finally we need to define a voice command in `apps/vscode.talon` to use it:

```
app: Code
-
go to file:
    user.vscode_go_to_file()
```

And that's it! Now this command can be used on both macOS and Linux. Providing a Windows implementation is as simple as adding an `apps/windows/vscode.talon` file to provide the relevant keybind.

Additional actions can be defined by repeating these steps:

- Declare the action in a python module.
- Provide context-specific implementations.
- Define a command to use the action.

The same process can be used to define an action with implementations that differ between applications. This is how [knausj_talon](https://github.com/knausj85/knausj_talon) implements `app.next_tab()` and `app.previous_tab()`. Things look a bit different here because there's no python file - `app.next_tab()` and `app.previous_tab()` are defined by talon itself. Regardless, the talonscript implementations work the same way:

- [apps/mac/app.talon](https://github.com/knausj85/knausj_talon/blob/16ff5c6c548f375bc2dcb87bbe7c14200e01b5f7/apps/mac/app.talon#L12) provides a generic implementation that works across most macOS applications.
- [apps/mac/terminal.talon](https://github.com/knausj85/knausj_talon/blob/16ff5c6c548f375bc2dcb87bbe7c14200e01b5f7/apps/mac/terminal.talon#L13) implements Terminal-specific keybinds.
- [tabs.talon](https://github.com/knausj85/knausj_talon/blob/16ff5c6c548f375bc2dcb87bbe7c14200e01b5f7/misc/tabs.talon#L15) defines the `last tab` and `next tab` commands which use the actions.

Now you should have everything you need to define your own talon actions! If you have any questions, feel free to message me [on the talon slack](https://talonvoice.slack.com); I'm Artemis Everfree.
