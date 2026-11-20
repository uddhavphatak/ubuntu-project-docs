(merge-preliminary-steps)=
# Preliminary steps

Prepare for the merging procedure by cloning the required repository and filing a merge bug.


::::{admonition} **Merging** series
The article series provides guidance on performing package merges.

Process overview:
:   * {ref}`merges-syncs`

```{raw} html
<span style="font-size:0.1px"></span>
```

{ref}`How to do a merge <merging>`:
:   1. {ref}`merge-preliminary-steps` (this article)
    1. {ref}`merge-process`
    1. {ref}`merge-fix-the-changelog`
    1. {ref}`merge-upload-a-ppa`
    1. {ref}`merge-test-the-new-build`
    1. {ref}`merge-git-ubuntu-merge-proposal`

Extra:
:   * {ref}`merge-manually`
    * {ref}`merge-cheat-sheet`
::::


(merge-before-you-start)=
## Before you start

Complete {ref}`how-to-set-up-for-ubuntu-development` first. The commands in
this series use the source package name, which can differ from the names of
the binary packages you install and test.

This workflow merges Debian unstable into the Ubuntu development release.
Choose the target Ubuntu series and pocket explicitly; the historical
versions and release names in the examples are not current recommendations.
For an already released Ubuntu series, follow the
{ref}`stable-release-updates` process instead.


(merge-decide-on-a-merge-candidate)=
## Decide on a merge candidate

Check if a newer version is available from Debian. Use the {manpage}`rmadison(1)` tool:

```none
$ rmadison <package>
$ rmadison -u debian <package>
```

Example:

```none
$ rmadison at
 at | 3.1.13-1ubuntu1   | precise | source, amd64, armel, armhf, i386, powerpc
 at | 3.1.14-1ubuntu1   | trusty  | source, amd64, arm64, armhf, i386, powerpc, ppc64el
 at | 3.1.18-2ubuntu1   | xenial  | source, amd64, arm64, armhf, i386, powerpc, ppc64el, s390x
 at | 3.1.20-3.1ubuntu2 | bionic  | source, amd64, arm64, armhf, i386, ppc64el, s390x
 at | 3.1.20-3.1ubuntu2 | cosmic  | source, amd64, arm64, armhf, i386, ppc64el, s390x
 at | 3.1.20-3.1ubuntu2 | disco   | source, amd64, arm64, armhf, i386, ppc64el, s390x
$ rmadison -u debian at
at         | 3.1.13-2+deb7u1 | oldoldstable       | source, amd64, armel, armhf, i386, ia64, kfreebsd-amd64, kfreebsd-i386, mips, mipsel, powerpc, s390, s390x, sparc
at         | 3.1.16-1        | oldstable          | source, amd64, arm64, armel, armhf, i386, mips, mipsel, powerpc, ppc64el, s390x
at         | 3.1.16-1        | oldstable-kfreebsd | source, kfreebsd-amd64, kfreebsd-i386
at         | 3.1.20-3        | stable             | source, amd64, arm64, armel, armhf, i386, mips, mips64el, mipsel, ppc64el, s390x
at         | 3.1.23-1        | testing            | source, amd64, arm64, armel, armhf, i386, mips, mips64el, mipsel, ppc64el, s390x
at         | 3.1.23-1        | unstable           | source, amd64, arm64, armel, armhf, hurd-i386, i386, kfreebsd-amd64, kfreebsd-i386, mips, mips64el, mipsel, ppc64el, s390x
at         | 3.1.23-1        | unstable-debug     | source
```

In this example, the candidate from Debian `unstable` is `3.1.23-1`.
Record the current Ubuntu source version, the desired Debian source version
and suite, and the target Ubuntu series and pocket before continuing.

A merge is normally needed when the Ubuntu package carries Ubuntu-specific
changes, the {term}`Ubuntu delta`, that are still required in Ubuntu but are
not part of the Debian package. The merge carries those changes forward onto
the new Debian version.

If Debian already includes every required Ubuntu change, consider a
{ref}`sync request <how-to-request-a-sync>` instead of a merge. Check the
actual changes, not just the version strings.


(merge-check-existing-bug-entries)=
## Check existing bug entries

Check the Debian and Ubuntu bug lists for existing merge work and issues
that affect the merge.

* Ubuntu bug tracker: `https://bugs.launchpad.net/ubuntu/+source/<package>`
* Debian bug tracker: `https://tracker.debian.org/pkg/<package>`

Agree on any additional fixes with your reviewer and keep unrelated fixes
separate. Changes needed for the merge belong in their own logical commits,
as described in {ref}`merge-adding-new-changes`.


(merge-make-a-bug-report-for-the-merge)=
## Make a bug report for the merge

Many regular Ubuntu team merges are pre-planned and likely already exist as bugs, or can be found in the team merge schedule.

Merges can also be picked up from [Merge-o-Matic](https://merges.ubuntu.com/main.html), weekly Merge Opportunities Reports (e.g. the [Ubuntu Server report](https://lists.ubuntu.com/archives/ubuntu-server/2024-June/010077.html)), or through awareness being raised for other reasons.

If there is no obvious pre-created bug yet, check if there is an existing merge request bug entry in Launchpad. If you don't find one, create one to avoid duplicate efforts and to allow coordination.

To do so, go to the package's Launchpad page:

`https://bugs.launchpad.net/ubuntu/+source/<package>`

From there, create a new bug report requesting a merge.

:::{admonition} Example bug:

:URL: https://bugs.launchpad.net/ubuntu/+source/at/+filebug
:Summary: "Please merge 3.1.23-1 into noble"
:Description: "tracking bug"
:Result: https://bugs.launchpad.net/ubuntu/+source/at/+bug/1802914

:::

Set the bug status to "in-progress" and assign it to yourself.

To let people who only use Merge-o-Matic know, go to the summary page (for example, [universe](https://merges.ubuntu.com/universe.html)), and if the package is listed there, leave a comment linking to the bug.

This way, those not studying the LP bugs discover more easily that there is already a bug filed for that merge. To do so:

* Click in the {guilabel}`Comment` column on the invisible text entry field.
* Leave a comment like `bug #123456` and press {kbd}`Enter`.
* The page updates and links to your bug.

```{important}
**Save the bug report number, because you'll be using it throughout the merge process.**
```


(merge-get-the-package-repository)=
## Get the package repository

Cloning the repository is the start of all further interactions. If you have already cloned the repository, update it to ensure you have the newest content before taking any further action.


(merge-clone-the-package-repository)=
### Clone the package repository

```none
$ git ubuntu clone <package> [<package>-gu]
```

Example:

```none
$ git ubuntu clone at at-gu
```

It's a good idea to append some `git-ubuntu` specific label (like `-gu`) to distinguish it from clones of Debian or upstream Git repositories (which tend to want to clone as the same name).

If this is your first time working on a given package, you may get an error like `fatal: remote error: Repository '~your-lp-user/ubuntu/+source/at' not found`. This happens because you don't yet have a repository under your LP account for this package. It is safe to ignore the error.

(merge-update-the-package-repository)=
### Update the package repository

Since this is just Git, the best way to update the `git-ubuntu`-based content (and any other remotes) is to update them all before going into the merge process.

```none
$ git fetch --all
```


## Next

* {ref}`merge-process`
