# macOS Automation

## AppleScript Integration

### Basic Usage
```bash
# Single-line AppleScript
osascript -e 'tell application "Finder" to activate'

# Multi-line AppleScript
osascript <<EOF
tell application "System Events"
    set frontApp to name of first application process whose frontmost is true
end tell
return frontApp
EOF

# With shell variables
APP_NAME="Safari"
osascript -e "tell application \"${APP_NAME}\" to activate"

# Capture output
FRONT_APP=$(osascript -e 'tell application "System Events" to get name of first application process whose frontmost is true')
```

### Common AppleScript Patterns
```bash
# Get clipboard content
CLIPBOARD=$(osascript -e 'get the clipboard')

# Set clipboard
osascript -e 'set the clipboard to "text to copy"'

# Open URL
osascript -e 'open location "https://example.com"'

# Get selected Finder items
osascript <<EOF
tell application "Finder"
    set selectedItems to selection
    set pathList to {}
    repeat with theItem in selectedItems
        set end of pathList to POSIX path of (theItem as alias)
    end repeat
    return pathList
end tell
EOF
```

### Dialog Boxes
```bash
# Alert dialog
osascript -e 'display alert "Title" message "Message"'

# Dialog with input
USER_INPUT=$(osascript <<EOF
set dialogResult to display dialog "Enter your name:" default answer ""
return text returned of dialogResult
EOF
)

# Choose from list
CHOICE=$(osascript <<EOF
choose from list {"Option 1", "Option 2", "Option 3"} \
    with title "Select Option" \
    with prompt "Choose one:"
EOF
)

# File chooser
FILE=$(osascript <<EOF
set chosenFile to choose file with prompt "Select a file:"
return POSIX path of chosenFile
EOF
)

# Folder chooser
FOLDER=$(osascript <<EOF
set chosenFolder to choose folder with prompt "Select folder:"
return POSIX path of chosenFolder
EOF
)
```

---

## Keychain Access

### Generic Passwords
```bash
# Store password
security add-generic-password \
    -a "username" \
    -s "service-name" \
    -w "password" \
    -T "" # Allow all apps (optional)

# Retrieve password
PASSWORD=$(security find-generic-password \
    -a "username" \
    -s "service-name" \
    -w)

# Delete password
security delete-generic-password \
    -a "username" \
    -s "service-name"

# Check if exists
if security find-generic-password \
    -a "username" \
    -s "service-name" \
    >/dev/null 2>&1; then
    echo "Password exists"
else
    echo "Password not found"
fi

# Update password (delete then add)
security delete-generic-password -a "username" -s "service-name" 2>/dev/null
security add-generic-password -a "username" -s "service-name" -w "new_password"
```

### Internet Passwords
```bash
# Store internet password
security add-internet-password \
    -a "username" \
    -s "example.com" \
    -w "password" \
    -r "htps" # Protocol: htps=https, http=http

# Retrieve
PASSWORD=$(security find-internet-password \
    -a "username" \
    -s "example.com" \
    -w)
```

### Helper Function
```bash
keychain_get() {
    local service="$1"
    local account="${2:-$USER}"
    security find-generic-password -a "$account" -s "$service" -w 2>/dev/null
}

keychain_set() {
    local service="$1"
    local password="$2"
    local account="${3:-$USER}"

    # Delete if exists, then add
    security delete-generic-password -a "$account" -s "$service" 2>/dev/null
    security add-generic-password -a "$account" -s "$service" -w "$password"
}

# Usage
API_KEY=$(keychain_get "my-api-key")
keychain_set "my-api-key" "secret123"
```

---

## Launch Agents

### Plist Template
```xml
<!-- ~/Library/LaunchAgents/com.example.script.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.example.script</string>

    <key>ProgramArguments</key>
    <array>
        <string>/path/to/script.sh</string>
        <string>--option</string>
    </array>

    <!-- Run at specific interval (seconds) -->
    <key>StartInterval</key>
    <integer>3600</integer>

    <!-- OR run at specific time -->
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>9</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>

    <!-- Run at load -->
    <key>RunAtLoad</key>
    <true/>

    <!-- Keep alive (restart if crashes) -->
    <key>KeepAlive</key>
    <true/>

    <!-- Working directory -->
    <key>WorkingDirectory</key>
    <string>/path/to/workdir</string>

    <!-- Environment variables -->
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/usr/local/bin:/usr/bin:/bin</string>
    </dict>

    <!-- Logging -->
    <key>StandardOutPath</key>
    <string>/tmp/script.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/script.error.log</string>
</dict>
</plist>
```

