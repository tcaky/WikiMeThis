# Bulk Delete projects
Assumptions:
1. projects are located in subdirectories below where this script is executed.
2. follow the naming standard we use for projects.

```bash
#!/bin/bash

# Enable error handling
set -euo pipefail

# Function to handle errors
function handle_error {
  echo "Error on line $1"
  exit 1
}
trap 'handle_error $LINENO' ERR

# Function to delete a project
delete_project() {
    local project_id="$1"

    # Validate GCP Project ID Format
    if [[ ! $project_id =~ ^(ph|hc)[xtpd]-([a-z0-9]{12})$ ]]; then
        echo "Error: Invalid project ID format for '$project_id'. Expected format: (ph|hc)[xtpd]-([a-z0-9]{12})"
        return 1
    fi

    # Extract components and create formatted variable
    project_prefix="${BASH_REMATCH[1]}"
    project_suffix="${BASH_REMATCH[2]}"
    project_var="${project_prefix}-${project_suffix}"

    echo "Processing project: $project_var"

    # Find the directory matching the project variable
    project_dir=$(find . -maxdepth 1 -type d -name "$project_var" -print -quit)

    if [[ -z "$project_dir" ]]; then
        echo "Error: No directory found matching '$project_var'"
        return 1
    fi

    echo "Found directory: $project_dir"

    # Git operations
    branch_name="${project_var}-project-deletion"

    # Create a new Git branch
    echo "Creating Git branch: $branch_name"
    git checkout -b "$branch_name"

    # Delete the directory
    echo "Deleting directory: $project_dir"
    rm -rf "$project_dir"

    # Stage the deletion
    git add .

    # Commit the deletion
    commit_message="${project_var} project deletion"
    git commit -m "$commit_message"

    # Push the branch to upstream
    git push origin "$branch_name"

    # Switch back to main branch
    git checkout main

    echo "Project deletion process completed for $project_var."
}

# If multiple project IDs are provided, loop through them
if [[ $# -eq 0 ]]; then
    echo "Usage: $0 <GCP Project ID 1> [GCP Project ID 2] ... [GCP Project ID N]"
    exit 1
fi

for project_id in "$@"; do
    delete_project "$project_id"
done
```