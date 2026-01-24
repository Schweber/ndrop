# ndrop

This Bash script emulates the main features of [tdrop](https://github.com/noctuid/tdrop) in [niri](https://github.com/YaLTeR/niri):

- if the specified program is not running: launch it and bring it to the foreground.
- if the specified program is already running on another workspace: bring it to the current workspace and focus it.
- if the specified program is already on the current workspace: move it to workspace 'niri', thereby hiding it until called up again by ndrop.

> \[!NOTE]
> Niri doesn't implement scratchpads yet so ndrop is inferior to hdrop until they are added to niri.

> \[!NOTE]
> There is a modern version of this script available for **nushell** [here](https://github.com/Schweber/nudrop).

#### Usage:

> ndrop [OPTIONS] [COMMAND]

#### Arguments:

> [COMMAND]  
> The usual command you would run to start the desired program

#### Options:

> -c, --class  
> Set classname of the program to be run. Use this if the classname is different from the name of the [COMMAND] and ndrop does not have a hardcoded replacement.
>
> -F, --focus  
> Changes the default behaviour: focus the specified program's window and switch to its present workspace if necessary. Do not hide it, if it's already on the current workspace."
>
> -H, --help  
> Print help message
>
> -i, --insensitive  
> Case insensitive partial matching of class names. Can work as a stopgap if a running program is not recognized and a new instance is launched instead. Note: incorrect matches may occur, adding a special handling of the program to ndrop (hardcoded or via `-c, --class`) is preferable.
>
> -o, --online  
> Delay initial launch for up to 20 seconds until internet connectivity is established.
>
> -v, --verbose  
> Show notifications regarding the matching process. Try this to figure out why running programs are not matched.
>
> -V, --version  
> Print version

#### Multiple instances:

Multiple instances of the same program can be run concurrently, if different class names are assigned to each instance. Presently, there is support for the following flags in the [COMMAND] string:

> `-a` | `--app-id` ([foot](https://codeberg.org/dnkl/foot/) terminal emulator)  
> `--class` (all other programs)

#### Example bindings in niri config:

```kdl
workspace "ndrop"
spawn-at-startup "niri" "msg" "action" "focus-workspace-down"

binds {
    Mod+e { spawn "ndrop" "kitty" "--class" "kitty_1"; }
    Mod+Alt+e { spawn "ndrop" "kitty" "--class" "kitty_2"; }
    Mod+a { spawn "ndrop" "kitty" "--class" "kitty_hx" "hx"; }
    Mod+Alt+a { spawn "ndrop" "kitty" "--class" "kitty_code"; }
}
```

> \[!NOTE] 
> Defining a class name is only necessary when running several instances of the same program.

## Troubleshooting

### Programs are not moved away from the present workspace

Please see the example bindings. There has to be a workspace named `ndrop` for ndrop to work and `ndrop` may not be your present workspace.

Niri doesn't implement scratchpads yet so the behaviour of `ndrop` is inferior to `hdrop`.

### Further instances of programs are started instead of hiding/unhiding a running instance

If ndrop can't match an already running program and starts a new instance instead, then its class name is most likely different from its command name. For example, the class name of `telegram-desktop` is `org.telegram.desktop` and the class name of `logseq` is `Logseq`.

Run `ndrop -v [COMMAND]` _in the terminal_ to see maximum output for troubleshooting and find out the actual class name. Then use `ndrop -c CLASSNAME` to make it work. `ndrop -i [COMMAND]` might be sufficient, as long as a case insensitive (partial) match is sufficient.

Please report instances of programs with differing class names, so that they can be added to `ndrop`.

## Installation

### Repositories

[![Packaging status](https://repology.org/badge/vertical-allrepos/ndrop.svg)](https://repology.org/project/ndrop/versions)

### Manual

Make sure that `bash` and `jq` are in your PATH.

Download the script, make it executable and add it to your PATH.

Note: `ndrop` will only work in a `niri` session.  

### Makefile

Use the provided Makefile.

## See also

[hdrop](https://github.com/schweber/hdrop) is the equivalent for [hyprland](https://github.com/hyprwm/hyprland).

[nudrop](https://github.com/Schweber/nudrop) is the modern version of this script available for **nushell**.
