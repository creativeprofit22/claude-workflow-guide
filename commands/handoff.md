# Handoff Command - Complete

You are helping the user prepare their code changes for handoff. This command performs comprehensive pre-flight checks, validates the repository state, reviews all changes, commits safely, and generates a detailed handoff document.

---

## Step 0: Pre-flight Checks

First, run comprehensive pre-flight checks to ensure the repository is in a safe state:

1. **Check if in a git repository**:
```bash
git rev-parse --git-dir 2>&1
```

If this fails, inform the user:
> This directory is not a git repository. Initialize one with `git init` or navigate to an existing repository.

Then STOP - do not proceed further.

2. **Check for ongoing git operations**:
```bash
# Check for merge in progress
test -f .git/MERGE_HEAD && echo "MERGE_IN_PROGRESS"

# Check for rebase in progress
test -d .git/rebase-merge -o -d .git/rebase-apply && echo "REBASE_IN_PROGRESS"

# Check for cherry-pick in progress
test -f .git/CHERRY_PICK_HEAD && echo "CHERRY_PICK_IN_PROGRESS"
```

If any operation is in progress, inform the user:
> WARNING: A [merge/rebase/cherry-pick] is currently in progress. Please complete or abort it before using this command.
> - To complete: resolve conflicts and run `git [merge/rebase/cherry-pick] --continue`
> - To abort: run `git [merge/rebase/cherry-pick] --abort`

Then STOP - do not proceed further.

3. **Check for detached HEAD state**:
```bash
git symbolic-ref -q HEAD || echo "DETACHED_HEAD"
```

If in detached HEAD state, warn the user:
> WARNING: You are in a detached HEAD state. This means you're not on any branch.
> - To create a new branch here: `git checkout -b new-branch-name`
> - To return to a branch: `git checkout branch-name`

Ask the user if they want to continue anyway with options:
- [C]reate new branch and continue
- [S]witch to existing branch
- [A]bort handoff

4. **Check for submodules**:
```bash
test -f .gitmodules && cat .gitmodules
```

If submodules exist, inform the user:
> INFO: This repository contains git submodules. Submodule changes require special handling.
> Detected submodules:
> [list the submodules from .gitmodules]
>
> Note: This handoff will include submodule reference updates but not the submodule contents themselves.

5. **Show repository root and confirm scope**:
```bash
git rev-parse --show-toplevel
```

Display to the user:
> Repository root: [absolute path]
>
> This handoff command will operate on the entire repository from this root.
> Is this the correct scope for your handoff?
>
> Options:
> - [Y]es, proceed with full repository
> - [N]o, cancel and let me navigate to correct location
> - [I]nfo, show me more details about repository structure

If user selects [I]nfo, show:
```bash
# Get repo root first
REPO_ROOT=$(git rev-parse --show-toplevel)

# Show directory structure (top 2 levels)
tree -L 2 -a "$REPO_ROOT" 2>/dev/null || find "$REPO_ROOT" -maxdepth 2 -type d

# Show total file counts
git ls-files | wc -l
git ls-files --others --exclude-standard | wc -l
```

Wait for user confirmation before proceeding to Step 1.

---

## Step 1: Repository State Display

Gather and display FULL repository context before proceeding:

1. **Collect repository information**:
```bash
# Repository root
REPO_ROOT=$(git rev-parse --show-toplevel)

# Remote URL and name
REMOTE_NAME=$(git remote | head -1)
REMOTE_URL=$(git remote get-url $REMOTE_NAME 2>/dev/null || echo "No remote configured")

# Current branch
CURRENT_BRANCH=$(git symbolic-ref --short HEAD 2>/dev/null || echo "detached HEAD")

# Staged files count
STAGED_COUNT=$(git diff --cached --name-only | wc -l)

# Modified files count
MODIFIED_COUNT=$(git diff --name-only | wc -l)

# Untracked files count
UNTRACKED_COUNT=$(git ls-files --others --exclude-standard | wc -l)

# Last commit info
LAST_COMMIT=$(git log -1 --format="%h \"%s\" (%ar)" 2>/dev/null || echo "No commits yet")
```

2. **Display formatted repository state**:

Present the information in a clear, structured format:

```
====================================================================
REPOSITORY STATE
====================================================================

Repository: [REPO_ROOT]
Remote:     [REMOTE_URL] ([REMOTE_NAME])
Branch:     [CURRENT_BRANCH]
Status:     [STAGED_COUNT] staged, [MODIFIED_COUNT] modified, [UNTRACKED_COUNT] untracked
Last:       [LAST_COMMIT]

====================================================================
```

3. **Ask for explicit confirmation**:

> Please review the repository state above carefully.
>
> What would you like to do?
>
> - [Y]es - Proceed with handoff
> - [N]o - Cancel handoff
> - [C]hange URL - Modify the remote URL before proceeding
> - [V]iew details - Show detailed file listings and recent commits

4. **Handle user choice**:

**If [V]iew details selected**, show:
```bash
# Show last 5 commits
echo "=== Recent Commits ==="
git log -5 --oneline --decorate --graph

# Show staged files
echo -e "\n=== Staged Files ==="
git diff --cached --name-status

# Show modified files
echo -e "\n=== Modified Files ==="
git diff --name-status

# Show untracked files
echo -e "\n=== Untracked Files ==="
git ls-files --others --exclude-standard
```

Then return to the confirmation options.

**If [C]hange URL selected**, proceed to Step 2 (Remote Validation) with change mode enabled.

**If [N]o selected**, stop and inform:
> Handoff cancelled. No changes were made.

**If [Y]es selected**, proceed to Step 2 (Remote Validation).

