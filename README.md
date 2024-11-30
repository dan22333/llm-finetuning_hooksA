# Setting Up Manual Git Hooks for LLM Fine-tuning

This tutorial does not focus on implementing LLM finetuning. For that tutorial, refer [here](https://github.com/dlops-io/llm-finetuning/).

[Pre-commit hooks](https://git-scm.com/book/ms/v2/Customizing-Git-Git-Hooks#_client_side_hooks) are programs that automatically run when the user commits, but before the commit is registered on the index. It is used to inspect the code to check for syntax, run test or any other function.

This tutorial builds client-side hooks as scripts to run locally during the  pre-commit phase.

To set up manual hooks, follow these steps:

<!-- ### 1. 

```bash
touch .git/hooks/pre-commit
``` -->

### Step 1: Inspect the Samples
Git stores local hooks in the `.git/hooks/` folder. To enable them, create a file without the `.sample` extension.

You can find the different types and their details by

```
ls .git/hooks/
```

### Step 2: Create the Hooks
Hooks are shell scripts that will run on your local machine. 

#### 2.1: Create a Pre-Commit Hook
This script will run Black on staged files. Save it as `.git/hooks/pre-commit`.

```bash
#!/bin/sh

# Run Black on all staged Python files
# Identify staged Python files and format them with Black.
STAGED_FILES=$(git diff --cached --name-only --diff-filter=d | grep '\.py$')

if [ -n "$STAGED_FILES" ]; then
    echo "Running Black on staged Python files..."
    black $STAGED_FILES

    # Add the formatted files back to the staging area
    git add $STAGED_FILES
fi
```

#### 2.2: Commit-Message Hook

This script enforces a commit message length between 10 and 50 characters. 
 Save it under `.git/hooks/commit-msg`.

```bash
#!/bin/sh

# Path to the commit message file
COMMIT_MSG_FILE=$1
COMMIT_MSG=$(cat "$COMMIT_MSG_FILE")

# Check if commit message length is between 10 and 50 characters
if [ ${#COMMIT_MSG} -lt 10 ] || [ ${#COMMIT_MSG} -gt 50 ]; then
    echo "Error: Commit message length must be between 10 and 50 characters."
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

### Step 3: Give Execution Permissions

Add execution permissions for each hook. Without execution permissions, the scripts will not run.

```bash
chmod +x .git/hooks/pre-commit
chmod +x .git/hooks/commit-msg
```
<!-- 
### 3. Configure Git
Tell Git to use the custom hooks folder using `hooksPath`.
```bash
git config core.hooksPath .githooks
``` -->

### Step 4 (Optional): Create Shared Client-Side Hooks

Instead of writing the files on the `.git/hooks` folder, create them on a special folder, i.e., `.githooks/`.
This approach ensures all team members use the same Git hook configurations, enhancing consistency in the development workflow.

1. Create and configure the hooks as described in Steps 2 and 3 above.

2. Configure Git to use the custom hooks folder using `hooksPath`.
```bash
git config core.hooksPath .githooks
```

3. Commit the changes to standardize your pipeline across the team.

```bash
git add .githooks
git commit -m "Add custom Git hooks in .githooks directory"
```
4. After cloning the repository, team members should run:
```
git config core.hooksPath .githooks
chmod +x .githooks/*
```
