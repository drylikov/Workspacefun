# Workspace

## Issue 1 — Add workspace without `workspace:`

```bash
$ bun run clean
$ bun install
$ cd packages/a
$ bun add b # installs from npm
```

For full npm compatibility, this should read the `"version"` from `packages/b/package.json` and add this to `packages/a/package.json` as a dependency.

```json
{
  "dependencies": {
    "b": "^<version>"
  }
}
```

Both of these work (install the workspace).

```sh
$ bun add workspace:b
$ bun add b@1.0.0 # works if version of workspace/b is in range
```

It seems to me that basically any version of `bun add (workspace:)?b@<version>` should install the workspace here. If the version specifier is not compatible with the current version of the workspace being installed, it should fail. This may not be the same as npm/yarn (not sure).

## Issue 2 - Idempotency issue

```sh
$ bun run clean
$ bun install
$ cd packages/a
$ bun add workspace:b # works
bun add v0.7.2 (7a8f57c4)
 + c@workspace:packages/c

 installed b@workspace:packages/b


 2 packages installed [4.00ms]
$ bun add workspace:b
bun add v0.7.2 (7a8f57c4)
 + c@workspace:packages/c

 installed b@workspace:packages/b


 2 packages installed [10.00ms]
  Removed: 2 # <- this line gets printed every other run
```

## Issue 3 - Don't add `workspace:` in version specifier in package.json

```sh
$ bun run clean
$ bun install
$ cd packages/a
$ bun add workspace:b # works
```

This adds the following line to dependencies:

```
"b": "workspace:packages/b"
```

I think this should add a version range and (possibly) omit the `workspace:` scope from the version specifier. Read the current `"version"` from `packages/b/package.json` and add this to `packages/a/package.json` as a dependency.

```json
{
  "dependencies": {
    "b": "^<version>"
  }
}
```

## Issue 4 — Add, then remove, then add again

```sh
$ bun run clean
$ bun install
$ cd packages/a
$ bun add workspace:b # works
$ bun remove b # works
$ bun add workspace:b
bun add v0.7.1 (53cc4df1)


error: workspace dependency "workspace:b" not found

Searched in "./b"

Workspace documentation: https://bun.sh/docs/install/workspaces
```

## Issue 5 - Add workspace dependency without running `bun install` in root first

Currently none of these work without running `bun install` in the root first. Perhaps this is intentional to avoid scanning up the file system to check if the current project is a workspace? If not, I think any `bun add/install/remove` command inside a workspace should check if the current project is a workspace and hoist appropriately if so.