---

## Step 2: Remote Validation

Validate and configure the remote repository:

1. **Check if remote exists**:
```bash
REMOTE_NAME=$(git remote | head -1)
REMOTE_URL=$(git remote get-url $REMOTE_NAME 2>/dev/null)
```

2. **Handle no remote case**:

If no remote is configured, inform the user:

> No remote repository is configured. A remote is recommended for sharing your code.
>
> What would you like to do?
>
> - [A]dd remote - Add a new remote URL (e.g., GitHub, GitLab)
> - [C]reate new - Help me create a new GitHub repository
> - [L]ocal only - Continue without a remote (local handoff only)
> - [S]top - Cancel the handoff

**If [A]dd remote selected**:
Ask the user to provide the remote URL, then:
```bash
git remote add origin [USER_PROVIDED_URL]
REMOTE_URL=[USER_PROVIDED_URL]
```

**If [C]reate new selected**:
> To create a new GitHub repository, you can:
> 1. Visit https://github.com/new to create a repository via the web
> 2. Use GitHub CLI: `gh repo create`
>
> After creating the repository, you'll receive a URL like:
> - HTTPS: https://github.com/username/repo.git
> - SSH: git@github.com:username/repo.git
>
> Would you like to add the remote URL now? [Y/N]

**If [L]ocal only selected**:
> Proceeding with local-only handoff. No remote synchronization will be performed.

Set a flag to skip remote operations and proceed to Step 3.

**If [S]top selected**:
> Handoff cancelled.

Stop execution.

3. **Validate remote connectivity** (if remote exists):

Test if the remote is reachable:
```bash
echo "Testing connection to remote..."
git ls-remote "$REMOTE_URL" 2>&1
```

If this fails, inform the user:
> WARNING: Cannot connect to remote: [REMOTE_URL]
>
> Possible issues:
> - Network connectivity problem
> - Invalid URL or repository doesn't exist
> - Authentication failure (check credentials)
> - SSH key not configured (for SSH URLs)
>
> What would you like to do?
>
> - [R]etry - Test connection again
> - [C]hange URL - Use a different remote URL
> - [L]ocal only - Continue without remote
> - [S]top - Cancel handoff

Handle the user's choice accordingly.

4. **Test push permissions** (if remote is valid):

```bash
echo "Testing push permissions..."
git push --dry-run "$REMOTE_NAME" "$CURRENT_BRANCH" 2>&1
```

Analyze the output:
- If successful: Proceed to Step 3
- If "permission denied" or "403":
  > ERROR: You don't have push permission to this remote.
  > Please check your access rights or use a different remote.

  Offer options: [C]hange URL / [L]ocal only / [S]top

- If "rejected" due to remote changes:
  > WARNING: The remote has changes that you don't have locally.
  > You'll need to pull and merge before pushing.

  Offer options: [P]ull now / [L]ocal only / [S]top

5. **Explain change URL option** (when presented):

When offering the [C]hange URL option, explain:

> The "Change URL" option allows you to modify where your code will be pushed.
>
> Use cases:
> - Switch from HTTPS to SSH (or vice versa)
> - Point to a different repository (e.g., a fork)
> - Fix an incorrect URL
> - Use a different git hosting service
>
> Current URL: [CURRENT_URL]
>
> What would you like to do?
> - [U]pdate - Change to a new URL (will modify existing remote)
> - [A]dd new - Add an additional remote (keeps existing one)
> - [B]ack - Return to previous menu

**If [U]pdate selected**:
```bash
git remote set-url $REMOTE_NAME [NEW_URL]
```

**If [A]dd new selected**:
Ask for remote name and URL:
```bash
git remote add [NEW_REMOTE_NAME] [NEW_URL]
```

After changing URL, re-run connectivity and permission tests.

Proceed to Step 3 once remote validation is complete or skipped.

---

## Step 3: Review Uncommitted Changes

Comprehensively review all changes before staging:

1. **Categorize and count all changes**:
```bash
# Get staged files
STAGED_FILES=$(git diff --cached --name-only)
STAGED_COUNT=$(echo "$STAGED_FILES" | grep -c . || echo 0)

# Get modified (unstaged) files
MODIFIED_FILES=$(git diff --name-only)
MODIFIED_COUNT=$(echo "$MODIFIED_FILES" | grep -c . || echo 0)

# Get untracked files
UNTRACKED_FILES=$(git ls-files --others --exclude-standard)
UNTRACKED_COUNT=$(echo "$UNTRACKED_FILES" | grep -c . || echo 0)
```

2. **Display categorized changes**:

```
====================================================================
UNCOMMITTED CHANGES REVIEW
====================================================================

STAGED FILES ($STAGED_COUNT):
[list each staged file with status: A=added, M=modified, D=deleted]

MODIFIED FILES ($MODIFIED_COUNT):
[list each modified file]

UNTRACKED FILES ($UNTRACKED_COUNT):
[list each untracked file]

====================================================================
```

Show detailed status:
```bash
# Staged files with status
git diff --cached --name-status

# Modified files
git diff --name-status

# Untracked files
git ls-files --others --exclude-standard
```

3. **Check for sensitive files and warn**:

