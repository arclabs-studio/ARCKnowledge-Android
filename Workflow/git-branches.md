# 🌳 Git Branch Naming Conventions

**Consistent branch naming enables automation, traceability, and clear communication. ARC Labs follows a Git Flow-inspired workflow.**

---

## Standard Format

```
<type>/<issue-id>-<short-description>
```

### Components

| Component | Required | Description | Example |
|-----------|----------|-------------|---------|
| Type | Yes | Category of work | `feature`, `bugfix` |
| Issue ID | Recommended | Linear issue reference | `ARC-123`, `FAVRES-45` |
| Description | Yes | Short kebab-case summary | `restaurant-search` |

### Branch Types

| Type | Purpose | Base Branch | Merges Into | Example |
|------|---------|-------------|-------------|---------|
| `feature` | New feature | `develop` | `develop` | `feature/ARC-123-restaurant-search` |
| `bugfix` | Bug fix | `develop` | `develop` | `bugfix/ARC-145-crash-on-empty-list` |
| `hotfix` | Critical production fix | `main` | `main` + `develop` | `hotfix/ARC-178-auth-vulnerability` |
| `release` | Release preparation | `develop` | `main` + `develop` | `release/1.2.0` |
| `spike` | Research/exploration | `develop` | (may not merge) | `spike/compose-animation-perf` |
| `docs` | Documentation only | `develop` | `develop` | `docs/update-architecture-guide` |

---

## Primary Branches

### `main`

The production branch. Always reflects the currently released version.

**Rules:**
- Protected: requires PR with approvals
- Only merged from `release/*` or `hotfix/*` branches
- Tagged with version numbers after each merge
- No direct pushes allowed
- No force pushes allowed
- All CI checks must pass

**Tags on main:**
```
v1.0.0
v1.1.0
v1.2.0
v1.2.1  (hotfix)
```

### `develop`

The integration branch. Contains the latest delivered development changes.

**Rules:**
- Protected: requires PR with approvals
- Feature and bugfix branches merge here
- Base for release branches
- Should always be in a buildable state
- No direct pushes allowed

---

## Branch Naming Guidelines

### Short Description Rules

- Use **kebab-case** (lowercase letters and hyphens)
- 3-5 words maximum
- Descriptive but concise
- Focus on WHAT the change does, not HOW

### Good Names

```
feature/ARC-123-restaurant-search
feature/ARC-124-user-profile-screen
feature/ARC-125-dark-theme-support
bugfix/ARC-145-crash-empty-list
bugfix/ARC-146-incorrect-price-format
hotfix/ARC-178-auth-token-refresh
hotfix/ARC-179-payment-validation
spike/compose-lazy-column-perf
spike/room-fts-search
docs/update-testing-guide
docs/add-architecture-diagrams
release/1.2.0
release/2.0.0-beta.1
```

### Bad Names

```
feature/my-branch                     # No issue ID, too vague
fix/stuff                             # Way too vague
feature/ARC-123                       # No description
feature/ARC-123-implement-the-new-restaurant-search-feature-with-filters  # Too long
Feature/ARC-123-Search                # Wrong case (uppercase)
feature/ARC_123_search                # Wrong separator (underscores)
feature/arc-123-search                # Wrong issue ID case
john/feature                          # Personal namespace (not allowed)
test-branch                           # No type prefix
```

### Naming Rules Summary

| Rule | Good | Bad |
|------|------|-----|
| Lowercase | `feature/arc-123-search` | `Feature/ARC-123-Search` |
| Kebab-case | `restaurant-search` | `restaurant_search` |
| Include issue ID | `ARC-123-search` | `search` |
| Short description | `restaurant-search` | `implement-restaurant-search-with-filters-and-sorting` |
| Type prefix | `feature/...` | `ARC-123-search` |

---

## Linear Integration

### Automatic Issue Linking

When branch names include a Linear issue ID, the following automation occurs:

1. **Branch appears in Linear issue** - The issue timeline shows the branch creation
2. **PRs auto-link** - Pull requests from the branch appear in the issue
3. **Status updates** - Issue status can auto-update based on PR status
4. **Traceability** - Full audit trail from issue to branch to PR to merge

### Issue Reference Format

- Use the full issue identifier: `ARC-123`, `FAVRES-45`
- Place it after the type prefix and before the description
- Use a hyphen to separate from description

```
feature/ARC-123-restaurant-search
         ^^^^^^^ ^^^^^^^^^^^^^^^^^
         Issue ID  Description
```

### Multiple Issues

If a branch addresses multiple issues, reference the primary issue in the branch name and mention others in commit messages or PR description:

```
# Branch name references primary issue
feature/ARC-123-restaurant-search

# Commit messages reference additional issues
feat(home): add search bar (relates to ARC-124)
```

---

## Branch Workflow

### Creating a Feature Branch

