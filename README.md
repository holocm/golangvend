# golangvend - minimalistic vendoring tool for Golang dependencies

I made this for my own Go projects since none of the existing vendoring tools
for Go libraries fit my needs, especially:

1. I want for my users to be able to `git clone` the repo and `make` it without
   worrying about shit like GOPATHs or git submodules, and most certainly
   without having to install the vendoring tool.

2. I want to be able to make a release tarball that people can compile without
   needing an internet connection. Package building should work even if Github
   is down.

All the vendoring tools that I found, such as godep, rely in some way on the
source tree being placed at the right location in `GOPATH`. Even Go 1.6's
`vendor/` path feature has the same requirement.

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
2. Make sure there is no `localdeps/` directory. This is where golangvend will
   write stuff.
3. Format your code with `gofmt`. golangvend expects your code to be formatted
   according to `gofmt`, and will likely explode violently when fed
   malformatted code. :)

Now run golangvend. It will look for external dependencies in all source files.
For every such dependency, it will pull the source code into `localdeps/`, e.g.
at `localdeps/github.com/user/repo` for dependencies from Github. Then
golangvend will go through the source code another time, and replace `import`
statements for all vendored dependencies to point at the localdeps tree by
using a relative import path.

Review the changes and `git add` everything. Now there's a small catch: If the
dependencies you just vendored have other dependencies, you did not vendor
these yet. The solution is easy: Just run golangvend again. Repeat until
golangvend does not add any new dependencies into the localdeps tree.

## Adding another dependency

Just add the new import statement to your code. Then save the source file and
run golangvend. Again, if the dependency pulls in other dependencies, multiple
runs may be necessary.

## Updating a vendored dependency

golangvend will pin each vendored dependency to its most recent commit, by
writing the commit hash to `localdeps/pins/`. To update a dependency to the
newest commit, delete that dependency's pin file, then run golangvend.

To update to a specific commit of your dependency, write the commit ID into the
pin file, then run golangvend.

## Removing a dependency

golangvend will not remove dependencies from the localdeps tree by itself when
they are not used anymore. (Should be easy to add, though. Pull requests are
welcome.) If you remove a dependency and it's not used anymore, you can remove
its source tree from below `localdeps/` and its pin from `localdeps/pins`
manually.

## When all is fucked and you want to get out

To revert all the changes that golangvend did, delete the `localdeps` and
`.golangvend-cache` trees, then

```bash
grep /localdeps/ $(find -name \*.go)
```

to find all the import statements that golangvend edited, and remove the path
up until and including `/localdeps/`.

# The fine print

I have only tested this on my own code. Your code is likely different, so you
may run across bugs. Please don't trust the output of golangvend blindly;
review its changes with `git diff` or `git add -p`. As a rule of thumb, if you
can `go build` and `go test` everything with the `GOPATH` variable unset, then
everything should be fine. It's actually a good idea to [put this in your Makefile or build script](https://github.com/holocm/holo-build/blob/96f0e5b4ea7d758dff462a7f4bb3e16d9afb22b0/Makefile#L5),
to enforce that your contributors also use golangvend when necessary.