Search for potentially sensitive files:
```bash
# Pattern list for sensitive files
SENSITIVE_PATTERNS="\.env$ \.env\. .*\.key$ .*\.pem$ .*\.p12$ .*\.pfx$ credentials config\.json secrets\. .*\.secrets id_rsa id_dsa .*_rsa$ .*_dsa$ \.htpasswd token\.txt .*\.credentials password\.txt"

# Check all files (staged + modified + untracked)
ALL_FILES=$(echo -e "$STAGED_FILES\n$MODIFIED_FILES\n$UNTRACKED_FILES" | sort -u)

# Find matches
SENSITIVE_FOUND=""
for pattern in $SENSITIVE_PATTERNS; do
  MATCHES=$(echo "$ALL_FILES" | grep -E "$pattern" || true)
  if [ -n "$MATCHES" ]; then
    SENSITIVE_FOUND="${SENSITIVE_FOUND}${MATCHES}\n"
  fi
done
```

If sensitive files are detected:

```
====================================================================
WARNING: POTENTIALLY SENSITIVE FILES DETECTED
====================================================================

The following files may contain sensitive information:

[list each sensitive file]

Common sensitive file types:
  .env, .env.* - Environment variables (API keys, passwords)
  *.key, *.pem - Private keys and certificates
  credentials.* - Credential files
  *token*, *secret*, *password* - Authentication data
  id_rsa, id_dsa - SSH private keys

RECOMMENDATION: Exclude these files before proceeding.

====================================================================
```

> SECURITY WARNING: Sensitive files detected (see above).
>
> These files should typically NOT be committed to version control.
> Sharing them could expose secrets, API keys, or credentials.

4. **Ask user to confirm what should be included**:

> Please review the changes above and decide what to include in the handoff.
>
> Options:
> - [A]ll - Stage all modified and untracked files (WARNING: includes sensitive files if any)
> - [S]elective - Let me choose which files to include/exclude
> - [E]xclude sensitive - Stage all except the sensitive files listed above
> - [C]urrent - Keep only currently staged files, don't stage anything new
> - [V]iew - Show detailed diffs for specific files
> - [N]one - Unstage everything and cancel
>
> What would you like to do?

5. **Handle user choices**:

**If [V]iew selected**:
Ask which files to view:
> Enter file path(s) to view (space-separated), or "staged"/"modified"/"untracked" for all in that category:

Then show:
```bash
git diff [--cached] -- [FILE_PATH]
```

Return to the options menu.

**If [A]ll selected**:
If sensitive files were detected, show additional confirmation:
> WARNING: This will include sensitive files. Are you absolutely sure?
> - [Y]es, I understand the risks
> - [N]o, take me back

If confirmed:
```bash
git add -A
```

**If [E]xclude sensitive selected**:
```bash
# Stage everything first
git add -A

# Then unstage sensitive files (handles paths with spaces)
echo -e "$SENSITIVE_FOUND" | while read -r file; do
  [ -n "$file" ] && git reset HEAD "$file"
done
```

Show summary:
> Staged all files except:
> [list excluded sensitive files]

**If [S]elective selected**:
> You can now:
> 1. Manually stage files with: `git add <file>`
> 2. Unstage files with: `git reset HEAD <file>`
> 3. View specific file diffs with: `git diff <file>`
>
> When ready, run this handoff command again.

Stop and let user manually stage files.

**If [C]urrent selected**:
> Keeping current staging state:
> - Staged: $STAGED_COUNT files
> - Not staged: $MODIFIED_COUNT modified, $UNTRACKED_COUNT untracked
>
> Proceeding with staged files only.

Continue to next step.

**If [N]one selected**:
```bash
git reset HEAD .
```
> All files unstaged. Handoff cancelled.

Stop execution.

6. **Final staging summary**:

After staging is complete, show:
```bash
# Final staged count
FINAL_STAGED=$(git diff --cached --name-only | wc -l)

echo "=== Final Staging Summary ==="
git diff --cached --stat
echo ""
echo "Total files staged: $FINAL_STAGED"
```

> Ready to proceed with $FINAL_STAGED staged files.
>
> Continue to commit and push? [Y/N]

If [Y], proceed to Step 4.
If [N], stop:
> Handoff paused. Staged files are ready whenever you want to continue.

---

## Step 4: Clean Redundant Files (Optional)

Identify and optionally remove redundant files to clean up the workspace before handoff.

### Find Cleanup Candidates

```bash
echo "=== Scanning for redundant files ==="
echo ""

# Find backup files
echo "--- Backup files (*.bak, *.orig, *.old, *~) ---"
backup_files=$(find . -type f \( -name "*.bak" -o -name "*.orig" -o -name "*.old" -o -name "*~" \) \
  -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/venv/*" \
  -not -path "*/vendor/*" 2>/dev/null)

if [ -n "$backup_files" ]; then
  echo "$backup_files" | while read -r f; do
    size=$(ls -lh "$f" 2>/dev/null | awk '{print $5}')
    date=$(ls -lh "$f" 2>/dev/null | awk '{print $6, $7, $8}')
    echo "  $f ($size, $date)"
  done
else
  echo "  None found"
fi

# Find empty files (excluding .gitkeep and intentional placeholders)
echo ""
echo "--- Empty files (excluding .gitkeep) ---"
empty_files=$(find . -type f -empty -not -name ".gitkeep" -not -name ".gitignore" \
  -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/venv/*" 2>/dev/null)

if [ -n "$empty_files" ]; then
  echo "$empty_files" | while read -r f; do
    date=$(ls -lh "$f" 2>/dev/null | awk '{print $6, $7, $8}')
    echo "  $f ($date)"
  done
else
  echo "  None found"
fi

# Find potentially orphaned test files
echo ""
echo "--- Potentially orphaned test files ---"
echo "  (test files without corresponding source files)"

orphaned_tests=""
for testfile in $(find . -type f \( -name "*.test.js" -o -name "*.test.ts" -o -name "*.spec.js" \
  -o -name "*.spec.ts" -o -name "*_test.py" -o -name "test_*.py" \) \
  -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/venv/*" 2>/dev/null); do

  # Extract base filename (remove test suffix/prefix)
  basename_file=$(basename "$testfile" | \
    sed -E 's/\.(test|spec)\.(js|ts|jsx|tsx)$/.\2/' | \
    sed -E 's/^test_(.*)\.py$/\1.py/' | \
    sed -E 's/^(.*)_test\.py$/\1.py/')

  dirname_file=$(dirname "$testfile")

  # Check if corresponding source exists
  if [ ! -f "$dirname_file/$basename_file" ] && \
     [ ! -f "$dirname_file/../src/$basename_file" ] && \
     [ ! -f "$dirname_file/../lib/$basename_file" ]; then
    echo "  $testfile (no matching source found)"
    orphaned_tests="$orphaned_tests$testfile\n"
  fi
done

if [ -z "$orphaned_tests" ]; then
  echo "  None found"
fi
```

