# Bulk Delete projects
Assumptions:
1. gh cli tools are installed (this create PRs)
2. follow the naming standard we use for projects.


```bash
#!/bin/bash

# Enable robust error handling
set -euo pipefail

# Function to handle errors
function handle_error {
    echo "Error on line $1"
    exit 1
}
trap 'handle_error $LINENO' ERR

# Function to delete a project and create a PR
delete_project() {
    local project_id="$1"
    local search_path="$2"

    # Validate GCP Project ID Format (allowing variable-length IDs)
    if [[ ! $project_id =~ ^(ph|hc)[xtpd]-([a-z0-9]+)$ ]]; then
        echo "Error: Invalid project ID format for '$project_id'. Expected format: (ph|hc)[xtpd]-([a-z0-9]+)"
        return 1
    fi

    # Extract components and create formatted variable
    project_prefix="${BASH_REMATCH[1]}"
    project_suffix="${BASH_REMATCH[2]}"
    project_var="${project_prefix}-${project_suffix}"

    echo "Processing project: $project_var in $search_path"

    # Ensure the search path exists
    if [[ ! -d "$search_path" ]]; then
        echo "Error: The specified search path '$search_path' does not exist."
        return 1
    fi

    # Find the directory matching the project variable inside the specified path
    project_dir=$(find "$search_path" -maxdepth 1 -type d -name "$project_var" -print -quit)

    if [[ -z "$project_dir" ]]; then
        echo "Error: No directory found matching '$project_var' in '$search_path'"
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

    # Create a Pull Request using GitHub CLI (`gh`)
    echo "Creating GitHub Pull Request..."
    gh pr create --base main --head "$branch_name" --title "$branch_name" --body "Automated deletion of $project_var project directory."

    # Switch back to main branch
    git checkout main

    echo "Project deletion process completed for $project_var."
}

# Ensure at least two arguments are provided
if [[ $# -lt 2 ]]; then
    echo "Usage: $0 <search_path> <GCP Project ID 1> [GCP Project ID 2] ... [GCP Project ID N]"
    exit 1
fi

# Extract search path and shift arguments
search_path="$1"
shift

# Process each project ID
for project_id in "$@"; do
    delete_project "$project_id" "$search_path"
done

```