# golangvend - minimalistic vendoring tool for Golang dependencies

Vendor your dependencies into a `vendor/` directory without all the hazzle.

# Installation

`git clone` this repo, then put golangvend into your PATH. For example:

```bash
git clone https://github.com/holocm/golangvend
ln -s $PWD/golangvend/golangvend ~/bin # or wherever you put your tools
```

No compilation required, no obscure dependencies. Just a shell script.

# Usage

## Setting up vendoring in a repository

Go to the repository root directory. Before running golangvend,

1. Add the path `.golangvend-cache/` to your `.gitignore`. You don't want to
   commit that.
2. Make sure there is no `vendor/` directory. This is where golangvend will
   write stuff.

Now run golangvend. It will look for external dependencies in all source files.
For every such dependency, it will pull the source code into `vendor/`, e.g.
at `vendor/github.com/user/repo` for dependencies from Github. If dependencies
have dependencies themselves, these will be vendored, too.

Review the changes and `git add` everything.

## Adding another dependency

Just add the new import statement to your code. Then save the source file and
run golangvend.

## Updating a vendored dependency

golangvend will pin each vendored dependency to its most recent commit at
initial import time, by writing the commit hash to `vendor/pins/`. To update a
dependency to the newest commit, delete that dependency's pin file, then run
golangvend.

To update to a specific commit of your dependency, write the commit ID into the
pin file, then run golangvend.

## Removing a dependency

Remove the import statement from your code. The next golangvend run will detect
the unused import. To remove unused imports, run `golangvend --delete-unused`.

## When all is fucked and you want to get out

To revert all the changes that golangvend did, delete the `vendor` and
`.golangvend-cache` trees.

# The fine print

Please don't trust the output of golangvend blindly; review its changes with
`git diff` or `git add -p`.

To ensure that all dependencies are vendored, it's a good idea to set up your
Makefile such that the build sees an otherwise empty `GOPATH`. For example,
assuming that your project is at `github.com/foo/bar`:

```bash
cd $GOPATH/src/github.com/foo/bar
mkdir -p gopath/src/github.com/foo
ln -s $PWD gopath/src/github.com/foo/bar
env GOPATH=gopath go build github.com/foo/bar
```

This is also necessary when trying to build a golangvend-vendored project
outside of a GOPATH, e.g. when extracted from a release tarball.