### Review and Confirm Deletions

```bash
echo ""
echo "=== File Cleanup Options ==="
echo ""

# Count files
backup_count=$(echo "$backup_files" | grep -c . 2>/dev/null || echo 0)
empty_count=$(echo "$empty_files" | grep -c . 2>/dev/null || echo 0)
orphan_count=$(echo -e "$orphaned_tests" | grep -c . 2>/dev/null || echo 0)

echo "Found:"
echo "  - $backup_count backup files"
echo "  - $empty_count empty files"
echo "  - $orphan_count potentially orphaned test files"
echo ""

if [ "$backup_count" -eq 0 ] && [ "$empty_count" -eq 0 ] && [ "$orphan_count" -eq 0 ]; then
  echo "✓ No redundant files found. Skipping cleanup."
else
  echo "Cleanup options:"
  echo "  1. Review and delete files individually"
  echo "  2. Delete all backup files at once"
  echo "  3. Delete all empty files at once"
  echo "  4. Skip cleanup (proceed to next step)"
  echo ""
  read -p "Choose option (1-4): " cleanup_choice

  case "$cleanup_choice" in
    1)
      # Individual review
      echo ""
      echo "=== Reviewing files individually ==="

      # Function to preview and delete
      review_and_delete() {
        local filepath="$1"

        echo ""
        echo "--- File: $filepath ---"
        ls -lh "$filepath" 2>/dev/null

        # Show content preview for text files
        if file "$filepath" 2>/dev/null | grep -q "text"; then
          echo ""
          echo "Preview (first 10 lines):"
          head -n 10 "$filepath" 2>/dev/null
        fi

        echo ""
        read -p "Delete this file? (y/n/q to quit): " delete_choice

        case "$delete_choice" in
          y|Y)
            # Check if tracked by git
            if git ls-files --error-unmatch "$filepath" >/dev/null 2>&1; then
              git rm "$filepath" 2>/dev/null && echo "✓ Deleted (git rm): $filepath"
            else
              rm "$filepath" 2>/dev/null && echo "✓ Deleted: $filepath"
            fi
            ;;
          q|Q)
            echo "Cleanup cancelled"
            return 1
            ;;
          *)
            echo "Skipped: $filepath"
            ;;
        esac
        return 0
      }

      # Review backup files
      if [ -n "$backup_files" ]; then
        echo "$backup_files" | while read -r f; do
          review_and_delete "$f" || break
        done
      fi

      # Review empty files
      if [ -n "$empty_files" ]; then
        echo "$empty_files" | while read -r f; do
          review_and_delete "$f" || break
        done
      fi
      ;;

    2)
      # Delete all backup files
      if [ -n "$backup_files" ]; then
        echo ""
        echo "Deleting all backup files..."
        echo "$backup_files" | while read -r f; do
          if git ls-files --error-unmatch "$f" >/dev/null 2>&1; then
            git rm "$f" 2>/dev/null && echo "✓ Deleted: $f"
          else
            rm "$f" 2>/dev/null && echo "✓ Deleted: $f"
          fi
        done
        echo "✓ All backup files deleted"
      else
        echo "No backup files to delete"
      fi
      ;;

    3)
      # Delete all empty files
      if [ -n "$empty_files" ]; then
        echo ""
        echo "Deleting all empty files..."
        echo "$empty_files" | while read -r f; do
          if git ls-files --error-unmatch "$f" >/dev/null 2>&1; then
            git rm "$f" 2>/dev/null && echo "✓ Deleted: $f"
          else
            rm "$f" 2>/dev/null && echo "✓ Deleted: $f"
          fi
        done
        echo "✓ All empty files deleted"
      else
        echo "No empty files to delete"
      fi
      ;;

    *)
      echo "Skipping cleanup"
      ;;
  esac
fi

echo ""
echo "✓ File cleanup step complete"
```

---

## Step 5: Git Operations (With Checkpoints)

Commit and push changes with safety checkpoints and clear feedback.

### Show What Will Be Committed

```bash
echo ""
echo "=== Preparing to commit changes ==="
echo ""

# Check if there are changes to commit
if git diff --cached --quiet; then
  echo "No changes to commit"
else
  echo "=== Changes to be committed ==="
  echo ""
  git status --short

  echo ""
  echo "=== Diff summary ==="
  git diff --cached --stat

  echo ""
  echo "=== Detailed diff (first 50 lines) ==="
  git diff --cached | head -50
  echo ""

  # Count changes
  added=$(git diff --cached --numstat 2>/dev/null | awk '{sum+=$1} END {print sum+0}')
  removed=$(git diff --cached --numstat 2>/dev/null | awk '{sum+=$2} END {print sum+0}')
  files=$(git diff --cached --numstat 2>/dev/null | wc -l)

  echo "Summary: $files files changed, $added insertions(+), $removed deletions(-)"
fi
```

