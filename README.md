# Courage Browser

A crypto & spyware free privacy-focused Webbrowser

**NOTE: THIS IS STILL VERY MUCH A WORK IN PROGRESS**

Courage is based on [brave](https://brave.com/), however in light of recent controversy we aim to provide users with a lighter, less invasive version of it. The original source-code is availabe [here](https://github.com/brave/brave-browser).

## Overview

This repository holds the build tools needed to build the Courage desktop browser for macOS, Windows, and Linux.  In particular, it fetches and syncs code from the projects we define in `package.json` and `src/courage/DEPS`:

  - [Chromium](https://chromium.googlesource.com/chromium/src.git)
    - Fetches code via `depot_tools`.
    - sets the branch for Chromium (ex: 65.0.3325.181).
  - [courage-core](https://github.com/courage-browser/courage-core)
    - Mounted at `src/courage`.
    - Maintains patches for 3rd party Chromium code.
  - [adblock-rust](https://github.com/brave/adblock-rust)
    - Linked through [brave/adblock-rust-ffi](https://github.com/brave/adblock-rust-ffi)

## Downloads

We don't offer any downloads at this moment

## Contributing

Please see the [contributing guidelines](./CONTRIBUTING.md)

## Install prerequisites

Follow the instructions for your platform:

- [macOS](https://github.com/courage-browser/courage-browser/wiki/macOS-Development-Environment)
- [Windows](https://github.com/courage-browser/courage-browser/wiki/Windows-Development-Environment)
- [Linux/Android](https://github.com/courage-browser/courage-browser/wiki/Linux-Development-Environment)

(These will likely be not 100% up to date for our fork)

## Clone and initialize the repo

Once you have the prerequisites installed, you can get the code and initialize the build environment.

```bash
git clone https://github.com/courage-browser/courage-browser.git
cd courage-browser
npm install

# this takes 30-45 minutes to run
# the Chromium source is downloaded which has a large history
npm run init
```
courage-core based android builds should use `npm run init -- --target_os=android --target_arch=arm` (or whatever cpu type you want to build for)

You can also set the target_os and target_arch for init and build using

```
npm config set target_os android
npm config set target_arch arm
```

## Build Courage
The default build type is component.

```
# start the component build compile
npm run build
```

To do a release build:

```
# start the release compile
npm run build Release
```

courage-core based android builds should use `npm run build -- --target_os=android --target_arch=arm` or set the npm config variables as specified above for `init`

### Build Configurations

Running a release build with `npm run build Release` can be very slow and use a lot of RAM especially on Linux with the Gold LLVM plugin.

To run a statically linked build (takes longer to build, but starts faster)

```bash
npm run build -- Static
```

To run a debug build (Component build with is_debug=true)

```bash
npm run build -- Debug
```

You may also want to try [[using sccache|sccache-for-faster-builds]].

## Run Courage
To start the build:

`npm start [Release|Component|Static|Debug]`

# Update Courage

`npm run sync -- [--force] [--init] [--create] [brave_core_ref]`

**This will attempt to stash your local changes in courage-core, but it's safer to commit local changes before running this**

`npm run sync` will (depending on the below flags):

1. 📥 Update sub-projects (chromium, courage-core) to latest commit of a git ref (e.g. tag or branch)
2. 🤕 Apply patches
3. 🔄 Update gclient DEPS dependencies
4. ⏩ Run hooks (e.g. to perform `npm install` on child projects)

| flag | Description |
|---|---|
|`[no flags]`|updates chromium if needed and re-applies patches. If the chromium version did not change it will only re-apply patches that have changed. Will update child dependencies **only if any project needed updating during this script run** <br> **Use this if you want the script to manage keeping you up to date instead of pulling or switching branch manually. **|
|`--create`|when used with `brave_core_ref` it will create a branch if one does not already exist|
|`--force`|updates both _Chromium_ and _courage-core_ to the latest remote commit for the current courage-core branch and the _Chromium_ ref specified in courage-browser/package.json (e.g. `master` or `74.0.0.103`). Will re-apply all patches. Will force update all child dependencies <br> **Use this if you're having trouble and want to force the branches back to a known state. **|
|`--init`|force update both _Chromium_ and _courage-core_ to the versions specified in courage-browser/package.json and force updates all dependent repos - same as `npm run init`|


Run `npm run sync brave_core_ref` to checkout the specified _courage-core_ ref and update all dependent repos including chromium if needed

### Scenarios

#### Create a new branch
```bash
courage-core> git checkout -b branch_name
```

or

```bash
courage-browser> npn run sync -- --create branch_name
```

### Checkout an existing branch or tag
```bash
courage-core> git fetch origin
courage-core> git checkout [-b] branch_name
courage-core> npm run sync
...Updating 2 patches...
...Updating child dependencies...
...Running hooks...
```

or

```bash
courage-browser> npn run sync --create branch_name
...Updating 2 patches...
...Updating child dependencies...
...Running hooks...
```

### Update the current branch to latest remote
```bash
courage-core> git pull
courage-core> npm run sync
...Updating 2 patches...
...Updating child dependencies...
...Running hooks...
```

#### Reset to latest courage-browser master, courage-core master and chromium
```bash
courage-browser> git checkout master
courage-browser> git pull
courage-browser> npm run sync -- --init
```

#### When you know that DEPS didn't change, but .patch files did (quickest)
```bash
courage-core> git checkout featureB
courage-core> git pull
courage-browser> npm run apply_patches
...Applying 2 patches...
```

# Troubleshooting

See [Troubleshooting](https://github.com/courage-browser/courage-browser/wiki/Troubleshooting) for solutions to common problems.
(This is likely out-of-date)