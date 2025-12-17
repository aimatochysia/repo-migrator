# Repository Migrator

This repository contains a GitHub Actions workflow (`migrator.yml`) that migrates the complete commit history from a source repository to a target repository with custom author attribution.

## Purpose

The `migrator.yml` workflow allows you to:

1. **Migrate Complete Commit History**: Transfer all commits from the main branch of a source repository to a target repository
2. **Rewrite Author Attribution**: Assign all commits to a specified author and committer
3. **Squash Copilot Commits**: Automatically detect and squash commits made by GitHub Copilot into single commits
4. **Handle Merge Conflicts**: Automatically resolve conflicts by accepting changes from the source repository
5. **Clean Target Repository**: Completely clean the target repository before migration to ensure a fresh start

## How It Works

The workflow performs the following steps:

1. **Validates PAT**: Ensures the Personal Access Token is available in GitHub secrets
2. **Clones Source Repository**: Clones the source repository to access its commit history
3. **Prepares Target Repository**: Clones and completely cleans the target repository, removing all existing history and files
4. **Rewrites History**: 
   - Iterates through all commits in the source repository's main branch
   - Skips merge commits (their changes are already included via parent commits)
   - Identifies Copilot commits and accumulates them for squashing
   - Cherry-picks each commit with the new author/committer information
   - Automatically resolves merge conflicts using the "theirs" strategy (accepting changes from source)
   - Commits squashed Copilot changes when a non-Copilot commit is encountered
5. **Pushes to Target**: Force-pushes the rewritten history to the target repository

## Usage

### Prerequisites

1. A Personal Access Token (PAT) with appropriate permissions:
   - `repo` scope (full repository access)
   - Permissions to read from the source repository
   - Permissions to write to the target repository

2. Add the PAT as a secret named `PAT` in this repository's settings

### Running the Workflow

1. Navigate to the **Actions** tab in this repository
2. Select the **"Recommit Source Repo History to Target Repo"** workflow
3. Click **"Run workflow"**
4. Fill in the required inputs:
   - **Source repository**: The repository to migrate from (format: `owner/name`)
   - **Target repository**: The repository to migrate to (format: `owner/name`)
   - **Author name**: The name to use for all commits (e.g., "John Doe")
   - **Author email**: The email to use for all commits (e.g., "john.doe@example.com")
5. Click **"Run workflow"**

### Example

```yaml
Source repository: octocat/hello-world
Target repository: myorg/new-hello-world
Author name: John Doe
Author email: john.doe@example.com
```

## Features

### Automatic Conflict Resolution

The workflow automatically resolves merge conflicts by accepting changes from the source repository. This ensures the migration completes even if there are conflicts between commits.

### Copilot Commit Squashing

Commits authored by GitHub Copilot (identified by "copilot" in the author name or email) are automatically detected and squashed together. This keeps the target repository's history cleaner by combining automated commits.

### Preserved Commit Dates

While the author and committer are changed to the specified values, the original commit dates are preserved to maintain chronological accuracy.

### Merge Commit Handling

Merge commits are automatically skipped because their changes are already included in the history through their parent commits. This prevents duplicate changes and keeps the history linear.

## Important Notes

- **Destructive Operation**: The target repository is completely cleaned before migration. All existing content and history will be lost.
- **Force Push**: The workflow uses force push to overwrite the target repository's main branch.
- **Linear History**: The resulting history is linear (no merge commits) with a single author/committer.
- **Same Final State**: Despite potential conflicts during migration, the final state of files in the target repository will match the final state in the source repository.

## Troubleshooting

If the workflow fails:

1. **PAT Issues**: Verify the PAT secret is correctly set and has the necessary permissions
2. **Repository Access**: Ensure the PAT has access to both source and target repositories
3. **Branch Names**: The workflow assumes both repositories use `main` as the default branch
4. **Conflict Resolution**: If automatic conflict resolution fails, check the workflow logs for details

## Security Considerations

- Never commit the PAT directly in code
- Use GitHub Secrets to store the PAT securely
- Limit PAT permissions to only what's necessary
- Consider using fine-grained PATs for better security
- Review the target repository after migration to ensure expected results

## License

This workflow is provided as-is for repository migration purposes.