### Get Commit Message from User

```bash
echo ""
echo "=== Commit Message ==="
echo ""

# Analyze changes to suggest messages
echo "Suggested commit messages based on changes:"
echo ""

suggestion_num=1

if git diff --cached --name-only | grep -q "package.json\|requirements.txt\|go.mod\|Cargo.toml"; then
  echo "$suggestion_num. Update dependencies and project configuration"
  ((suggestion_num++))
fi

if git diff --cached --name-only | grep -q "README\|CHANGELOG\|\.md$"; then
  echo "$suggestion_num. Update documentation"
  ((suggestion_num++))
fi

if git diff --cached --name-only | grep -q "test\|spec"; then
  echo "$suggestion_num. Add/update tests"
  ((suggestion_num++))
fi

echo "$suggestion_num. Refactor and improve code quality"
((suggestion_num++))
echo "$suggestion_num. Add new features and functionality"
((suggestion_num++))
echo "$suggestion_num. Clean up and prepare for handoff"
((suggestion_num++))
echo "$suggestion_num. Custom message (enter manually)"

echo ""
read -p "Select a commit message (1-$suggestion_num) or press Enter for custom: " msg_choice

case "$msg_choice" in
  1) commit_msg="Update dependencies and project configuration" ;;
  2) commit_msg="Update documentation" ;;
  3) commit_msg="Add/update tests" ;;
  4) commit_msg="Refactor and improve code quality" ;;
  5) commit_msg="Add new features and functionality" ;;
  6) commit_msg="Clean up and prepare for handoff" ;;
  *)
    echo "Enter your commit message:"
    read -r commit_msg
    ;;
esac

if [ -z "$commit_msg" ]; then
  commit_msg="Clean up and prepare for handoff"
fi

echo ""
echo "Commit message: $commit_msg"
read -p "Proceed with this message? (y/n): " confirm_msg

if [ "$confirm_msg" != "y" ] && [ "$confirm_msg" != "Y" ]; then
  echo "Commit cancelled"
  exit 1
fi
```

### Create Pre-Handoff Tag for Rollback

```bash
echo ""
echo "=== Creating rollback checkpoint ==="

# Create tag with timestamp
tag_name="pre-handoff-$(date +%Y%m%d-%H%M%S)"
git tag -a "$tag_name" -m "Checkpoint before handoff on $(date)" 2>/dev/null

if [ $? -eq 0 ]; then
  echo "✓ Created rollback tag: $tag_name"
  echo "  To rollback: git reset --hard $tag_name"
else
  echo "⚠ Could not create tag (may already exist or no commits yet)"
  tag_name=""
fi
```

### Commit Changes

```bash
echo ""
echo "=== Committing changes ==="

git commit -m "$commit_msg"

if [ $? -eq 0 ]; then
  echo "✓ Commit successful"
  commit_hash=$(git rev-parse HEAD)
  commit_msg_saved=$(git log -1 --pretty=%B)
  echo "  Commit hash: $commit_hash"
else
  echo "✗ Commit failed"
  exit 1
fi
```

### Push with Progress Feedback

```bash
if [ "$skip_push" != "true" ]; then
  echo ""
  echo "=== Pushing to remote ==="

  # Get current branch
  current_branch=$(git branch --show-current)

  # Check if branch has upstream
  upstream=$(git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null)

  if [ -z "$upstream" ]; then
    echo "Branch has no upstream. Setting upstream to origin/$current_branch"
    git push -u origin "$current_branch" --progress 2>&1
  else
    git push --progress 2>&1
  fi

  push_exit_code=$?

  if [ $push_exit_code -eq 0 ]; then
    echo "✓ Push successful"
    echo "  Remote: $remote_url"
    echo "  Branch: $current_branch"
  else
    echo "✗ Push failed (exit code: $push_exit_code)"
    echo ""
    echo "Common reasons and solutions:"
    echo "1. No upstream branch set"
    echo "   Solution: git push -u origin $current_branch"
    echo ""
    echo "2. Remote has changes you don't have (rejected)"
    echo "   Solution: git pull --rebase && git push"
    echo ""
    echo "3. Authentication failed"
    echo "   Solution: Check your git credentials or SSH keys"
    echo ""
    echo "4. Network issues"
    echo "   Solution: Check internet connection and try again"
    echo ""
    read -p "Would you like to try 'git pull --rebase && git push'? (y/n): " retry

    if [ "$retry" = "y" ] || [ "$retry" = "Y" ]; then
      git pull --rebase && git push --progress 2>&1
      if [ $? -eq 0 ]; then
        echo "✓ Push successful after rebase"
        push_exit_code=0
      else
        echo "✗ Push still failed. Manual intervention required."
      fi
    else
      echo "Push skipped. You can manually push later with: git push"
    fi
  fi
else
  echo ""
  echo "⚠ Push skipped (no remote configured)"
  push_exit_code=1
fi
```

---

## Step 6: Generate Comprehensive Handoff Document

Create a detailed handoff document with all context needed for the next developer.

