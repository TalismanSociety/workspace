# Talisman Workspace 🔨

<img src="1f9ff.svg" alt="Talisman" width="15%" align="right" />

[![discord-link](https://img.shields.io/discord/858891448271634473?logo=discord&logoColor=white&style=flat-square)](https://discord.gg/rQgTD9SGtU)

**A setup for building on multiple @talismn projects in parallel.**

While working on Talisman, you may find yourself needing to coordinate your code changes across multiple repositories.

For example, you might build a new endpoint for [@talismn/api](https://github.com/talismansociety/api) along with its react interface in [@talismn/api-react-hooks](https://github.com/talismansociety/api-react-hooks).

This repo's goal is to make that experience as uncomplicated as possible.

## Usage

#### Clone and initiailze your talisman workspace

```bash
git clone git@github.com:TalismanSociety/workspace.git
cd workspace
bash init
```

You should now find a bunch of talisman repositories have been checked out locally:

```bash
.
├── api
│   └── package.json
├── api-react-hooks
│   └── package.json
├── chaindata-js
│   └── package.json
├── util
│   └── package.json
└── web
    └── package.json
```

> Repeated calls to `bash init` are safe; existing directories will be ignored, missing directories will be created.

#### Link all workspace repositories to eachother

> NOTE: Please don't publish your linked repositories to git/npm!  
> Instead, run `bash unlink` in your workspace before you run any `git add` / `git commit` / `yarn publish` commands.  
> We are investigating solutions to make this process less prone to human error 💀

```bash
bash link
```

Your repositories should now be linked.

#### Unlink repositories (set dependencies back to published NPM versions)

```bash
bash unlink
```

## Adding new workspace repositories

The process here is admittedly quite manual and error-prone (though it is far better than not having the talisman-workspace at all!).  
In the future we might be able to improve it by utilising `yarn workspaces` or perhaps `lerna` for workspace management.  
But for now, this is the process:

First, **make sure your repo is using yarn v3**.  
[Here is an example commit](https://github.com/TalismanSociety/api/commit/8d2d092e77d4fe45fae758d4403b143ac4ec0da2) where `@talismn/api` was upgraded from yarn v1 to v3.  
You can copy the `.gitignore`, `.yarn/releases/yarn-3.0.1.cjs` and `.yarnrc.yml` files straight from that repo.

Next, add a new line for your new repository to the `init` command so that future you and other talisman developers can clone it to their workspace with `bash init`:

```bash
# replace my-new-repo with your repository's name
#
# echo is not necessary, feel free to just edit the text file `init` with a text editor of your choice

echo '[[ -d "my-new-repo" ]] || git clone git@github.com:TalismanSociety/my-new-repo.git' >> init
```

Finally, add a new section to the `link` and `unlink` commands to link your repo to its talisman dependents.

For our `my-new-repo` example, these changes would look like:

```diff
diff --git a/init b/init
index cd551d4..9652b69 100755
--- a/init
+++ b/init
@@ -6,5 +6,6 @@ set -xeuo pipefail
 [[ -d "api" ]] || git clone git@github.com:TalismanSociety/api.git
 [[ -d "api-react-hooks" ]] || git clone git@github.com:TalismanSociety/api-react-hooks.git
 [[ -d "chaindata-js" ]] || git clone git@github.com:TalismanSociety/chaindata-js.git
+[[ -d "my-new-repo" ]] || git clone git@github.com:TalismanSociety/my-new-repo.git
 [[ -d "util" ]] || git clone git@github.com:TalismanSociety/util.git
 [[ -d "web" ]] || git clone git@github.com:TalismanSociety/talisman-web.git web
diff --git a/link b/link
index 680a5ff..dc02444 100755
--- a/link
+++ b/link
@@ -20,6 +20,13 @@ rm -rf dist
 yarn build:types
 cd ..

+cd my-new-repo
+yarn
+yarn link -Ar ../chaindata-js # my-new-repo has chaindata-js in its package.json
+rm -rf dist
+yarn build:types
+cd ..
+
 cd api-react-hooks
 yarn
 yarn link -Ar ../chaindata-js
@@ -32,6 +39,7 @@ cd ..
 cd web
 yarn
 yarn link -Ar ../api-react-hooks
+yarn link -Ar ../my-new-repo # talismn-web has my-new-repo in its package.json
 yarn link -Ar ../util
 cd ..

diff --git a/unlink b/unlink
index dc515e4..1656a4b 100755
--- a/unlink
+++ b/unlink
@@ -15,6 +15,10 @@ cd api
 yarn unlink -A
 cd ..

+cd my-new-repo
+yarn unlink -A
+cd ..
+
 cd api-react-hooks
 yarn unlink -A
 cd ..
```

If you have issues getting this to work, feel free to ping alec on Discord.

## FAQ

**Why are my changes not being included/hot loaded by webpack?**

When developing with an unpublished (and not bundled/transpiled!) dependency, webpack needs to be told to include the dependency in `babel-loader`.  
To do this, try running `@talismn/web` with `TALISMAN_LIBS_FAST_REFRESH=true yarn start`.

If your changes are still not being hot loaded, you might have a built copy of your dependency sitting around, for example at `api/dist/index.js`.  
Files inside `dist` take priority over those in `src`, so you will need to delete the built copy to continue.

To do this, you can either:

1. Manually remove the offending directory: `cd api; rm -rf dist; yarn build:types`, or:
2. Re-run the link process: `bash link` (automatically removes `dist` directories and rebuilds typedefs)

**Why does my code editor not show type hints in development?**

The `bash link` script builds typedefs for all repos once, and only once.  
After making code changes, you need to rebuild the types (but only the types!) for each dependency yourself.

```bash
cd api
yarn build:types

# code editor type hints will again work as expected
```

Alternatively, you can safely re-run `bash link` and the types will be rebuilt.
