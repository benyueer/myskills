---
name: feature-flag-cleanup
description: Use when removing stale feature flags, cleaning up conditional code paths after a feature is fully rolled out, or reviewing flag lifecycle. Triggers on feature flag removal PRs, launchDarkly/cleanup branches, or code with dead flag branches.
---

# Feature Flag Cleanup

## Overview

Every feature flag has a cost: conditional branches, doubled test paths, and cognitive load. Flags that outlive their purpose become dead code that's harder to remove than regular dead code because it's wrapped in conditionals. This skill enforces a cleanup discipline.

## When to Use

- A feature flag has been at 100% rollout for 2+ weeks with no rollback needed
- Removing a flag-gated feature that was abandoned
- Auditing the flag inventory for stale entries
- Writing the cleanup PR after a successful feature launch

## When NOT to Use

- Flags that are actively being ramped (0% → 100%)
- Operational kill switches meant to be permanent (e.g., circuit breakers)
- Flags for features still in beta with active cohort testing

## Cleanup Workflow

### Step 1: Verify the Flag Is Safe to Remove

```bash
# Check rollout status in your flag system (LaunchDarkly, Unleash, etc.)
# Confirm: 100% enabled, no experiments active, no recent toggles

# Search for all references
grep -rn "FEATURE_FLAG_NAME" --include="*.ts" --include="*.tsx" --include="*.py"
```

If the flag appears in more than ~10 files, break the cleanup into multiple PRs.

### Step 2: Remove the Flag, Keep the New Code

```typescript
// BEFORE: conditional branch
if (featureFlags.isEnabled('new-checkout')) {
  return <NewCheckout />;
} else {
  return <LegacyCheckout />;
}

// AFTER: keep the new path, delete the old path
return <NewCheckout />;
```

Always keep the **enabled** code path. Delete the disabled path and the conditional.

### Step 3: Clean Up the Full Chain

| Layer | What to Remove |
|-------|---------------|
| Code | Flag checks, disabled code paths, flag-specific tests |
| Config | Flag definition in flag system, default values |
| Types | Flag name from feature flag enum/type union |
| Docs | References to the flag in runbooks, ADRs |
| Tests | Test cases for both flag-on and flag-off scenarios — replace with single test for the final behavior |

### Step 4: Verify No Orphaned References

```bash
# After removing, confirm no stale references remain
grep -rn "FEATURE_FLAG_NAME" .

# Run the full test suite — flag-off tests should now fail (expected)
# If they pass, the flag is still referenced somewhere
```

## Anti-Patterns

| Wrong | Right |
|-------|-------|
| Remove the flag check but leave both code paths | Delete the disabled path entirely |
| Comment out the old code | Delete it — git history remembers |
| Clean up code but leave flag in config | Remove from flag system too |
| One giant PR cleaning 15 flags | One PR per flag (easy to review, easy to revert) |
| `// TODO: remove this flag` for months | Set a calendar reminder or ticket at flag creation time |

## Flag Hygiene Rules

1. **Every flag gets a cleanup ticket** at creation time, assigned to the flag owner
2. **Flags older than 90 days** without a cleanup decision get flagged in sprint retro
3. **Flag-off tests are mandatory** during the flag's active period — they prove rollback works
4. **Naming convention**: `enable_<feature>` for temporary flags, `use_<variant>` for long-lived experiments

## Cleanup PR Template

```markdown
## Flag: FEATURE_FLAG_NAME

- **Created:** 2025-11-01
- **100% rollout since:** 2025-12-15
- **Rollback needed:** No (confirmed with PM)
- **Files changed:** X
- **Test impact:** Removed N flag-off test cases, kept M flag-on tests as canonical
```
