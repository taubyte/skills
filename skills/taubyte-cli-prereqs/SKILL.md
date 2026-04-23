---
name: taubyte-cli-prereqs
description: Hard preflight for tau/dream installation and version checks. Must run before any Taubyte workflow.
---

# Taubyte CLI Prerequisites (Hard Gate)

## Rule

Run this gate before any other Taubyte skill. If `tau` or `dream` is missing, install first. If install fails, stop immediately and do nothing else in Taubyte flow.

## Step 1: detect binaries and versions

```bash
command -v tau >/dev/null 2>&1 && tau version
command -v dream >/dev/null 2>&1 && (dream --version || dream --help)
```

## Step 2: install missing CLI(s)

Use `npm` as default cross-platform installer.

```bash
npm i -g @taubyte/cli
npm i -g @taubyte/dream
```

Alternative installer (when npm path is blocked):

```bash
curl https://get.tau.link/cli | sh
curl https://get.tau.link/dream | sh
```

## Step 3: verify install after changes

```bash
tau version
dream --version || dream --help
```

## Hard stop condition

- If either command still fails after install attempts, stop the workflow.
- Log the failure and attempted fixes in `.taubyte_ai/logs.txt`.
- Do not continue to cloud/project/resource operations.
