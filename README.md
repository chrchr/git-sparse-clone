# Git Sparse Clone

Custom Git command to easily clone selected subdirectories from a repository.

Git supports checking out subdirectories of a repository, but doing
so efficiently requires multiple steps and some esoteric options. This
plugin collects those procedures into a custom command, so that you
can clone a subdirectory of a Git repo with a single command, like
this:

```
git sparse-clone https://github.com/owner/repo.git subdirectory1 subdir2/path ...
```

The procedure used by this command is described in the GitHub blog post
[Bring your monorepo down to size with sparse-checkout](https://github.blog/open-source/git/bring-your-monorepo-down-to-size-with-sparse-checkout/).


## Installation

Copy the file `bin/git-sparse-clone` somewhere in your `PATH`, or add
its location to your `PATH`. It is a single, stand-alone shell script
and its dependencies are `git`, `grep`, `mkfifo`, `mktemp`, `sed`,
`sh`, and `tee`. You should already have those!


## Usage

When `git-sparse-clone` is in your `PATH`, you can invoke it with
`git sparse-clone`, like an ordinary `git` command. To borrow the example
from the blog post above, the following command creates a tree
in the directory `sparse-checkout-example` with all of the top-level
files from the source repo and all of the files in `service/common`
and `service/identity`, but no other files.

```
git sparse-clone \
"https://github.com/derrickstolee/sparse-checkout-example" \
service/common service/identity
```

To specify the location of your git tree, use the `-d` or `--directory` option.

To checkout a branch other than the default branch, use `-b` or `--branch`, as in `git clone`.

The `--depth` option allows the creation of shallow clones. Use the
`--depth=1` option to create a shallow clone with no revision history,
reducing storage and speeding up the cloning process.

The `--no-filter` option disables partial cloning. When `--no-filter`
is enabled, the initial clone will download all of the objects from
the repo. When disabled (the default), `sparse-clone` will download as
little as possible on the initial clone. See [Server Configuration](#Server-Configuration)
for more information.

The flags `--sparse-index` and `--no-sparse-index` enable or disable
`git`'s experimental sparse index feature. The default is `--sparse-index`.


## Working in Sparse Checkouts

You can modify the cloned tree using Git's built-in `sparse-checkout`
command, adding or removing subdirectories from the source repo. The
command `git sparse-checkout add <directories> ...` adds additional
directories from the source repo to your tree. The Git command `git
sparse-checkout set <directories>` changes the list of directories in
the tree to only those specified, replacing any that are currently
selected. The command `git sparse-checkout list` shows you the current
state of included directories, and `git sparse-checkout disable`
converts the tree to a standard, non-sparse tree with the full
contents of the source repo.


## Server Configuration

If you're running your own git server, you may need to enable some
options on your Git host to support efficient sparse checkouts. These
are `uploadpack.allowFilter` and
`uploadpack.allowReachableSHA1InWant`. They can be enabled by running
the following in the account that runs the Git server:

```
git config --global uploadpack.allowFilter true &&
git config --global uploadpack.allowReachableSHA1InWant true
```

These options are already enabled on popular Git hosting platforms such as GitHub
and GitLab. Also, `sparse-clone` can be run with the `--no-filter`
option to enable compatibility with Git hosts that have not configured
these options.

To explain, `allowFilter` enables the filtering feature that reduces
the data transfer volume needed to clone a repo. When filtering is not
enabled, the clone still succeeds, but it may take longer and use more
bandwidth, and results in the following warning:

```
warning: filtering not recognized by server, ignoring
```

The `allowReachableSHA1InWant` option must be enabled on the server to
create a sparse tree with filtering enabled. When it is disabled and
the `--no-filter` option is not used, the following error occurs:

```
error: Server does not allow request for unadvertised object <sha1>
fatal: could not fetch <sha1> from promisor <remote>
```

This is a consequence of running `git checkout` on a filtered tree. On
checkout, the local `git` must request all of the objects it needs
from the server, and that operation is not permitted by default.


## Limitations

* `git clone` includes numerous options and features which `git
  sparse-clone` does not support.

* While core Git operations work with sparse clones, they
  may be incompatible with some tools.


## References

* [git-sparse-checkout Documentation](https://git-scm.com/docs/git-sparse-checkout) is the official documentation for sparse checkouts.
* [Git Internals - Transfer protocols](http://git-scm.com/book/en/Git-Internals-Transfer-Protocols) details how git clients and servers interact.
* [git-config Documentation](https://git-scm.com/docs/git-config) provides information on the `uploadPack.allowFilter` and `uploadPack.allowReachableSHA1InWant` options.
* The blog post [Bring your monorepo down to size with sparse-checkout](https://github.blog/open-source/git/bring-your-monorepo-down-to-size-with-sparse-checkout/) describes the process for creating sparse checkouts.
* The blog post [Make your monorepo feel small with git's sparse-index](https://github.blog/open-source/git/make-your-monorepo-feel-small-with-gits-sparse-index/) explores Git's sparse index feature.
