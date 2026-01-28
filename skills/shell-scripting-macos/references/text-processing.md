# Text Processing

## sed Patterns

### Basic Operations
```bash
# Replace first occurrence per line
sed 's/old/new/' file.txt

# Replace all occurrences
sed 's/old/new/g' file.txt

# Case-insensitive replace
sed 's/old/new/gi' file.txt

# In-place editing (macOS)
sed -i '' 's/old/new/g' file.txt

# In-place with backup
sed -i '.bak' 's/old/new/g' file.txt

# Delete lines
sed '/pattern/d' file.txt

# Print specific lines
sed -n '5p' file.txt       # Line 5
sed -n '5,10p' file.txt    # Lines 5-10

# Delete empty lines
sed '/^$/d' file.txt
```

### Advanced sed
```bash
# Multiple commands
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file.txt

# Address ranges
sed '1,5s/old/new/g' file.txt      # Lines 1-5
sed '/start/,/end/s/old/new/g'     # Between patterns

# Capture groups
sed 's/\(.*\)@\(.*\)/User: \1, Domain: \2/' emails.txt

# Append after line
sed '/pattern/a\new line' file.txt

# Insert before line
sed '/pattern/i\new line' file.txt

# Replace line
sed '/pattern/c\replacement line' file.txt

# Transform to uppercase
sed 's/.*/\U&/' file.txt  # GNU sed
tr '[:lower:]' '[:upper:]' < file.txt  # macOS
```

---

## awk Programming

### Basic Usage
```bash
# Print specific columns
awk '{print $1, $3}' file.txt

# Custom field separator
awk -F',' '{print $2}' file.csv
awk -F':' '{print $1}' /etc/passwd

# Pattern matching
awk '/pattern/ {print}' file.txt

# Conditional printing
awk '$3 > 100 {print $1, $3}' file.txt

# Line numbers
awk '{print NR, $0}' file.txt
```

### Variables and Operations
```bash
# Built-in variables
# NR - current line number
# NF - number of fields
# $0 - entire line
# $1, $2... - individual fields
# FS - field separator
# OFS - output field separator

# Sum column
awk '{sum += $1} END {print sum}' numbers.txt

# Average
awk '{sum += $1; count++} END {print sum/count}' numbers.txt

# Count lines
awk 'END {print NR}' file.txt

# String concatenation
awk '{name = $1 " " $2; print name}' file.txt
```

### Advanced awk
```bash
# BEGIN/END blocks
awk 'BEGIN {print "Header"} {print} END {print "Footer"}' file.txt

# Custom output separator
awk -F',' 'BEGIN {OFS="\t"} {print $1, $2}' file.csv

# Associative arrays
awk '{count[$1]++} END {for (word in count) print word, count[word]}' file.txt

# Multiple patterns
awk '/start/,/end/ {print}' file.txt

# Printf formatting
awk '{printf "%-10s %5d\n", $1, $2}' file.txt

# External variables
awk -v threshold=100 '$2 > threshold {print}' file.txt

# Functions
awk '
function square(x) { return x * x }
{print $1, square($1)}
' numbers.txt
```

---

## grep and Regex

### Basic grep
```bash
# Basic search
grep "pattern" file.txt

# Case insensitive
grep -i "pattern" file.txt

# Recursive search
grep -r "pattern" directory/

# Show line numbers
grep -n "pattern" file.txt

# Count matches
grep -c "pattern" file.txt

# Show only filenames
grep -l "pattern" *.txt

# Invert match
grep -v "pattern" file.txt

# Context lines
grep -A 3 "pattern" file.txt  # After
grep -B 3 "pattern" file.txt  # Before
grep -C 3 "pattern" file.txt  # Both
```

### Extended Regex
```bash
# Extended regex (-E)
grep -E "pattern1|pattern2" file.txt
grep -E "^start" file.txt      # Line start
grep -E "end$" file.txt        # Line end
grep -E "[0-9]+" file.txt      # One or more digits
grep -E "col(ou)?r" file.txt   # Optional group

# Perl regex (-P) - Linux only
grep -P "\d{3}-\d{4}" file.txt
grep -P "(?<=prefix).*(?=suffix)" file.txt

# Common patterns
grep -E "^\s*$" file.txt              # Empty/whitespace lines
grep -E "^[^#]" file.txt              # Non-comment lines
grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" file.txt  # Emails
grep -E "\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b" file.txt  # IP addresses
```

