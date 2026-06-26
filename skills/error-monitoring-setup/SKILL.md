---
name: error-monitoring-setup
description: Use when adding or configuring error tracking (Sentry, Bugsnag, Datadog, Rollbar) in an application, including source map uploads, environment tagging, release tracking, and alert routing. Triggers on Sentry.init, errorHandler setup, or monitoring integration PRs.
---

# Error Monitoring Setup

## Overview

An error tracker that's misconfigured is worse than none — it creates false confidence. This skill ensures monitoring is set up with proper environment separation, source maps, release tracking, and actionable alert rules.

## When to Use

- Integrating Sentry, Bugsnag, Datadog, or Rollbar for the first time
- Configuring source map uploads in CI/CD
- Setting up alert rules and on-call routing
- Debugging "Unknown Error" or minified stack traces in production

## When NOT to Use

- Application-level logging (use structured logging instead)
- Infrastructure monitoring (CPU, memory, disk — use Prometheus/Grafana)
- Business metrics (conversion rates, funnel dropoff — use product analytics)

## Core Rules

### 1. Environment Must Be Explicit

```javascript
Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV, // NEVER default to "production"
  tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
});
```

Never let dev/staging errors pollute the production error stream.

### 2. Attach Release on Every Deploy

```javascript
Sentry.init({
  release: process.env.CI_COMMIT_SHA, // Git SHA, not "v1.0"
});
```

Without release tracking, you can't correlate errors to deploys or know if a fix shipped.

### 3. Source Maps Must Be Uploaded in CI, Not Bundled

```yaml
# CI pipeline — after build, before deploy
- name: Upload source maps
  run: |
    npx @sentry/cli releases files "$COMMIT_SHA" \
      upload-sourcemaps ./dist --url-prefix "~/static/js"
```

Bundled source maps leak your source code to the public. Upload to the error tracker, then delete from the build artifact.

### 4. Filter Noisy Errors Before They Reach the Tracker

```javascript
Sentry.init({
  beforeSend(event, hint) {
    // Don't report browser extension errors
    if (event.exception?.values?.[0]?.stacktrace?.frames?.some(
      f => f.filename?.includes('chrome-extension://')
    )) return null;

    // Don't report expected validation errors
    if (hint?.originalException?.code === 'VALIDATION_ERROR') return null;

    return event;
  },
});
```

### 5. Tag Errors With Business Context

```javascript
Sentry.setUser({ id: user.id, email: user.email });
Sentry.setTags({
  tenant: tenant.slug,
  feature_flag: currentExperiment,
  api_version: req.headers['x-api-version'],
});
```

Tags make errors searchable and help identify blast radius during incidents.

## Alert Routing Template

| Severity | Condition | Route | Response Time |
|----------|-----------|-------|---------------|
| P0 | New error, >100 occurrences/hr | #incidents + on-call pager | 15 min |
| P1 | Error spike (>3x baseline) | #eng-alerts | 1 hour |
| P2 | New error, <10 occurrences/hr | Weekly digest | Next sprint |
| P3 | Known error, no user impact | Suppressed | Backlog |

## Common Mistakes

| Wrong | Right |
|-------|-------|
| No `environment` set | Always set explicitly from env var |
| Source maps bundled in dist | Upload in CI, then delete |
| Reporting every 4xx | Filter 400/401/404 — they're not bugs |
| No release tracking | Tag every deploy with git SHA |
| Alert on every error | Threshold-based alerts with dedup |