```bash
echo ""
echo "=== Generating handoff document ==="

# Gather information
project_name=$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || basename "$(pwd)")
timestamp=$(date "+%Y-%m-%d %H:%M:%S %Z")
current_branch=$(git branch --show-current 2>/dev/null || echo "unknown")
commit_hash=$(git rev-parse HEAD 2>/dev/null || echo "N/A")

# Detect project type and dependencies
runtime=""
runtime_version=""
package_manager=""
install_cmd=""
start_cmd=""
test_cmd=""
build_cmd=""

# Node.js project
if [ -f "package.json" ]; then
  runtime="Node.js"
  runtime_version=$(node --version 2>/dev/null || echo "Not installed")

  if [ -f "package-lock.json" ]; then
    package_manager="npm"
    install_cmd="npm install"
  elif [ -f "yarn.lock" ]; then
    package_manager="yarn"
    install_cmd="yarn install"
  elif [ -f "pnpm-lock.yaml" ]; then
    package_manager="pnpm"
    install_cmd="pnpm install"
  else
    package_manager="npm"
    install_cmd="npm install"
  fi

  # Extract scripts from package.json
  start_cmd=$(grep -A 1 '"start"' package.json 2>/dev/null | tail -1 | grep -o '"[^"]*"' | tail -1 | tr -d '"' || echo "npm start")
  test_cmd=$(grep -A 1 '"test"' package.json 2>/dev/null | tail -1 | grep -o '"[^"]*"' | tail -1 | tr -d '"' || echo "npm test")
  build_cmd=$(grep -A 1 '"build"' package.json 2>/dev/null | tail -1 | grep -o '"[^"]*"' | tail -1 | tr -d '"' || echo "npm run build")
fi

# Python project
if [ -f "requirements.txt" ] || [ -f "setup.py" ] || [ -f "pyproject.toml" ]; then
  runtime="Python"
  runtime_version=$(python3 --version 2>/dev/null || python --version 2>/dev/null || echo "Not installed")

  if [ -f "requirements.txt" ]; then
    install_cmd="pip install -r requirements.txt"
  elif [ -f "setup.py" ]; then
    install_cmd="pip install -e ."
  elif [ -f "pyproject.toml" ]; then
    install_cmd="pip install ."
  fi

  test_cmd="pytest"
fi

# Go project
if [ -f "go.mod" ]; then
  runtime="Go"
  runtime_version=$(go version 2>/dev/null | awk '{print $3}' || echo "Not installed")
  install_cmd="go mod download"
  build_cmd="go build"
  test_cmd="go test ./..."
fi

# Ruby project
if [ -f "Gemfile" ]; then
  runtime="Ruby"
  runtime_version=$(ruby --version 2>/dev/null || echo "Not installed")
  install_cmd="bundle install"
  test_cmd="bundle exec rspec"
fi

# Rust project
if [ -f "Cargo.toml" ]; then
  runtime="Rust"
  runtime_version=$(rustc --version 2>/dev/null || echo "Not installed")
  install_cmd="cargo build"
  build_cmd="cargo build --release"
  test_cmd="cargo test"
fi

# Get recent commits
recent_commits=$(git log --oneline -10 2>/dev/null)

# Find TODO/FIXME items
todos=$(grep -rn "TODO\|FIXME\|XXX\|HACK" --include="*.js" --include="*.ts" --include="*.py" \
  --include="*.go" --include="*.rb" --include="*.rs" --include="*.java" --include="*.c" \
  --include="*.cpp" --include="*.h" . 2>/dev/null | head -20)

# Find environment variables
env_vars=""
if [ -f ".env.example" ]; then
  env_vars=$(grep "^[A-Z_]*=" .env.example 2>/dev/null | cut -d'=' -f1 | sort -u)
elif [ -f ".env.template" ]; then
  env_vars=$(grep "^[A-Z_]*=" .env.template 2>/dev/null | cut -d'=' -f1 | sort -u)
fi

# Identify key files
key_files=$(find . -maxdepth 2 -type f \( -name "*.md" -o -name "package.json" -o -name "requirements.txt" \
  -o -name "go.mod" -o -name "Cargo.toml" -o -name "docker-compose.yml" -o -name "Dockerfile" \
  -o -name "Makefile" -o -name ".env.example" \) -not -path "*/node_modules/*" \
  -not -path "*/.git/*" 2>/dev/null | sed 's|^\./||')

# Create HANDOFF.md
cat > HANDOFF.md << 'HANDOFF_EOF'
# Project Handoff - PROJECT_NAME

**Date:** TIMESTAMP
**Repository:** REPO_URL
**Branch:** CURRENT_BRANCH
**Commit:** COMMIT_HASH
**Last Commit Message:** "COMMIT_MSG"

---

## Current State

This project is currently in active development. The codebase has been organized, dependencies updated, and code quality checks have been run. All changes have been committed and pushed to the remote repository.

**Status:** Ready for continued development

---

## Environment & Dependencies

### Runtime
- **Platform:** RUNTIME
- **Version:** RUNTIME_VERSION
- **Package Manager:** PACKAGE_MANAGER

### Key Dependencies
See dependency files (package.json, requirements.txt, etc.) for complete list.

### External Services
Review code for API integrations and external service dependencies.

### Environment Variables Required
ENV_VARIABLES

---

## Build & Test Status

**Install Command:** `INSTALL_COMMAND`
**Build Command:** `BUILD_COMMAND`
**Test Command:** `TEST_COMMAND`
**Start Command:** `START_COMMAND`

**Recommended: Run tests and build before continuing development.**

---

## Recent Changes (Last 10 Commits)

```
RECENT_COMMITS
```

---

## In Progress / TODOs

TODO_ITEMS

---

## Blocked Items

**External Dependencies:** None identified
**Waiting On:** N/A

---

## Key Files & Structure

KEY_FILES_LIST

---

## How to Resume Development

### 1. Clone Repository
```bash
git clone REPO_URL
cd PROJECT_NAME
```

### 2. Checkout Branch
```bash
git checkout CURRENT_BRANCH
```

### 3. Install Dependencies
```bash
INSTALL_COMMAND
```

### 4. Setup Environment
```bash
# Copy environment template (if exists)
cp .env.example .env 2>/dev/null || echo "No .env.example found"