### ripgrep (rg) - Faster Alternative
```bash
# Install
brew install ripgrep

# Basic usage (respects .gitignore)
rg "pattern" directory/

# Include hidden files
rg --hidden "pattern"

# Search specific types
rg -t py "import"
rg -t js "function"

# Regex
rg -e "pattern1" -e "pattern2"
```

---

## jq for JSON

### Basic Operations
```bash
# Pretty print
echo '{"name":"John"}' | jq '.'

# Extract field
echo '{"name":"John","age":30}' | jq '.name'

# Multiple fields
echo '{"name":"John","age":30}' | jq '.name, .age'

# Array elements
echo '[1,2,3]' | jq '.[0]'
echo '[1,2,3]' | jq '.[]'

# Array length
echo '[1,2,3]' | jq 'length'
```

### Filtering and Transforming
```bash
# Select with condition
jq '.[] | select(.active == true)' users.json

# Map transform
jq '.[] | {name: .name, email: .email}' users.json

# Filter array
jq '[.[] | select(.age > 18)]' users.json

# Sort
jq 'sort_by(.name)' users.json

# Group by
jq 'group_by(.department)' users.json

# Unique values
jq '[.[] | .category] | unique' items.json
```

### Creating and Modifying JSON
```bash
# Create JSON
jq -n --arg name "John" --arg email "john@example.com" \
    '{name: $name, email: $email}'

# Add field
jq '. + {newField: "value"}' input.json

# Update field
jq '.name = "Jane"' input.json

# Delete field
jq 'del(.unwantedField)' input.json

# Merge objects
jq -s '.[0] * .[1]' file1.json file2.json

# Array operations
jq '.items += ["new item"]' input.json
```

### Practical Examples
```bash
# Parse API response
curl -s "https://api.example.com/users" | jq '.data[].name'

# Format output
jq -r '.[] | "\(.name): \(.email)"' users.json

# CSV output
jq -r '.[] | [.name, .email] | @csv' users.json

# Tab-separated
jq -r '.[] | [.name, .email] | @tsv' users.json

# Compact output
jq -c '.' input.json
```

---

## Other Utilities

### cut
```bash
# Extract columns
cut -d',' -f1,3 file.csv     # Fields 1 and 3
cut -d':' -f1 /etc/passwd    # First field
cut -c1-10 file.txt          # Characters 1-10
```

### paste
```bash
# Merge files side by side
paste file1.txt file2.txt

# Custom delimiter
paste -d',' file1.txt file2.txt

# Serial (one file per line)
paste -s file.txt
```

### sort
```bash
# Basic sort
sort file.txt

# Numeric sort
sort -n numbers.txt

# Reverse
sort -r file.txt

# By field
sort -t',' -k2 file.csv

# Unique
sort -u file.txt

# Human numeric (1K, 2M)
sort -h sizes.txt
```

### uniq
```bash
# Remove duplicates (requires sorted input)
sort file.txt | uniq

# Count occurrences
sort file.txt | uniq -c

# Only duplicates
sort file.txt | uniq -d

# Only unique
sort file.txt | uniq -u
```

### tr
```bash
# Replace characters
tr 'a-z' 'A-Z' < file.txt

# Delete characters
tr -d '0-9' < file.txt

# Squeeze repeats
tr -s ' ' < file.txt

# Complement (delete everything except)
tr -cd '0-9\n' < file.txt
```

### xargs
```bash
# Basic usage
find . -name "*.txt" | xargs grep "pattern"

# Null separator (for spaces in names)
find . -name "*.txt" -print0 | xargs -0 grep "pattern"

# Max arguments
ls *.txt | xargs -n 1 echo

# Parallel execution
ls *.txt | xargs -P 4 -I {} process.sh {}

# With placeholder
find . -name "*.bak" | xargs -I {} mv {} /backup/
```
