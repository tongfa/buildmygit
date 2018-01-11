# description

A bash script which detects changes in source controlled by git and then runs a build command.

# requirements

This script works with inotifywait or as a fallback fswatch.

## Debian / Ubuntu
`sudo apt-get install inotify-tools`

## centos
I think you will need to build inotify-tools from source.  good luck.
You'll also need a kernel that has inotify built in.
Might be easier to install fswatch?

## MacOS
`brew install fswatch`

`brew install git` if you do not already have it.

## other dependencies

This tool is not quite pure bash.  It also depends on `awk`, `grep`, `true`, `false` and `git`.  If you are already building your project under linux, odds are you'll have those dependencies already installed. 

# usage

## setup

you'll need to define at least on environment variable to specify the build command.  And then one more if you need to bump up the hysteresis time.

The `BMG_BUILD_CMD` command does receive one parameter, and that would be the directory that `buildmygit` is run on. 

I would recommend defining the build command like this:
```
build_my_stuff() {
  (cd angularSubFolder && ng build) && \
  drush cr && \
  sed -i.bumped '/^  version:/ s/$/0/' ${1}/MyDrupalTheme.libraries.yml
}
BMG_BUILD_CMD=build_my_stuff
```

The hysteresis is a time to wait for more file changes before starting a build.  For example this would allow a git checkout of a new branch that changes many files at once to only cause a single build.  It is defined as the input to the `sleep` command, which is seconds.  It's default is 0.1 (100 ms).  I imagine increasing it may be necessary if your project is complex.

There many ways to define these environment variables, I would suggest one of these two options:
1) in a local file in your git repo.  A toplevel file `.bmg` in your git repo will be sourced by this script on each invocation if it exists.
2) In environment variables defined in your `.bashrc` or `.profile`.

## running

Once everything is setup, it should be as simple as `buildmygit .`, where . specifies the folder you want to watch and build from.