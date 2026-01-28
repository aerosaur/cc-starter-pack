# Shell Scripting Patterns

## Script Structure

### Complete Template
```bash
#!/usr/bin/env bash
# Script: script_name.sh
# Description: Brief description
# Usage: script_name.sh [options] <arguments>

set -euo pipefail

# Constants
readonly SCRIPT_NAME="$(basename "$0")"
readonly SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
readonly LOG_FILE="/tmp/${SCRIPT_NAME}.log"

# Defaults
VERBOSE=false
DRY_RUN=false

# Colors (optional)
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Logging
log() { echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"; }
log_error() { echo -e "${RED}[ERROR]${NC} $*" >&2; }
log_warn() { echo -e "${YELLOW}[WARN]${NC} $*"; }
log_success() { echo -e "${GREEN}[OK]${NC} $*"; }
debug() { [[ "$VERBOSE" == true ]] && echo "[DEBUG] $*"; }

# Help
show_help() {
    cat <<EOF
Usage: ${SCRIPT_NAME} [options] <argument>

Options:
    -h, --help      Show this help message
    -v, --verbose   Enable verbose output
    -n, --dry-run   Show what would be done

Arguments:
    argument        Required argument description

Examples:
    ${SCRIPT_NAME} example
    ${SCRIPT_NAME} -v --dry-run example
EOF
}

# Cleanup
cleanup() {
    local exit_code=$?
    debug "Cleaning up (exit code: $exit_code)"
    # Add cleanup tasks here
    exit $exit_code
}
trap cleanup EXIT ERR INT TERM

# Argument parsing
parse_args() {
    while [[ $# -gt 0 ]]; do
        case $1 in
            -h|--help) show_help; exit 0 ;;
            -v|--verbose) VERBOSE=true; shift ;;
            -n|--dry-run) DRY_RUN=true; shift ;;
            -*) log_error "Unknown option: $1"; exit 1 ;;
            *) ARGS+=("$1"); shift ;;
        esac
    done
}

# Validation
validate() {
    if [[ ${#ARGS[@]} -eq 0 ]]; then
        log_error "Missing required argument"
        show_help
        exit 1
    fi
}

# Main logic
main() {
    parse_args "$@"
    validate

    log "Starting ${SCRIPT_NAME}"
    # Main logic here
    log_success "Completed successfully"
}

main "$@"
```

---

## Argument Parsing

### Positional Arguments
```bash
# Simple positional
INPUT="$1"
OUTPUT="${2:-default}"  # With default

# With validation
if [[ $# -lt 1 ]]; then
    echo "Usage: $0 <input>"
    exit 1
fi
```

### Named Arguments with getopts
```bash
while getopts "hvf:o:" opt; do
    case $opt in
        h) show_help; exit 0 ;;
        v) VERBOSE=true ;;
        f) INPUT_FILE="$OPTARG" ;;
        o) OUTPUT_FILE="$OPTARG" ;;
        ?) exit 1 ;;
    esac
done
shift $((OPTIND-1))
```

### Long Options
```bash
while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            show_help
            exit 0
            ;;
        -v|--verbose)
            VERBOSE=true
            shift
            ;;
        -f|--file)
            INPUT_FILE="$2"
            shift 2
            ;;
        --file=*)
            INPUT_FILE="${1#*=}"
            shift
            ;;
        --)
            shift
            break
            ;;
        -*)
            echo "Unknown option: $1"
            exit 1
            ;;
        *)
            ARGS+=("$1")
            shift
            ;;
    esac
done
```

---

## Error Handling

### Exit Codes
```bash
# Standard exit codes
readonly EXIT_SUCCESS=0
readonly EXIT_ERROR=1
readonly EXIT_USAGE=2
readonly EXIT_NOT_FOUND=3

exit_with_error() {
    local message="$1"
    local code="${2:-$EXIT_ERROR}"
    echo "Error: $message" >&2
    exit "$code"
}
```

### Try-Catch Pattern
```bash
# Capture error output
if ! result=$(command 2>&1); then
    echo "Command failed: $result"
    exit 1
fi

# With timeout
if ! timeout 30 command; then
    echo "Command timed out"
    exit 1
fi
```

### Trap Handlers
```bash
# Multiple traps
cleanup() {
    echo "Cleaning up temp files..."
    rm -f /tmp/script_*
}

on_error() {
    echo "Error on line $1" >&2
}

on_interrupt() {
    echo "Script interrupted"
    exit 130
}

trap cleanup EXIT
trap 'on_error $LINENO' ERR
trap on_interrupt INT TERM
```

