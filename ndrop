#!/usr/bin/env bash

# set -euo pipefail

print_help() {
  echo ""
  echo "Usage: ndrop [OPTIONS] [COMMAND]"
  echo ""
  echo "Command:"
  echo "          The usual command you would run to start the desired program"
  echo ""
  echo "Options:"
  echo "  -c, --class"
  echo "          Set classname of the program to be run. Use this if the"
  echo "          classname is different from the name of the [COMMAND]"
  echo "          and ndrop has no hardcoded replacement."
  echo ""
  echo "  -F, --focus"
  echo "          Changes the default behaviour: focus the specified program's"
  echo "          window and switch to its present workspace if necessary."
  echo "          Do not hide it, if it's already on the current workspace."
  echo ""
  echo "  -H, --help"
  echo "          Print this help message"
  echo ""
  echo "  -i, --insensitive"
  echo "          Case insensitive partial matching of class names."
  echo "          You can try this if a running program is not recognized and"
  echo "          a new instance is launched instead."
  echo ""
  echo "  -o, --online"
  echo "          Delay initial launch for up to 20 seconds"
  echo "          until internet connectivity is established."
  echo ""
  echo "  -v, --verbose"
  echo "          Show notifications regarding the matching process."
  echo "          Use this to figure out why running programs are not matched."
  echo ""
  echo "  -V, --version"
  echo "          Print version"
  echo ""
  echo "Multiple instances:"
  echo ""
  echo "Multiple instances of the same program can be run concurrently, if"
  echo "different class names are assigned to each instance. Presently there is"
  echo "support for the following flags in the [COMMAND] string:"
  echo ""
  echo " -a         ('foot' terminal emulator)"
  echo " --class    (all other programs)"
  echo ""
  echo "See man page for more information"
}

print_version() {
  echo "ndrop version: 0.1.0-dev"
}

notify() {
  notify-send "$@"
  echo "$@"
}

notify_low() {
  notify-send -u low "$@"
  echo "$@"
}

partial_match() {
  CLASS=$(niri msg --json windows | jq -r ".[] | select((.app_id |match(\"$1\";\"i\"))) | .app_id" | head -1)
}

wait_online() {
  wait_online=0
  while [[ $wait_online -lt 100 ]]; do
    ping -qc 1 -W 0.2 github.com && break
    sleep 0.1
    $wait_online++
  done
}

ndrop_flags() {
  while true; do
    case "$1" in
    -c | --class)
      shift
      CLASS_OVERRIDE="$1"
      ;;
    -F | --focus)
      FOCUS=true
      ;;
    -H | --help)
      print_help
      exit
      ;;
    -i | --insensitive)
      INSENSITIVE=true
      ;;
    -o | --online)
      ONLINE=true
      ;;
    -v | --verbose)
      VERBOSE=true
      ;;
    -V | --version)
      print_version
      exit
      ;;
    *) break ;;
    esac
    shift
  done
}

FOCUS=false
INSENSITIVE=false
ONLINE=false
VERBOSE=false
NDROP_FLAGS=()

while true; do
  case "$1" in
  "")
    notify "ndrop: Missing Argument" "Run 'ndrop -h' for more information"
    print_help
    exit 1
    ;;
  -c | --class)
    NDROP_FLAGS+=("$1" "$2")
    shift
    ;;
  -*)
    NDROP_FLAGS+=("$1")
    ;;
  *) break ;;
  esac
  shift
done

# shellcheck disable=SC2128 # erroneous warning
if [[ -n $NDROP_FLAGS ]]; then
  # shellcheck disable=SC2046 # avoids having to run 'eval set -- "$NDROP_FLAGS"' to remove leading whitespace
  ndrop_flags $(getopt -u --options c:FHiovV --longoptions class:,focus,help,insensitive,online,verbose,version, -n ndrop -- "${NDROP_FLAGS[@]}")
fi

CLASS="$1"
COMMANDLINE="${*:1}"
ACTIVE_WORKSPACE=$(niri msg --json workspaces | jq -r ".[] | select(.is_focused==true) | .id") || notify "ndrop: Error executing dependencies 'niri msg' or 'jq'" "Check terminal output of 'ndrop $COMMANDLINE'"

case "$1" in
epiphany)
  CLASS="org.gnome.Epiphany"
  ;;
brave)
  CLASS="brave-browser"
  ;;
foot)
  OPT=$(getopt --options a: --longoptions app-id: -n ndrop -- "$@")
  ;;
godot4)
  partial_match "org.godotengine."
  ;;