# Configure required environment variables:
ENV_VARIABLES_LIST
```

### 5. Verify Installation
```bash
# Run tests
TEST_COMMAND

# Build project
BUILD_COMMAND
```

### 6. Start Development
```bash
# Start development server
START_COMMAND
```

**Next Steps:**
- Review the recent commits above to understand latest changes
- Check TODO items for pending work
- Run tests before making changes

---

## Next Steps

### Immediate Priorities
1. Review and address any TODO/FIXME items in code
2. Update tests for recent changes
3. Consider adding integration tests if missing
4. Update documentation as features evolve

### Suggested Improvements
1. Add CI/CD pipeline if not present
2. Improve error handling and logging
3. Add performance monitoring
4. Enhance test coverage

---

## Rollback Information

ROLLBACK_INFO

---

## Contact & Resources

**Documentation:** See README.md
**Issues:** Check repository issues
**Contributing:** See CONTRIBUTING.md (if present)

---

*This handoff document was automatically generated on TIMESTAMP*
HANDOFF_EOF

# Escape ampersands in variables for sed (& is special in sed replacement)
escape_for_sed() {
  echo "$1" | sed 's/&/\\&/g'
}

# Replace placeholders with actual values
# IMPORTANT: Replace longer placeholders first to avoid partial matches
# e.g., RUNTIME_VERSION must be replaced before RUNTIME
sed -i "s|RUNTIME_VERSION|$(escape_for_sed "${runtime_version:-N/A}")|g" HANDOFF.md
sed -i "s|RUNTIME|$(escape_for_sed "${runtime:-Not detected}")|g" HANDOFF.md
sed -i "s|INSTALL_COMMAND|$(escape_for_sed "${install_cmd:-Not configured}")|g" HANDOFF.md
sed -i "s|TEST_COMMAND|$(escape_for_sed "${test_cmd:-Not configured}")|g" HANDOFF.md
sed -i "s|BUILD_COMMAND|$(escape_for_sed "${build_cmd:-Not configured}")|g" HANDOFF.md
sed -i "s|START_COMMAND|$(escape_for_sed "${start_cmd:-Not configured}")|g" HANDOFF.md
sed -i "s|PACKAGE_MANAGER|$(escape_for_sed "${package_manager:-N/A}")|g" HANDOFF.md
sed -i "s|PROJECT_NAME|$(escape_for_sed "$project_name")|g" HANDOFF.md
sed -i "s|TIMESTAMP|$(escape_for_sed "$timestamp")|g" HANDOFF.md
sed -i "s|REPO_URL|$(escape_for_sed "${remote_url:-Local repository (no remote)}")|g" HANDOFF.md
sed -i "s|CURRENT_BRANCH|$(escape_for_sed "$current_branch")|g" HANDOFF.md
sed -i "s|COMMIT_HASH|$(escape_for_sed "$commit_hash")|g" HANDOFF.md
sed -i "s|COMMIT_MSG|$(escape_for_sed "${commit_msg_saved:-N/A}")|g" HANDOFF.md

# Insert recent commits
if [ -n "$recent_commits" ]; then
  echo "$recent_commits" > /tmp/handoff_commits.txt
  sed -i "/RECENT_COMMITS/{
    r /tmp/handoff_commits.txt
    d
  }" HANDOFF.md
else
  sed -i "s|RECENT_COMMITS|No commits found|g" HANDOFF.md
fi

# Insert TODO items
if [ -n "$todos" ]; then
  echo "Found TODO/FIXME items:" > /tmp/handoff_todos.txt
  echo "" >> /tmp/handoff_todos.txt
  echo "$todos" | head -15 >> /tmp/handoff_todos.txt
  sed -i "/TODO_ITEMS/{
    r /tmp/handoff_todos.txt
    d
  }" HANDOFF.md
else
  sed -i "s|TODO_ITEMS|No TODO items found in codebase.|g" HANDOFF.md
fi

# Insert environment variables
if [ -n "$env_vars" ]; then
  echo "$env_vars" | sed 's/^/- /' > /tmp/handoff_envvars.txt
  sed -i "/ENV_VARIABLES_LIST/{
    r /tmp/handoff_envvars.txt
    d
  }" HANDOFF.md
  sed -i "/ENV_VARIABLES/{
    r /tmp/handoff_envvars.txt
    d
  }" HANDOFF.md
else
  sed -i "s|ENV_VARIABLES|No environment variables detected (check .env.example if it exists)|g" HANDOFF.md
  sed -i "s|ENV_VARIABLES_LIST|# No environment variables detected|g" HANDOFF.md
fi

# Insert key files
if [ -n "$key_files" ]; then
  echo "$key_files" | sed 's/^/- /' > /tmp/handoff_keyfiles.txt
  sed -i "/KEY_FILES_LIST/{
    r /tmp/handoff_keyfiles.txt
    d
  }" HANDOFF.md
else
  sed -i "s|KEY_FILES_LIST|- No key files identified|g" HANDOFF.md
fi

# Insert rollback information
if [ -n "$tag_name" ]; then
  rollback_text="**Pre-handoff checkpoint:** \`$tag_name\`

To rollback to state before this handoff:
\`\`\`bash
git reset --hard $tag_name
\`\`\`"
  echo "$rollback_text" > /tmp/handoff_rollback.txt
  sed -i "/ROLLBACK_INFO/{
    r /tmp/handoff_rollback.txt
    d
  }" HANDOFF.md
else
  sed -i "s|ROLLBACK_INFO|No rollback tag created.|g" HANDOFF.md
fi

echo "✓ Created HANDOFF.md"
```

