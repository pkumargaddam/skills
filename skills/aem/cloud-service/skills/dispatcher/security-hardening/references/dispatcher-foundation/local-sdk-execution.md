# Local Dispatcher SDK Execution (Cloud)

Use this reference when cloud dispatcher work requires a local SDK-backed runtime, not just static validation.

This file documents the launcher contract exposed by the shipped Dispatcher SDK scripts. Treat the help output from the installed SDK package as the final authority for your exact SDK version.

## When To Use This

Use local SDK execution when you need one of these:

- container-backed runtime verification for rewrites, filters, vhosts, cache behavior, or probe-path handling
- local `test` mode to confirm the processed config and basic HTTPD/dispatcher validity
- hot-reload feedback while iterating on dispatcher config changes

Do not confuse this with deployable cloud config content. These commands are local runtime helpers.

## Positional Contract

The shipped launcher usage is:

```bash
./bin/docker_run.sh <deployment-folder> <aem-host>:<aem-port> <local-port>
./bin/docker_run.sh <deployment-folder> <aem-host>:<aem-port> test
```

Common examples:

```bash
./bin/docker_run.sh out <aem-host>:4503 8080
DISP_RUN_MODE=stage ./bin/docker_run.sh out <aem-host>:4503 8080
DISP_LOG_LEVEL=trace1 ./bin/docker_run.sh out <aem-host>:4503 8080
REWRITE_LOG_LEVEL=trace2 ./bin/docker_run.sh out <aem-host>:4503 8080
./bin/docker_run.sh out <aem-host>:4503 test
```

Meaning of the positional arguments:

- `<deployment-folder>`: prepared SDK deployment directory, often `out`
- `<aem-host>:<aem-port>`: backend AEM endpoint the local Dispatcher runtime should talk to
- `<local-port>`: host port that exposes the local Dispatcher runtime
- `test`: run config test mode instead of exposing a live local port

## Hot Reload Launcher

Some shipped SDK packages also include:

```bash
./bin/docker_run_hot_reload.sh <deployment-folder> <aem-host>:<aem-port> <local-port>
```

In shipped packages, this launcher follows the same positional contract as `docker_run.sh` and enables hot-reload semantics for the deployment folder. If your installed SDK version prints different help text, prefer the installed script output.

## Runtime Environment Variables

The shipped launcher help exposes these environment variables:

### `DISP_RUN_MODE`

Defines the simulated environment type for the local run.

Valid values:

- `dev`
- `stage`
- `prod`

Default is `dev`.

### `DISP_LOG_LEVEL`

Sets dispatcher log verbosity.

Valid values:

- `trace1`
- `debug`
- `info`
- `warn`
- `error`

Default is `warn`.

Use `trace1` when you need backend request flow or dispatcher decision detail.

### `REWRITE_LOG_LEVEL`

Sets rewrite-engine log verbosity.

Valid values:

- `trace1` through `trace8`
- `debug`
- `info`
- `warn`
- `error`

Default is `warn`.

Use this when debugging `RewriteRule` and `RewriteCond` behavior.

### `ENV_FILE`

Imports variables from a file before startup.

Use this when local SDK execution depends on variables that would otherwise be exported manually.

### `HOT_RELOAD`

Enables config reload when watched files change.

Valid values:

- `true`
- `false`

Default is `false`.

Use this for local iteration loops on rewrites, filters, and vhosts.

### `ALLOW_CACHE_INVALIDATION_GLOBALLY`

Overwrites the default invalidation behavior to allow all connections for cache invalidation.

Valid values:

- `true`
- `false`

Default is `false`.

Treat this as a local test convenience only. Do not carry this posture into production guidance.

### `HTTPD_DUMP_VHOSTS`

Enables vhost dump output for debugging.

Valid values:

- `true`
- `false`

Default is `false`.

### `ENABLE_MANAGED_REWRITE_MAPS_FLAG`

Enables managed rewrite maps.

Valid values:

- `true`
- `false`

Default is `true`.

This matters when local behavior depends on rewrite-map-backed redirects or readiness gating.

### `MANAGED_REWRITE_MAPS_PROBE_CHECK_SKIP`

Skips probe checks for managed rewrite maps.

Valid values:

- `true`
- `false`

Default is `false`.

Only use this when intentionally isolating local rewrite-map behavior and call out that the check was skipped.

## Host-Side Compatibility Variable

You may also see:

```bash
export DOCKER_API_VERSION=1.43
```

This is a host Docker-client compatibility override, not a dispatcher runtime setting. Use it only when the installed SDK scripts or local Docker environment require it.

## Recommended Execution Patterns

### 1. Static Config Test

Use this first when the goal is syntax or processed-config validation:

```bash
./bin/docker_run.sh out <aem-host>:<aem-port> test
```

Use this before claiming runtime behavior is verified.

### 2. Live Local Dispatcher Port

Use this when you need request/response verification:

```bash
./bin/docker_run.sh out <aem-host>:<aem-port> 8080
```

Then verify behavior with representative requests against the local port.

### 3. Hot Reload Iteration

Use this when tuning rewrites, filters, or vhosts repeatedly:

```bash
./bin/docker_run_hot_reload.sh out <aem-host>:<aem-port> 8080
```

If your SDK package does not include this launcher, use `HOT_RELOAD=true` with the standard launcher when supported by that package version.

## Skill Usage Guidance

When using local SDK execution in cloud dispatcher skills:

- state whether evidence came from static validation, local SDK runtime, or MCP runtime tools
- record the exact launcher mode used: live port, hot reload, or `test`
- record any non-default env vars that materially changed behavior
- call out when managed rewrite map checks, invalidation safety, or run-mode simulation could affect the conclusion
- do not treat local SDK behavior as stronger evidence than executed MCP/runtime evidence against the actual target environment
