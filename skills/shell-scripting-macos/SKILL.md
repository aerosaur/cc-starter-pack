---
name: shell-scripting-macos
description: Expert guidance for shell scripting, bash/zsh automation, and macOS system integration. Use when writing shell scripts, automating workflows with bash/zsh, working with macOS-specific tools (AppleScript, Keychain, Spotlight, Automator), creating command-line utilities, or integrating system-level automation. Covers scripting patterns, error handling, and macOS APIs.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

# Shell Scripting & macOS Automation

Complete toolkit for shell scripting with bash/zsh and macOS system automation.

## When to Use This Skill

- Writing shell scripts for automation
- Creating command-line tools and utilities
- Automating macOS workflows
- Integrating with AppleScript or Automator
- Managing secure credentials with Keychain
- Building deployment or setup scripts

## Core Capabilities

| Category | Topics |
|----------|--------|
| **Shell Scripting** | Bash/zsh syntax, argument parsing, error handling, piping |
| **macOS Integration** | AppleScript, Keychain, Spotlight (mdfind), Launch Agents |
| **Text Processing** | sed, awk, grep, jq for JSON |
| **Automation** | Cron jobs, Launch Agents, Notification Center |

## Quick Start

### Basic Script Structure
```bash
#!/usr/bin/env bash
set -euo pipefail  # Exit on error, undefined vars, pipe failures

readonly SCRIPT_NAME="$(basename "$0")"
readonly SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"

main() {
    echo "Hello from ${SCRIPT_NAME}"
}

main "$@"
```

### Argument Parsing
```bash
while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help) show_help; exit 0 ;;
        -v|--verbose) VERBOSE=true; shift ;;
        -o|--output) OUTPUT_FILE="$2"; shift 2 ;;
        *) echo "Unknown option: $1"; exit 1 ;;
    esac
done
```

### Error Handling
```bash
cleanup() {
    local exit_code=$?
    echo "Cleaning up..."
    exit $exit_code
}
trap cleanup EXIT ERR

# Check command exists
command -v jq >/dev/null 2>&1 || { echo "Error: jq required"; exit 1; }
```

### Keychain Access
```bash
# Store
security add-generic-password -a "username" -s "service-name" -w "password"

# Retrieve
PASSWORD=$(security find-generic-password -a "username" -s "service-name" -w)
```

### AppleScript Integration
```bash
osascript -e 'tell application "Finder" to activate'
osascript -e 'display notification "Done" with title "Script"'
```

### Spotlight Search
```bash
mdfind -name "filename.txt"
mdfind -onlyin ~/Documents "keyword"
mdfind "kMDItemContentType == 'public.image'"
```

## Best Practices

**Script Safety:**
- Use `set -euo pipefail` for strict error handling
- Quote all variables: `"$var"` not `$var`
- Use `[[ ]]` for conditionals
- Validate input parameters

**Code Organization:**
- One function per task
- Use meaningful variable names
- Separate configuration from code

**Security:**
- Never eval user input
- Use `mktemp` for temporary files
- Use Keychain for credentials

## Detailed Documentation

- [Shell Patterns](references/shell-patterns.md) - Script structure, error handling, logging, testing
- [macOS Automation](references/macos-automation.md) - AppleScript, Keychain, Launch Agents, Notifications
- [Text Processing](references/text-processing.md) - sed, awk, grep, jq patterns
