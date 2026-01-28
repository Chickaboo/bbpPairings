# Automated Upstream Sync and Build Workflow

This repository includes an automated GitHub Actions workflow that syncs with the upstream repository and produces builds.

## Overview

The `sync-and-build.yml` workflow automatically:

1. **Checks for upstream changes** on a bi-monthly schedule (1st and 15th of each month)
2. **Syncs with upstream** if new commits are detected
3. **Builds artifacts** for Linux and Windows platforms
4. **Publishes artifacts** to a rolling "latest" release

## Workflow Details

### Schedule

The workflow runs automatically:
- On the **1st of each month** at 00:00 UTC
- On the **15th of each month** at 00:00 UTC

This bi-monthly schedule minimizes CI usage while keeping the fork reasonably up-to-date.

### Manual Triggers

You can also trigger the workflow manually:
1. Go to the **Actions** tab in GitHub
2. Select **Sync Upstream and Build** workflow
3. Click **Run workflow**
4. Optionally check **Force sync** to sync even if no new commits are detected

### Behavior

#### Checking for Changes

The workflow:
- Fetches the latest commits from upstream (`BieremaBoyzProgramming/bbpPairings`)
- Compares the upstream default branch with the current branch
- Only proceeds to sync if new commits are detected
- Skips the build if no changes are found (saving CI minutes)

#### Syncing

If new commits are detected:
- Attempts to merge upstream changes into the current branch
- If merge succeeds: proceeds to build
- If merge conflicts occur: 
  - Aborts the merge
  - Logs a warning
  - Fails gracefully (manual intervention required)

#### Building

After a successful sync:
- Builds static binaries for **Linux** (Ubuntu) and **Windows**
- Uses the existing Makefile with `static=yes` flag
- Creates distribution packages (`.tar.gz` for Linux, `.zip` for Windows)

#### Publishing

All build artifacts are published to a rolling release:
- **Tag name**: `latest-{branch-name}` (e.g., `latest-main`)
- **Release name**: "Latest Automated Build - {branch} ({date})"
- **Artifacts**: Named with branch and date (e.g., `bbpPairings-linux-main-2024-01-15.tar.gz`)
- The release is **continuously updated** - old artifacts are replaced with new ones

### Release Contents

Each rolling release includes:
- **Linux build**: Static binary distribution for Linux systems
- **Windows build**: Static binary distribution for Windows systems
- **Release notes**: Information about the build date, branch, and upstream source

## Accessing Builds

To download the latest builds:

1. Go to the **Releases** page of this repository
2. Find the release tagged `latest-{branch-name}`
3. Download the appropriate artifact for your platform

## Upstream Source

This fork automatically syncs from: **https://github.com/BieremaBoyzProgramming/bbpPairings**

## Conflict Handling

If the workflow encounters merge conflicts:
- The sync will fail gracefully
- A warning will be logged in the workflow run
- Manual intervention is required to resolve conflicts
- The workflow will retry on the next scheduled run (or manual trigger)

## Minimizing CI Usage

The workflow is designed to minimize GitHub Actions usage:
- Only runs bi-monthly (not weekly or daily)
- Skips building if no new commits are detected
- Fails fast if merge conflicts occur
- Uses efficient build strategies (static linking, single-stage builds)

## Customization

To modify the schedule:
1. Edit `.github/workflows/sync-and-build.yml`
2. Change the `cron` expression in the `schedule` section
   - Current: `0 0 1,15 * *` (1st and 15th at midnight UTC)
   - Monthly: `0 0 1 * *` (1st at midnight UTC)
   - Weekly: `0 0 * * 0` (Sundays at midnight UTC)

## Troubleshooting

### Workflow not running

- Check the **Actions** tab to ensure workflows are enabled
- Verify the repository has GitHub Actions enabled in settings

### Merge conflicts

- Review the failed workflow run for details
- Manually resolve conflicts in a local clone
- Push the resolved changes
- The next workflow run will detect the updated state

### Build failures

- Check the build logs in the failed workflow run
- Verify the Makefile and build dependencies
- Test the build locally to reproduce the issue

## License

This workflow configuration is provided as-is. The upstream source code and builds are subject to the license terms of the upstream repository.
