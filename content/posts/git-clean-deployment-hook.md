---
title: "Using git clean in a deployment hook"
date: 2024-04-09
---

A typical post-receive hook for updating a target directory based on the most recent commit might look like this:

```sh
#!/bin/bash

export GIT_WORK_TREE="../myproject"
git checkout -f master
```

However, this does not remove any untracked files or directories in `../myproject`.  When files are removed or renamed in later commits, their remnants will remain untouched in the deployment directory.  How can we get rid of them?

Enter `git clean`!  Remove all untracked files and directories like so.

```sh
git clean -fd
# -f forces removal of files without prompting
#    or disabling config variable `clean.requireForce`
# -d recurses into untracked directories
```

But what about all of those extra runtime files like databases and logs---how do we avoid deleting them?  The strategy taken here is to add these files to `.gitignore`.  `git clean` ignores these files by default.

The other tricky point is that `git checkout` should be run before `git clean` in order to ensure an up-to-date `.gitignore`.  Otherwise, runtime files we later remember to add to `.gitignore` may be deleted before `.gitignore` gets updated.

Here's what the final hook looks like:

```sh
#!/bin/bash

# Enable immediate exit on error
set -e

# Set git directories
export GIT_WORK_TREE="../myproject"

# Checkout all files in the target directory to match the repository
# Must update .gitignore before running `git clean`
git -c advice.detachedHead=false checkout -f refs/heads/master

# Clean untracked files and directories in the target directory
# Do not remove files which are listed within .gitignore
git clean -fd
```