---

## Step 7: Summary & Verification

Display a comprehensive summary of everything that was done.

```bash
echo ""
echo "=============================================="
echo "      HANDOFF COMPLETE"
echo "=============================================="
echo ""

# Summary of actions
echo "=== Actions Completed ==="
echo ""

# File cleanup summary
if [ -n "$backup_files" ] || [ -n "$empty_files" ]; then
  deleted_count=$(($(echo "$backup_files" | grep -c .) + $(echo "$empty_files" | grep -c .)))
  if [ "$deleted_count" -gt 0 ]; then
    echo "✓ Reviewed and cleaned up redundant files"
  else
    echo "○ No files cleaned (skipped or none selected)"
  fi
else
  echo "○ No redundant files found"
fi

# Git operations summary
if [ -n "$tag_name" ]; then
  echo "✓ Created rollback checkpoint: $tag_name"
fi

echo "✓ Committed changes: $commit_hash"
echo "  Message: $commit_msg_saved"

# Push status
if [ "$skip_push" = "true" ]; then
  echo "○ Push skipped (no remote configured)"
elif [ $push_exit_code -eq 0 ]; then
  echo "✓ Pushed to remote: $remote_url"
  echo "  Branch: $current_branch"
else
  echo "⚠ Push failed - manual push may be required"
fi

# Handoff document
handoff_path="$(pwd)/HANDOFF.md"
echo "✓ Generated handoff document: $handoff_path"

echo ""
echo "=== Handoff Document Location ==="
echo "$handoff_path"
echo ""

# Show remote repository link
if [ -n "$remote_url" ] && [ "$remote_url" != "Local repository (no remote)" ]; then
  echo "=== Remote Repository ==="
  echo "$remote_url"

  # Convert SSH URL to HTTPS for clickable link
  if [[ "$remote_url" == git@* ]]; then
    https_url=$(echo "$remote_url" | sed 's|git@\([^:]*\):\(.*\)\.git|https://\1/\2|' | sed 's|\.git$||')
    echo "Web: $https_url"
  fi
  echo ""
fi

# Rollback information
if [ -n "$tag_name" ]; then
  echo "=== Rollback Information ==="
  echo "If you need to undo this handoff:"
  echo "  git reset --hard $tag_name"
  echo ""
fi

# Warnings and items needing attention
echo "=== Items Needing Attention ==="
warning_count=0

# Check for uncommitted changes
if ! git diff-index --quiet HEAD -- 2>/dev/null; then
  echo "⚠ WARNING: Uncommitted changes still present"
  ((warning_count++))
fi

# Check for unpushed commits
if [ "$skip_push" != "true" ]; then
  unpushed=$(git log @{u}.. --oneline 2>/dev/null | wc -l)
  if [ "$unpushed" -gt 0 ]; then
    echo "⚠ WARNING: $unpushed unpushed commits"
    ((warning_count++))
  fi
fi

# Check for TODO items
if [ -n "$todos" ]; then
  todo_count=$(echo "$todos" | wc -l)
  echo "ℹ INFO: $todo_count TODO/FIXME items in codebase (documented in HANDOFF.md)"
fi

# Suggest running tests
if [ -n "$test_cmd" ]; then
  echo "ℹ INFO: Run '$test_cmd' to verify tests before final handoff"
fi

if [ $warning_count -eq 0 ]; then
  echo "✓ No warnings - repository is clean"
fi

echo ""
echo "=== Next Steps for Recipient ==="
echo "1. Read $handoff_path"
if [ -n "$remote_url" ] && [ "$remote_url" != "Local repository (no remote)" ]; then
  echo "2. Clone repository: git clone $remote_url"
else
  echo "2. Access this directory: cd $(pwd)"
fi
echo "3. Install dependencies: ${install_cmd:-check project files}"
echo "4. Review recent commits and TODOs"
echo "5. Run tests: ${test_cmd:-check project files for test command}"
echo ""

echo "=============================================="
echo "  Handoff package ready!"
echo "=============================================="
```

---

## Usage Notes

This comprehensive handoff command handles:

1. **Pre-flight checks** - Ensures repository is in a safe state
2. **Repository verification** - Displays complete state and gets confirmation
3. **Remote validation** - Confirms and validates remote with connectivity tests
4. **Change review** - Comprehensive review with security warnings
5. **Optional cleanup** - Identifies and removes redundant files safely
6. **Safe git operations** - Creates checkpoints, confirms actions, handles errors
7. **Comprehensive documentation** - Auto-generates detailed handoff with context
8. **Clear summary** - Shows what was done and what needs attention

### Error Handling

- All git operations have rollback points via tags
- Push failures offer recovery options
- File deletions require confirmation
- Sensitive files get special warnings
- All operations provide clear feedback

### Safety Features

- Pre-flight checks prevent unsafe operations
- Preview before delete
- Rollback tags before commits
- Confirmation prompts for destructive actions
- Detailed logging of all operations
- Never pushes without explicit confirmation
- Security scanning for sensitive files

### Customization

Projects can customize:
- File cleanup patterns (add project-specific patterns in Step 4)
- Commit message templates (modify suggestions in Step 5)
- Handoff document sections (edit template in Step 6)
- Environment detection (add custom project types in Step 6)
- Sensitive file patterns (edit SENSITIVE_PATTERNS in Step 3)
