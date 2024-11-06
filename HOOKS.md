# Setting Up Manual Git Hooks

To set up manual hooks, follow these steps:

### 1. Create a folder to store the Hooks
```bash
mkdir .githooks
```

### 2. Write the scripts

Hooks are shell scripts that will run on your local machine.

#### 2.1 pre-commit hooks

This script will run Black on staged files. Save it under `.githooks/pre-commit`.

```bash
#!/bin/sh

# Run Black on all staged Python files
# Find staged files that end with .py and run black on them
STAGED_FILES=$(git diff --cached --name-only --diff-filter=d | grep '\.py$')

if [ -n "$STAGED_FILES" ]; then
    echo "Running Black on staged Python files..."
    black $STAGED_FILES

    # Add the formatted files back to the staging area
    git add $STAGED_FILES
fi
```
#### 2.2 commit-msg hooks

This script enforces a commit message length between 10 and 50 characters. 
 Save it under `.githooks/commit-msg`.

```bash
#!/bin/sh

# Path to the commit message file
COMMIT_MSG_FILE=$1
COMMIT_MSG=$(cat "$COMMIT_MSG_FILE")

# Check if commit message length is between 10 and 50 characters
if [ ${#COMMIT_MSG} -lt 10 ] || [ ${#COMMIT_MSG} -gt 50 ]; then
    echo "Error: Commit message must be between 10 and 50 characters."
    exit 1
fi

# Check if the commit message starts with a capital letter
if ! echo "$COMMIT_MSG" | grep -q "^[A-Z]"; then
    echo "Error: Commit message must start with a capital letter."
    exit 1
fi

# Check if the commit message ends with a period
if echo "$COMMIT_MSG" | grep -q "\.$"; then
    echo "Error: Commit message should not end with a period."
    exit 1
fi

# If all checks pass, allow the commit
exit 0
```

### 3. Give execution permissions

Add execution permissions for each hook:
```bash
chmod +x .githooks/pre-commit
chmod +x .githooks/commit-msg
```

### 4. Configure Git
Tell Git to use the custom hooks folder using `hooksPath`.
```bash
git config core.hooksPath .githooks
```

### 5. Commit the changes (Optional)

Commit the changes to standarize your pipeline across the entire team.
```bash
git add .githooks
git commit -m "Add custom Git hooks in .githooks directory"
```