logseq)
  CLASS="Logseq"
  ;;
telegram-desktop)
  CLASS="org.telegram.desktop"
  ;;
tor-browser)
  CLASS="Tor Browser"
  ;;
*)
  OPT=$(getopt --options "" --longoptions class: -n ndrop -- "$@")
  ;;
esac

if $VERBOSE && [[ $1 != "$CLASS" ]]; then notify_low "ndrop: Using '$CLASS' as hardcoded classname of '$1' for matching"; fi

if [[ -n $OPT ]]; then
  eval set -- "$OPT" # remove leading whitespace
  case "$1" in
  -a | --class)
    CLASS="$2"
    if $VERBOSE; then notify_low "ndrop: Extracted '$CLASS' from [COMMAND] for matching"; fi
    ;;
  esac
fi

if $INSENSITIVE && [[ -n $(niri msg --json windows | jq -r ".[] | select((.app_id |test(\"$CLASS\";\"i\")))") ]]; then
  if $VERBOSE; then notify_low "ndrop: --insensitive -> Insensitive (partial) match of class '$CLASS' successful"; fi
  CLASS=$(niri msg --json windows | jq -r ".[] | select((.app_id |match(\"$CLASS\";\"i\"))) | .app_id" | head -1) || notify "ndrop: Error assigning case insensitive (partial) match to CLASS"
  if $VERBOSE; then notify_low "ndrop: --insensitive -> Using class '$CLASS' after insensitive (partial) matching"; fi
fi

if [[ -n $CLASS_OVERRIDE ]]; then
  if $VERBOSE; then notify_low "ndrop: --class -> Using given class '$CLASS_OVERRIDE' instead of '$CLASS' for matching"; fi
  CLASS="$CLASS_OVERRIDE"
fi

WINDOW_ID=$(niri msg --json windows | jq -r "first(.[] | select(.app_id==\"$CLASS\") | .id)")

### OVERRIDE ###
# FOCUS=true

if [[ -n $(niri msg --json windows | jq -r ".[] | select(.app_id==\"$CLASS\" and .workspace_id!=\"$ACTIVE_WORKSPACE\")") ]]; then
  if [[ $FOCUS == false ]]; then
    # shellcheck disable=SC2140 # erroneous warning
    niri msg action move-window-to-workspace --window-id "$WINDOW_ID" "$ACTIVE_WORKSPACE" || notify "ndrop: Error moving '$COMMANDLINE' to current workspace"
    if $VERBOSE; then notify_low "ndrop: Matched class '$CLASS' on another workspace and moved it to current workspace"; fi
  fi
  niri msg action focus-window --id "$WINDOW_ID" || notify "ndrop: Error focusing '$COMMANDLINE' on current workspace"
elif [[ -n $(niri msg --json windows | jq -r ".[] | select(.app_id==\"$CLASS\" and .workspace_id==\"$ACTIVE_WORKSPACE\")") ]]; then
  if [[ $FOCUS == false ]]; then
    # shellcheck disable=SC2086 # integers won't be split
    niri msg action move-window-to-workspace --window-id "$WINDOW_ID" "ndrop" || notify "ndrop: Error moving '$COMMANDLINE' to workspace 'ndrop'"
    niri msg action focus-workspace "$ACTIVE_WORKSPACE"
    if $VERBOSE; then notify_low "ndrop: Matched class '$CLASS' on current workspace and moved it to workspace 'ndrop'"; fi
    # else
  elif [[ -n $(niri msg --json windows | jq -r ".[] | select(.app_id==\"$CLASS\" and .workspace_id!=\"ndrop\")") ]]; then
    niri msg action focus-window --id "$WINDOW_ID" || notify "ndrop: Error focusing '$COMMANDLINE' on current workspace"
  fi
else
  if $ONLINE; then wait_online; fi
  # 'foot' always throws an error when its window is closed. Thus we disable the notification.
  if [[ $COMMAND == "foot" ]]; then
    # shellcheck disable=SC2086 # when quoting COMMANDLINE the execution of the command fails
    $COMMANDLINE
  else
    # shellcheck disable=SC2086 # when quoting COMMANDLINE the execution of the command fails
    $COMMANDLINE || notify "ndrop: Error executing given command" "$COMMANDLINE"
  fi
  if $VERBOSE; then notify_low "ndrop: No running program matches class '$CLASS'." "Currently active classes are '$(niri msg --json windows | jq -r '.[] | .app_id' | sort | tr '\n' ' ')'. Executed '$COMMANDLINE' in case it was not running already."; fi
fi
