# Feature: Download Specific Kernel Versions with Public Scores

## Summary

Add CLI support for downloading specific versions of Kaggle notebooks/kernels and displaying their associated public scores.

## Problem Statement

Currently, the `kaggle kernels pull` command only downloads the **latest version** of a kernel. Users cannot:
1. Download a specific historical version of a notebook
2. View the public score associated with any version
3. List available versions of a kernel with their scores

This limits users who want to:
- Reproduce results from a specific kernel version
- Compare different versions of their competition submissions
- Audit score progression across versions

## Current State

### What Exists
- `kaggle kernels pull <owner>/<kernel>` — downloads latest version only
- `kaggle kernels list --sort-by scoreDescending` — sorts by score, but score value not displayed
- Version number is returned when pushing (`ApiSaveKernelResponse.versionNumber`)
- Infrastructure for parsing `owner/kernel/version` format exists in `split_dataset_string()` but is unused

### Code References
- `kernels_pull()`: `src/kaggle/api/kaggle_api_extended.py:3137-3278`
- `ApiGetKernelRequest`: Only accepts `user_name` and `kernel_slug` (no version)
- Unused version parsing: `src/kaggle/api/kaggle_api_extended.py:2872` (`kernels_list_files`)

## Proposed Solution

### 1. Add `--version` parameter to `kaggle kernels pull`

```bash
# Download specific version
kaggle kernels pull owner/kernel-slug --version 5

# Alternative: version in slug format
kaggle kernels pull owner/kernel-slug/5
```

**Changes Required:**
- Extend `ApiGetKernelRequest` to include `version` field (backend API change)
- Update `kernels_pull()` to accept version parameter
- Update `kernels_pull_cli()` to parse `--version` flag
- Update CLI argument parser in `cli.py`

### 2. Add `kaggle kernels versions` command

```bash
# List all versions with scores
kaggle kernels versions owner/kernel-slug

# Example output:
# version  date        publicScore  privateScore  status
# 12       2024-01-15  0.85234      -             complete
# 11       2024-01-14  0.84123      -             complete
# 10       2024-01-13  0.83456      -             complete
```

**Changes Required:**
- New API endpoint or extend existing endpoint to return version history
- New `kernels_versions()` function in `kaggle_api_extended.py`
- New `kernels_versions_cli()` function
- New CLI subcommand in `cli.py`

### 3. Display score in `kaggle kernels list` output

```bash
kaggle kernels list --competition titanic --sort-by scoreDescending

# Current output fields: ref, title, author, lastRunTime, totalVotes
# Proposed output fields: ref, title, author, lastRunTime, totalVotes, bestPublicScore
```

**Changes Required:**
- Verify `ApiKernelMetadata` includes score field (may need backend change)
- Add score to `fields` list in `kernels_list_cli()` at line 2853

## API Changes Required (Backend)

### Option A: Extend existing endpoints
1. `ApiGetKernelRequest` — add optional `version` field
2. `ApiKernelMetadata` — ensure `bestPublicScore` field is exposed

### Option B: New endpoints
1. `GET /kernels/{owner}/{slug}/versions` — list all versions with metadata
2. `GET /kernels/{owner}/{slug}/versions/{version}` — get specific version

## CLI Changes (This Repo)

| File | Change |
|------|--------|
| `src/kaggle/cli.py` | Add `--version` arg to `kernels pull`, add `kernels versions` subcommand |
| `src/kaggle/api/kaggle_api_extended.py` | Update `kernels_pull()`, add `kernels_versions()` |
| `docs/kernels.md` | Document new functionality |

## Acceptance Criteria

- [ ] `kaggle kernels pull owner/slug --version N` downloads version N
- [ ] `kaggle kernels pull owner/slug/N` downloads version N (alternative syntax)
- [ ] `kaggle kernels versions owner/slug` lists all versions with scores
- [ ] `kaggle kernels list` displays public score when available
- [ ] Documentation updated
- [ ] Unit tests added

## Dependencies

- **Backend API team**: Must expose version parameter in get kernel endpoint
- **Backend API team**: Must expose score in kernel metadata response

## Priority

Medium — Quality of life improvement for competition participants

## Labels

`feature`, `cli`, `kernels`, `api-dependency`