```bash
# Ensure develop is up to date
git checkout develop
git pull origin develop

# Create feature branch
git checkout -b feature/ARC-123-restaurant-search

# Verify branch name
git branch --show-current
# Output: feature/ARC-123-restaurant-search
```

### Working on a Branch

```bash
# Make changes and commit
git add src/main/java/com/arclabs/feature/home/
git commit -m "feat(home): add restaurant search bar"

# Continue working
git add src/test/java/com/arclabs/feature/home/
git commit -m "test(home): add search ViewModel tests"

# Keep in sync with develop (rebase preferred)
git fetch origin
git rebase origin/develop

# If there are conflicts
git rebase origin/develop
# Resolve conflicts in each file
git add <resolved-files>
git rebase --continue
```

### Pushing a Branch

```bash
# First push (set upstream)
git push -u origin feature/ARC-123-restaurant-search

# Subsequent pushes
git push

# After rebase (force push to YOUR branch only)
git push --force-with-lease
```

### Completing a Branch

```bash
# Ensure all changes are committed
git status

# Push final changes
git push

# Create pull request
gh pr create \
  --base develop \
  --title "feat(home): add restaurant search" \
  --body "## Summary
- Add search bar to home screen
- Implement debounced search with Flow
- Add unit tests for search ViewModel

## Testing
- [x] Unit tests pass
- [x] UI tested in light and dark theme

Closes ARC-123"
```

### After Merge

```bash
# Switch to develop
git checkout develop
git pull origin develop

# Delete local branch
git branch -d feature/ARC-123-restaurant-search

# Delete remote branch (usually automatic via GitHub)
git push origin --delete feature/ARC-123-restaurant-search
```

---

## Branch Protection Rules

### `main` Branch Protection

| Rule | Setting |
|------|---------|
| Require pull request | Yes |
| Required approvals | 1+ |
| Dismiss stale reviews | Yes |
| Require status checks | Yes |
| Required checks | `build`, `test`, `lint` |
| Require up-to-date branch | Yes |
| No direct pushes | Yes |
| No force pushes | Yes |
| No deletions | Yes |

### `develop` Branch Protection

| Rule | Setting |
|------|---------|
| Require pull request | Yes |
| Required approvals | 1+ |
| Require status checks | Yes |
| Required checks | `build`, `test`, `lint` |
| No direct pushes | Yes |
| No force pushes | Yes |

### Feature Branch Settings

| Rule | Setting |
|------|---------|
| No protection rules | Correct |
| Force push allowed | Yes (for rebase) |
| Delete after merge | Automatic |

---

## Pull Request Conventions

### PR Title Format

```
<type>(<scope>): <description>
```

The PR title should follow Conventional Commits format, matching the primary commit type:

### PR Title Examples

```
feat(home): add restaurant search with filters
fix(profile): resolve crash when avatar URL is null
refactor(data): extract user mapper to extension functions
test(domain): add GetRestaurantsUseCase edge cases
chore(deps): update Compose BOM to 2024.12.01
docs(readme): update installation instructions
build(gradle): enable configuration cache
ci(actions): add instrumented test workflow
```

### PR Description Template

```markdown
## Summary
Brief description of what this PR does and why.

## Changes
- Change 1: description
- Change 2: description
- Change 3: description

## Screenshots (if UI changes)
| Before | After |
|--------|-------|
| screenshot | screenshot |

## Testing
- [ ] Unit tests added/updated
- [ ] Instrumented tests added/updated (if applicable)
- [ ] UI tested in light theme
- [ ] UI tested in dark theme
- [ ] Accessibility verified (TalkBack)
- [ ] Tested on different screen sizes

## Architecture
- [ ] Follows Clean Architecture layers
- [ ] No business logic in Composables
- [ ] Dependencies injected via Hilt
- [ ] No force unwraps or force casts

## Related Issues
Closes ARC-123
Relates-to ARC-124
```

### PR Size Guidelines

| Size | Lines Changed | Review Time | Action |
|------|--------------|-------------|--------|
| Small | < 100 | 15 min | Merge quickly |
| Medium | 100-300 | 30 min | Standard review |
| Large | 300-500 | 1 hour | Consider splitting |
| Too Large | > 500 | > 1 hour | **Must split** |

---

## Common Workflows

### Feature Development Flow

```
                    PR
develop ──────────────────────────── develop
    \                              /
     feature/ARC-123-desc ────────
       commit 1
       commit 2
       commit 3
```

**Steps:**
1. Create branch from `develop`
2. Implement feature with atomic commits
3. Push and create PR to `develop`
4. Code review and approval
5. Merge (squash or merge commit)
6. Delete feature branch

### Bug Fix Flow

```
                    PR
develop ──────────────────────────── develop
    \                              /
     bugfix/ARC-145-desc ─────────
       test commit
       fix commit
```

**Steps:**
1. Create branch from `develop`
2. Write failing test that reproduces the bug
3. Implement the fix
4. Push and create PR
5. Review, merge, delete branch

### Hotfix Flow

