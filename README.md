# packcheck

[![Hackage](https://img.shields.io/hackage/v/packcheck.svg?style=flat)](https://hackage.haskell.org/package/packcheck)
[![Gitter chat](https://badges.gitter.im/composewell/gitter.svg)](https://gitter.im/composewell/packcheck)
[![Build Status](https://travis-ci.org/composewell/packcheck.svg?branch=master)](https://travis-ci.org/composewell/packcheck)
[![Windows Build status](https://ci.appveyor.com/api/projects/status/f7c0ncy84cxp8lbe?svg=true)](https://ci.appveyor.com/project/harendra-kumar/packcheck)
[![CircleCI](https://circleci.com/gh/composewell/packcheck/tree/master.svg?style=svg)](https://circleci.com/gh/composewell/packcheck/tree/master)
[![Coverage Status](https://coveralls.io/repos/composewell/packcheck/badge.svg?branch=master&service=github)](https://coveralls.io/github/composewell/packcheck?branch=master)

## Quick Start

### CI (Travis/Appveyor/CircleCI)
To use packcheck for CI testing of your repo:

* Add your package repo to travis/appveyor/circleci
* Copy
[.travis.yml](https://github.com/composewell/packcheck/blob/master/.travis.yml),
[appveyor.yml](https://github.com/composewell/packcheck/blob/master/appveyor.yml),
or
[.circleci/config.yml](https://github.com/composewell/packcheck/blob/master/.circleci/config.yml)
to your package repo 

CI should work out of the box for most packages. Uncomment the relevant lines
in the CI config files or change the values of the environment variables for
fine grained control or custom configuration.

### Local Machine
You can use packcheck to build or CI test a package on your local machine as
well.  For local use, copy
[packcheck.sh](https://github.com/composewell/packcheck/blob/master/packcheck.sh)
to your local machine (Linux/OSX/Windows), put it in your PATH, and run it
from your package directory. You can pass the same evironment variables that
are used in CI files to run the exact same tests locally. Usage is as simple
as:
```
$ packcheck.sh cabal-v2
$ packcheck.sh cabal-v2 GHCVER=8.6.5
$ packcheck.sh cabal-v2 ENABLE_GHCJS=y
$ packcheck.sh cabal-v1
$ packcheck.sh stack GHCVER=8.6
```

`packcheck` can automatically pick the requested version of GHC from:

* multiple GHC path components in your PATH environment variable
* [hvr ghc PPA](https://launchpad.net/~hvr/+archive/ubuntu/ghc) install directory
* stack installed ghc binaries

### Out of the box support

| Platforms     | Build Types     | CI Modes      | Compilers |
|:-------------:|:---------------:|:-------------:|:---------:|
| Linux         | stack           | Travis        | GHC       |
| OSX           | cabal           | Appveyor      | GHCJS     |
| Windows       | cabal new-build | CircleCI      |           |
|               |                 | Local Machine |           |

The script can be easily adapted to any CI with a single line build command.

## Key Features

* _Error messages:_ A lot of emphasis has been put on providing precise and
  detailed error messages when something fails so that the user can easily fix
  things.
* _Informational:_ The output provides all the information that you may want to
  know, tool paths being used, their versions, how they are invoked, build
  options, time taken by each build step etc. You can even copy the commands
  from the output and paste them on your local host to reproduce the build or
  failure and debug quickly. [See here for a sample
  output](https://travis-ci.org/composewell/packcheck).
* _Same tests everywhere:_ You can run exact same tests with same options or
  flags, in the same way, on all CI platforms.
* _Choose options:_ Conveniently control all aspects of build through command
  line or environment variables, including tool options or whether to enable
  benchmarks, haddock, coverage, install test etc. 
* _Picking GHC:_ Right GHC is picked up automatically from PATH or TOOLS_DIR
  (`hvr ghc PPA` installation dir) based on GHCVER. Stack installed GHC
  binaries can be picked automatically when available.
* _Test source distribution:_ `packcheck` creates the source distribution and
  builds the package from the generated tarball to make sure that you build
  what you release and don't miss adding a file to the distribution.
* _Upload coverage:_ To send coverage info to
  [coveralls.io](https://coveralls.io) just uncomment a line in your
  `.travis.yml`.
* _Non-destructive_: By default the script does not change any config or
  upgrade any tools on the host machine.
* _Auto tool install_: For stack builds, `stack` and `ghc` can be installed
  automatically

## Introduction

The package `packcheck` includes a script called `packcheck.sh`, it is a high
level universal super build script to uniformly, consistently build and
comprehensively sanity test a Haskell package across build tools (stack/cabal)
and across all platforms (Linux/MacOS/Windows).  You do not need to be familiar
with any of the build tools to use it.

To make sure that it works everywhere without installing anything it is
deliberately written using the `bash` shell scripting language. Any of the
parameters to control the builds can either be passed on the script command
line or as environment variables for convenient use on CI systems.

`packcheck` is also a minimal yet complete "hello world" Haskell package with
model `travis` and `appveyor` config files that can be used unmodified in any
Haskell package. The CI configs can be modified **declaratively**, using
environment variables, to adapt to **any** kind of build scenario you can
imagine.

This model package has everything that a Haskell package usually has; including
tests, benchmarks and Linux/MacOS/Windows CI already working. It can be used as
a starting point to develop a new package. Beginners can use it to learn about
haskell package metadata structure.

## What all does it do?

An invocation of `packcheck.sh` performs a whole battery of tests, all aspects
can be controlled via environment variables, command line. The flow goes
roughly as follows:

* Pick up the correct version of GHC/cabal/stack
* create source distribution and unpack it to test from it
* build source, benchmarks and docs
* run tests
* run `hlint`
* generate and upload coverage report (to coveralls.io)
* perform distribution checks

## Usage Examples

You can run these commands on your local machine as well as inside a CI script.
You can try these commands in the `packcheck` package itself:
```
$ cd packcheck
$ ./packcheck.sh cabal-v2 GHCVER=8.6.5
$ ./packcheck.sh cabal-v1 GHCVER=7.10.3 CABALVER=1.22
```

```
$ ./packcheck.sh stack RESOLVER=lts-13
$ ./packcheck.sh stack GHCVER=8.6.5
$ ./packcheck.sh stack RESOLVER=lts-7.24 STACK_YAML=stack-8.0.yaml STACK_BUILD_OPTIONS="--flag streamly:examples-sdl" CABALVER=1.24
# You can also do a cabal build using stack installed ghc:
$ stack exec ./packcheck.sh cabal RESOLVER=lts-11
```

Run hlint commands on the directories `src` and `test`:
```
$ ./packcheck.sh stack HLINT_COMMANDS="hlint lint src; hlint lint test"
```

Send coverage info of the testsuites named `test1` and `test2` to coveralls.io
using `hpc-coveralls`.
```
$ ./packcheck.sh cabal-v1 GHCVER=8.0.2 COVERALLS_OPTIONS="test1 test2"
```

## Picking GHC versions

When `GHCVER` parameter is not specified, `packcheck` looks for a binary named
`ghc` in your `PATH` environment variable. It uses first such binary found in
`PATH`.

When `GHCVER` parameter is specified, it looks for `ghc` in the `PATH` and if
`GHCVER` is a PREFIX of the actual version of `ghc` binary found then that
`ghc` binary is used. Otherwise, `packcheck` tries to look for another `ghc`
binary in the next PATH components until it finds a matching `ghc` version.

When both `GHCVER` and `TOOLS_DIR` are specified then in addition to searching
in `PATH` environment variable, `packcheck` also looks for ghc in
`${TOOLS_DIR}/ghc/${GHCVER}*/bin`. This is to facilitate selecting any GHC
version from an `hvr/ghc` ubuntu PPA installation without putting all the
myriad GHC version directories explicitly in your `PATH`.

If all of the above fails `packcheck` looks for ghc in the `stack` install
locations.

## packcheck-safe

`packcheck-safe.sh` is a more robust wrapper over `packcheck.sh`, it does not
trust or use any environment variables, all environment needs to be specified
explicitly on the command line. Therefore, it ensures better reproducibility.

It also catches any misspelled command line parameter names. For example,
`packcheck.sh` won't catch it if you typed `GHCVWR=8.4` instead of
`GHCVER=8.4`, it just assumes that `GHCVER` is not specified.
`packcheck-safe.sh` would generate an error saying that `GHCVWR` is not
recognized. Since it uses a clean environment you will have to specify PATH as
well on the command line. For example,

```
$ ./packcheck-safe.sh cabal-v2 PATH=/bin:/usr/bin:/opt/ghc/bin
```

## Full Reference

NOTE: Any of the parameters described below can either be passed on command
line or as an environment variable. Passing options on command line is more
convenient when running interactively, while environment variables are more
convenient when running on a CI system.

```
$ packcheck.sh --help

--------------------------------------------------
Usage
--------------------------------------------------
./packcheck.sh COMMAND [PARAMETER=VALUE ...]

For example:
./packcheck.sh stack RESOLVER=lts-10.0 GHC_OPTIONS="-O0 -Werror"

Control parameters can either be passed on command line or exported
as environment variables. Parameters marked DESTRUCTIVE may modify
your global user config or state.

Boolean parameters can be specified as
y|Y|yes|Yes|YES|true|True|TRUE|on|On|ON for an affirmative value and as
n|N|no|No|NO|false|False|FALSE|off|Off|OFF or empty for a negative value.

--------------------------------------------------
Commands and flags
--------------------------------------------------
cabal-v2                : build using cabal v2-build
cabal-new               : Deprecated alias to cabal-v2
cabal-v1                : build using cabal v1-build
cabal                   : Deprecated alias to cabal-v1
stack                   : build using stack
clean                   : remove the .packcheck directory
cleanall                : remove .packcheck, .stack-work, .cabal-sandbox directories
help | --help | -h      : show this help message
--version               : show packcheck version

--------------------------------------------------
Selecting tool versions
--------------------------------------------------
ENABLE_GHCJS            : [y] Use GHCJS instead of GHC to build
GHCVER                  : [a.b.c] GHC version prefix (may not be enforced when using stack)
CABALVER                : [a.b.c.d] Cabal version (prefix) to use
RESOLVER                : Stack resolver to use for stack builds or cabal builds using stack
STACKVER                : [a.b.c.d] Stack version (prefix) to use
STACK_UPGRADE           : [y] DESTRUCTIVE! Upgrades stack to latest version

--------------------------------------------------
Where to find the required tools
--------------------------------------------------
PATH                    : [path] Set PATH explicitly for predictable builds
TOOLS_DIR               : [dir] Find ghc|cabal by version as in TOOLS_DIR/ghc/<version>/bin

--------------------------------------------------
Specifying common tool options
--------------------------------------------------
GHC_OPTIONS             : Specify GHC options to use
SDIST_OPTIONS           : Arguments to stack/cabal sdist command
CABAL_REINIT_CONFIG     : [y] DESTRUCTIVE! Remove old config to avoid incompatibility issues

--------------------------------------------------
Specifying what to build
--------------------------------------------------
DISABLE_BENCH           : [y] Do not build benchmarks, default is to build but not run
DISABLE_TEST            : [y] Do not run tests, default is to run tests
DISABLE_DOCS            : [y] Do not build haddocks, default is to build docs
DISABLE_SDIST_BUILD     : [y] Do not build from source distribution
DISABLE_DIST_CHECKS     : [y] Do not perform source distribution checks
ENABLE_INSTALL          : [y] DESTRUCTIVE! Install the package after building

--------------------------------------------------
stack options
--------------------------------------------------
STACK_YAML              : Alternative stack config, cannot be a path, just the file name
STACK_OPTIONS           : ADDITIONAL stack global options (e.g. -v) to append
STACK_BUILD_OPTIONS     : ADDITIONAL stack build command options to append

--------------------------------------------------
cabal options
--------------------------------------------------
CABAL_PROJECT           : Alternative cabal project config, cannot be a path, just the file name
CABAL_BUILD_OPTIONS     : ADDITIONAL cabal v2-build options to append to defaults
CABAL_BUILD_TARGETS     : cabal v2-build targets, default is 'all'
CABAL_CONFIGURE_OPTIONS : ADDITIONAL cabal v1-configure options to append to defaults
CABAL_CHECK_RELAX       : [y] Do not fail if cabal check fails on the package.
CABAL_NO_SANDBOX        : [y] DESTRUCTIVE! Clobber (force install) global cabal ghc package db
CABAL_HACKAGE_MIRROR    : [y] DESTRUCTIVE! Specify an alternative mirror, modifies the cabal config file.

--------------------------------------------------
Coverage options
--------------------------------------------------
COVERALLS_OPTIONS       : hpc-coveralls args and options, usually just test suite names
COVERAGE                : [y] Just generate coverage information

--------------------------------------------------
hlint options
--------------------------------------------------
HLINT_COMMANDS          : hlint commands e.g.'hlint lint src; hlint lint test'

--------------------------------------------------
Diagnostics options
--------------------------------------------------
CHECK_ENV               : [y] Treat unknown env variables as error, used with env -i
BASE_TIME               : System time to be used as base for timeline reporting
```

Options marked `DESTRUCTIVE!` are fine in a CI environment. But on a
local machine sometimes it may not be desirable as it will change the
state of your global cabal config, so consider that before using these options.

By default cabal builds are done using sandboxes. It creates any temporary
files or build artifacts inside `.packcheck` directory. See the `clean` and
`cleanall` commands to release the temporary space.

`stack` is automatically installed and can be used to do cabal builds as well.
If you specify `BUILD=cabal-v2` and `RESOLVER` at the same time then the cabal
build uses stack installed `cabal` and `ghc`, both are installed automatically
when needed.

For pure cabal builds i.e. when `BUILD=cabal-v2` and `RESOLVER` is not
specified, `cabal` and `ghc` must be pre-installed on the system before
building.

## Diagnostics

Sometimes you may run into issues due to some environment variables unknowingly
set or some command line parameters or env variables being misspelled and
therefore silently ignored. To avoid any such issues the robust way to invoke
`packcheck` is to use a clean environment using `env -i` and passing
`CHECK_ENV=y` parameter. When this parameter is set unwanted/misspelled
variables are detected and reported.

```
$ env -i CHECK_ENV=y ./packcheck.sh stack
```

For performance diagnostics `packcheck` prints the time elapsed from the
beginning at each build step performed.
