# GitLab Merge Request Creation and Response Posting

This document explains how Claude Code handles different scenarios after executing tasks in GitLab CI/CD.

## Overview

After Claude Code executes a task, the system automatically detects whether any code changes were made and takes appropriate action:

1. **With Code Changes**: Creates a merge request with the changes
2. **Without Code Changes**: Posts Claude's response as a comment on the original issue/MR

## Scenarios

### 1. Code Changes Detected

When Claude modifies files during execution:

1. **Branch Creation**: A new branch is created with the format `claude-{type}-{id}-{timestamp}`
2. **Commit**: Changes are committed with a descriptive message
3. **Push with MR Options**: The branch is pushed using GitLab push options to automatically create an MR:
   ```bash
   git push \
     -o merge_request.create \
     -o merge_request.target=main \
     -o merge_request.title="..." \
     -o merge_request.description="..." \
     -o merge_request.remove_source_branch \
     origin branch-name
   ```
4. **Comment Update**: A comment is posted on the original issue/MR with a link to the new MR

Example MR created by Claude:

```
Title: Apply Claude's suggestions for issue #42

Description:
This merge request was automatically created by Claude AI.

**Original issue**: https://gitlab.com/project/-/issues/42

**Claude's task**: Applied automated fixes and improvements as requested.

/cc @username
```

### 2. No Code Changes

When Claude provides analysis or suggestions without modifying code:

1. **Extract Response**: Claude's response is extracted from the execution output
2. **Format Message**: The response is formatted with markdown
3. **Post Comment**: The formatted response is posted as a comment on the original issue/MR

Example comment:

```markdown
## 🤖 Claude's Response

Based on my research, most of your AI SDK dependencies are already at or very close to their latest versions...

[Claude's full response here]

---

_This response was generated by Claude AI. No code changes were made._
```

## Configuration

### Environment Variables

The following GitLab CI variables are used:

- `CI_JOB_TOKEN`: For git authentication
- `CI_SERVER_HOST`: GitLab server hostname
- `CI_PROJECT_PATH`: Project path (namespace/project)
- `CI_DEFAULT_BRANCH`: Target branch for MRs (defaults to "main")
- `CLAUDE_RESOURCE_TYPE`: Type of resource (issue/merge_request)
- `CLAUDE_RESOURCE_ID`: ID of the resource
- `GITLAB_USER_LOGIN`: User to mention in MR description

### Git Configuration

The system automatically configures git with:

```bash
git config user.name "Claude[bot]"
git config user.email "claude-bot@noreply.gitlab.com"
```

## Implementation Details

The logic is implemented in `src/entrypoints/gitlab_entrypoint.ts`:

1. `checkGitStatus()`: Checks if there are any uncommitted changes
2. `createMergeRequest()`: Handles branch creation, commit, and MR creation
3. `postClaudeResponse()`: Extracts and posts Claude's response when no changes exist
4. `runUpdatePhase()`: Orchestrates the decision logic

## Error Handling

- If MR creation fails, the error is logged but doesn't fail the job
- If response posting fails, it's logged as a warning
- The tracking comment is always updated regardless of the outcome

## Security Considerations

- Uses `CI_JOB_TOKEN` for authentication (automatically provided by GitLab)
- No long-lived tokens are stored or exposed
- Git remote URL includes authentication: `https://gitlab-ci-token:${CI_JOB_TOKEN}@...`