```
                         PR              PR
main ──────────────────────── main ─────────
    \                        /           \
     hotfix/ARC-178-desc ──              merge to develop
       fix commit
       test commit
```

**Steps:**
1. Create branch from `main`
2. Implement minimal fix
3. Push and create PR to `main`
4. Review, merge, tag release
5. Also merge/cherry-pick to `develop`
6. Delete hotfix branch

### Release Flow

```
                              PR                PR
develop ──────────────────────── ──────── develop
    \                           \       /
     release/1.2.0 ──────────── main
       version bump              tag: v1.2.0
       changelog
       bug fixes only
```

**Steps:**
1. Create branch from `develop`
2. Bump version numbers
3. Update CHANGELOG.md
4. Fix release-blocking bugs only (no new features)
5. PR to `main`, merge, tag
6. PR to `develop`, merge
7. Delete release branch

---

## Best Practices

### DO

- Include the Linear issue ID in branch names
- Keep branches short-lived (less than 1 week ideally)
- Rebase on `develop` regularly to avoid large merge conflicts
- Delete branches after they are merged
- Use `--force-with-lease` instead of `--force` when force pushing
- Verify the branch name before first push
- Create a PR as early as possible (use draft PRs)

### DON'T

- Don't push directly to `main` or `develop`
- Don't create long-lived feature branches (more than 2 weeks)
- Don't reuse deleted branch names
- Don't force push to shared branches (`main`, `develop`)
- Don't merge `develop` into your feature branch (rebase instead)
- Don't leave stale branches around
- Don't use personal namespaces in branch names

---

## Troubleshooting

### Renaming a Branch

If you need to rename a branch (e.g., typo in name):

```bash
# Rename local branch
git branch -m old-name new-name

# Push new name
git push origin -u new-name

# Delete old remote branch
git push origin --delete old-name
```

### Recovering a Deleted Branch

If a branch was accidentally deleted:

```bash
# Find the commit hash from reflog
git reflog

# Recreate the branch
git checkout -b recovered-branch <commit-hash>
```

### Resolving Rebase Conflicts

```bash
# Start rebase
git rebase origin/develop

# If conflicts occur
# 1. Open conflicting files
# 2. Resolve conflicts (keep both changes or choose one)
# 3. Stage resolved files
git add <resolved-files>

# 4. Continue rebase
git rebase --continue

# If rebase becomes too complex, abort and try merge instead
git rebase --abort
git merge origin/develop
```

### Branch Diverged from Remote

```bash
# Your local branch has diverged from the remote
# Option 1: Rebase (preferred for feature branches)
git pull --rebase origin feature/ARC-123-desc

# Option 2: Force update local to match remote
git fetch origin
git reset --hard origin/feature/ARC-123-desc
# WARNING: This discards local changes
```

### Finding Which Branch a Commit Is On

```bash
# Find branches containing a specific commit
git branch --contains <commit-hash>

# Find branches containing a commit (including remote)
git branch -a --contains <commit-hash>
```

---

## Branch Naming Automation

### Git Alias for Branch Creation

Add to your `~/.gitconfig`:

```ini
[alias]
    feature = "!f() { git checkout develop && git pull && git checkout -b feature/$1; }; f"
    bugfix = "!f() { git checkout develop && git pull && git checkout -b bugfix/$1; }; f"
    hotfix = "!f() { git checkout main && git pull && git checkout -b hotfix/$1; }; f"
```

**Usage:**
```bash
git feature ARC-123-restaurant-search
git bugfix ARC-145-crash-empty-list
git hotfix ARC-178-auth-vulnerability
```

### Branch Name Validation Hook

```bash
#!/bin/bash
# .git/hooks/pre-push
# Validates branch names before pushing

branch=$(git branch --show-current)
pattern="^(feature|bugfix|hotfix|release|spike|docs)/[A-Z]+-[0-9]+-[a-z0-9-]+$|^(release)/[0-9]+\.[0-9]+\.[0-9]+(-[a-z]+\.[0-9]+)?$|^(spike|docs)/[a-z0-9-]+$|^(main|develop)$"

if ! echo "$branch" | grep -qE "$pattern"; then
    echo "ERROR: Branch name '$branch' does not follow naming conventions."
    echo ""
    echo "Expected format: <type>/<issue-id>-<description>"
    echo "  feature/ARC-123-restaurant-search"
    echo "  bugfix/ARC-145-crash-empty-list"
    echo "  hotfix/ARC-178-auth-vulnerability"
    echo "  release/1.2.0"
    echo "  spike/compose-animation-perf"
    echo "  docs/update-testing-guide"
    echo ""
    exit 1
fi
```

---

## Further Reading

- [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/)
- [GitHub Flow](https://guides.github.com/introduction/flow/)
- [Git Commits](./git-commits.md)
- [Plan Mode](./plan-mode.md)

---

**Remember**: Branch names are communication. Make them clear, consistent, and traceable.