### Managing Launch Agents
```bash
# Load agent
launchctl load ~/Library/LaunchAgents/com.example.script.plist

# Unload agent
launchctl unload ~/Library/LaunchAgents/com.example.script.plist

# Start manually
launchctl start com.example.script

# Stop
launchctl stop com.example.script

# List running agents
launchctl list | grep com.example

# Check status
launchctl list com.example.script

# Bootstrap (modern alternative to load)
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.example.script.plist

# Bootout (modern alternative to unload)
launchctl bootout gui/$(id -u)/com.example.script
```

### Install Script
```bash
install_launch_agent() {
    local plist_name="$1"
    local plist_path="$HOME/Library/LaunchAgents/${plist_name}"

    # Copy plist
    cp "${plist_name}" "$plist_path"

    # Set permissions
    chmod 644 "$plist_path"

    # Load
    launchctl load "$plist_path"

    echo "Installed and loaded: $plist_name"
}
```

---

## Notification Center

### Display Notifications
```bash
# Basic notification
osascript -e 'display notification "Message" with title "Title"'

# With subtitle
osascript -e 'display notification "Message" with title "Title" subtitle "Subtitle"'

# With sound
osascript -e 'display notification "Complete" with title "Task" sound name "Glass"'

# Available sounds:
# Basso, Blow, Bottle, Frog, Funk, Glass, Hero, Morse, Ping, Pop, Purr, Sosumi, Submarine, Tink
```

### Notification Function
```bash
notify() {
    local title="$1"
    local message="$2"
    local subtitle="${3:-}"
    local sound="${4:-}"

    local script="display notification \"$message\" with title \"$title\""
    [[ -n "$subtitle" ]] && script="$script subtitle \"$subtitle\""
    [[ -n "$sound" ]] && script="$script sound name \"$sound\""

    osascript -e "$script"
}

# Usage
notify "Build Complete" "Project compiled successfully" "" "Glass"
```

### terminal-notifier (Alternative)
```bash
# Install
brew install terminal-notifier

# Usage
terminal-notifier -title "Title" -message "Message" -sound default

# With action
terminal-notifier -title "Click me" -message "Opens URL" -open "https://example.com"

# With icon
terminal-notifier -title "Title" -message "Message" -contentImage /path/to/icon.png
```

---

## Spotlight Search (mdfind)

### Basic Searches
```bash
# Find by filename
mdfind -name "filename.txt"

# Find by content
mdfind "search term"

# Limit to directory
mdfind -onlyin ~/Documents "keyword"

# Live updates
mdfind -live "search term"
```

### Metadata Queries
```bash
# Find by file type
mdfind "kMDItemContentType == 'public.image'"
mdfind "kMDItemContentType == 'com.adobe.pdf'"

# Find by modification date
mdfind "kMDItemFSContentChangeDate >= \$time.today"
mdfind "kMDItemFSContentChangeDate >= \$time.yesterday"

# Find by author
mdfind "kMDItemAuthors == '*Smith*'"

# Find by size (bytes)
mdfind "kMDItemFSSize > 1000000"

# Combined queries
mdfind "kMDItemContentType == 'public.image' && kMDItemFSSize > 500000"
```

### Common Metadata Attributes
```bash
# List attributes of a file
mdls /path/to/file

# Common attributes:
# kMDItemDisplayName - Display name
# kMDItemFSName - Filename
# kMDItemContentType - UTI type
# kMDItemFSSize - Size in bytes
# kMDItemFSCreationDate - Creation date
# kMDItemFSContentChangeDate - Modification date
# kMDItemAuthors - Author names
# kMDItemKind - Kind description
```

---

## Application Automation

### Open Commands
```bash
# Open application
open -a "Application Name"
open -a Safari

# Open URL
open "https://example.com"

# Open file in specific app
open -a "TextEdit" file.txt

# Open file with default app
open document.pdf

# Reveal in Finder
open -R /path/to/file

# Open in background
open -g "https://example.com"

# Wait for app to close
open -W -a "Preview" image.png
```

### Application Management
```bash
# Get bundle identifier
osascript -e 'id of application "Safari"'

# Check if running
pgrep -x "Safari" >/dev/null && echo "Running"

# Kill application
osascript -e 'tell application "Safari" to quit'
pkill -x "Safari"

# Activate (bring to front)
osascript -e 'tell application "Safari" to activate'

# Hide application
osascript -e 'tell application "System Events" to set visible of process "Safari" to false'
```

### System Events
```bash
# Get frontmost app
osascript -e 'tell application "System Events" to get name of first application process whose frontmost is true'

# List running apps
osascript -e 'tell application "System Events" to get name of every application process'

# Keystroke
osascript -e 'tell application "System Events" to keystroke "hello"'

# Key code
osascript -e 'tell application "System Events" to key code 36' # Return key

# With modifiers
osascript -e 'tell application "System Events" to keystroke "c" using command down'
```