### Defensive Checks
```bash
# Check command exists
require_command() {
    command -v "$1" >/dev/null 2>&1 || {
        echo "Required command not found: $1"
        exit 1
    }
}
require_command jq
require_command curl

# Check file exists
require_file() {
    [[ -f "$1" ]] || { echo "File not found: $1"; exit 1; }
}

# Check directory exists
require_dir() {
    [[ -d "$1" ]] || { echo "Directory not found: $1"; exit 1; }
}

# Check not empty
require_var() {
    [[ -n "${!1:-}" ]] || { echo "Variable not set: $1"; exit 1; }
}
```

---

## Logging

### Simple Logger
```bash
readonly LOG_FILE="${LOG_FILE:-/tmp/script.log}"

log() {
    local level="$1"
    shift
    local message="$*"
    local timestamp=$(date +'%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] [$level] $message" | tee -a "$LOG_FILE"
}

log_info() { log "INFO" "$@"; }
log_warn() { log "WARN" "$@"; }
log_error() { log "ERROR" "$@" >&2; }
log_debug() { [[ "$DEBUG" == true ]] && log "DEBUG" "$@"; }
```

### Colored Output
```bash
# Color codes
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

print_color() {
    local color="$1"
    shift
    echo -e "${color}$*${NC}"
}

# Usage
print_color "$RED" "Error message"
print_color "$GREEN" "Success!"
```

### Progress Indicator
```bash
# Spinner
spinner() {
    local pid=$1
    local chars="/-\|"
    while kill -0 "$pid" 2>/dev/null; do
        for (( i=0; i<${#chars}; i++ )); do
            printf "\r%s" "${chars:$i:1}"
            sleep 0.1
        done
    done
    printf "\r"
}

# Usage: long_command & spinner $!

# Progress bar
progress_bar() {
    local current=$1
    local total=$2
    local width=50
    local percent=$((current * 100 / total))
    local filled=$((current * width / total))
    local empty=$((width - filled))
    printf "\r[%s%s] %d%%" "$(printf '#%.0s' $(seq 1 $filled))" "$(printf ' %.0s' $(seq 1 $empty))" "$percent"
}
```

---

## Testing Shell Scripts

### Basic Test Framework
```bash
#!/usr/bin/env bash
# test_script.sh

TESTS_PASSED=0
TESTS_FAILED=0

assert_equals() {
    local expected="$1"
    local actual="$2"
    local message="${3:-}"

    if [[ "$expected" == "$actual" ]]; then
        echo "✓ PASS: $message"
        ((TESTS_PASSED++))
    else
        echo "✗ FAIL: $message"
        echo "  Expected: $expected"
        echo "  Actual:   $actual"
        ((TESTS_FAILED++))
    fi
}

assert_true() {
    local condition="$1"
    local message="${2:-}"

    if eval "$condition"; then
        echo "✓ PASS: $message"
        ((TESTS_PASSED++))
    else
        echo "✗ FAIL: $message"
        ((TESTS_FAILED++))
    fi
}

# Test functions
test_my_function() {
    local result
    result=$(my_function "input")
    assert_equals "expected" "$result" "my_function returns expected value"
}

# Run tests
run_tests() {
    test_my_function
    # Add more tests

    echo ""
    echo "Tests passed: $TESTS_PASSED"
    echo "Tests failed: $TESTS_FAILED"
    [[ $TESTS_FAILED -eq 0 ]]
}

run_tests
```

### Mocking
```bash
# Mock a command
mock_curl() {
    curl() {
        echo '{"status": "ok"}'
    }
}

# Restore original
restore_curl() {
    unset -f curl
}

# Use in tests
test_api_call() {
    mock_curl
    result=$(api_call)
    assert_equals "ok" "$result"
    restore_curl
}
```

### Dry Run Mode
```bash
DRY_RUN="${DRY_RUN:-false}"

run() {
    if [[ "$DRY_RUN" == true ]]; then
        echo "[DRY RUN] Would execute: $*"
    else
        "$@"
    fi
}

# Usage
run rm -rf /tmp/files
run git push origin main
```

---

## Configuration Management

### Config Files
```bash
# Load config
load_config() {
    local config_file="${1:-$HOME/.config/script.conf}"

    if [[ -f "$config_file" ]]; then
        # shellcheck source=/dev/null
        source "$config_file"
    fi
}

# Config file format (script.conf):
# VAR1="value1"
# VAR2="value2"
```

### Environment Variables
```bash
# With defaults
DB_HOST="${DB_HOST:-localhost}"
DB_PORT="${DB_PORT:-5432}"

# Required
: "${API_KEY:?API_KEY is required}"

# Export for subprocesses
export PATH="$HOME/bin:$PATH"
```

### Dotenv Loading
```bash
load_dotenv() {
    local env_file="${1:-.env}"

    if [[ -f "$env_file" ]]; then
        while IFS= read -r line || [[ -n "$line" ]]; do
            # Skip comments and empty lines
            [[ "$line" =~ ^#.*$ || -z "$line" ]] && continue
            # Export variable
            export "$line"
        done < "$env_file"
    fi
}
```
