---
title: ~/Brewfile
date: 2021-12-07T14:36:38+01:00
summary: Brewfile is literally a package.json for macos `brew`. Every installed software on your system is defined in the Brewfile, in a **declarative** way.
cover:
  image: imgs/cover.png
slug: brewfile
tags:
  - macos
  - osx
  - brew
  - brewfile
---

# Brewfile

Brewfile is a feature of `brew`, also referred to as `brew bundle`.

With brewfile you can install in a **declarative** way stuff like:
- normal packages (like as `brew install package`)
- taps (like as `brew tap homebrew/cask`)
- casks (like as `brew install --cask package`)
- mas (Mac App Store packages, like as `mas install package`)
- whalebrew (like as `whalebrew install repo/docker-image`)

## Usage

### From scratch

Create a file called `Brewfile` whenever you want (for example in home directory) and type `brew bundle` in a terminal to let `brew` read, update and install everything.

### From existing system

If you have already packages installed and you don't want to write a `Brewfile` from scratch, you can dump your packages with `brew bundle dump`.

### Force cleanup

If you want your `Brewfile` to match 1:1 with your system, you can use `brew bundle cleanup --force` to remove all packages that are not in your `Brewfile`.

### Example

Example of a Brewfile, taken from the [Official Github repository](https://github.com/Homebrew/homebrew-bundle):

```ruby
# 'brew tap'
tap "homebrew/cask"
# 'brew tap' with custom Git URL
tap "user/tap-repo", "https://user@bitbucket.org/user/homebrew-tap-repo.git"
# set arguments for all 'brew install --cask' commands
cask_args appdir: "~/Applications", require_sha: true

# 'brew install'
brew "imagemagick"
# 'brew install --with-rmtp', 'brew services restart' on version changes
brew "denji/nginx/nginx-full", args: ["with-rmtp"], restart_service: :changed
# 'brew install', always 'brew services restart', 'brew link', 'brew unlink mysql' (if it is installed)
brew "mysql@5.6", restart_service: true, link: true, conflicts_with: ["mysql"]

# 'brew install --cask'
cask "google-chrome"
# 'brew install --cask --appdir=~/my-apps/Applications'
cask "firefox", args: { appdir: "~/my-apps/Applications" }
# always upgrade auto-updated or unversioned cask to latest version even if already installed
cask "opera", greedy: true
# 'brew install --cask' only if '/usr/libexec/java_home --failfast' fails
cask "java" unless system "/usr/libexec/java_home --failfast"

# 'mas install'
mas "1Password", id: 443987910

# 'whalebrew install'
whalebrew "whalebrew/wget"
```
