git-reltag
==============================================================================

Maintain signed version tags for release and manage deployment checkouts

| scott@smemsh.net
| http://smemsh.net/src/git-reltag/
| http://spdx.org/licenses/GPL-2.0


Description
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Do cryptographically-verified checkouts of signed tags, or add new
signed tags, with version numbers based on the current checkout.  Three
ordinals are used to represent version info, ala the `Semantic
Versioning`__ scheme.

- **releases** (new tags) are prefixed with the branch name on which the
  tag occurred, and the new version number, relative to the last one
- **deploys** (checkouts) select the target tag after first narrowing
  matches to prefix of the current checkout (as determined by ``git
  describe``) and doing requested math if relative (i.e. 'next', 'prev')


__ http://semver.org/


Usage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Subcommands are provided for tagging, checkout, and initialization:
| ``git reltag <subcommand>``

- **deploys** can checkout a specific tag, the one before or after the
  checked out one, or the last one that tagged the deploy's parent
  branch.
- **releases** add a new signed tag, incrementing any of the three
  ordinals (resetting less significant ones to zero); a subcommand is
  provided for each of these intentions.

release subcommands
-------------------

:patch: add new signed tag '<branch>-<X>.<Y>.<Z+1>'
:minor: add new signed tag '<branch>-<X>.<Y+1>.0'
:major: add new signed tag '<branch>-<X+1>.0.0'
:init:  add new signed tag '<branch>-0.0.0' (use for first tag)
:tag:   alias for 'patch' (canonical name, original script)


deploy subcommands
------------------

:sync: fetch && verify && checkout, last tag on parent branch
:next: fetch && verify && checkout, tag just after the checkout
:prev: fetch && verify && checkout, tag just before the checkout
:ver:  fetch && verify && checkout, exact given release tag $1


default subcommand
------------------

If no subcommand specified, default depends on current checkout:

:detached tag: *deploy latest tag of parent branch* ('**sync**')
:on branch: *add new tag incrementing patchlevel* ('**patch**')


safeties
--------

- refuses action in uncommitted repositories (dirty work area)
- for tagging ops, checkout must have had commits since last tag
- warns if checking out on branch (usual case is sync from detached tag)


Example
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. initialize repository we'll use for development::

    $ git init develop
    Initialized empty Git repository in /tmp/project/develop/.git/

#. add our first file to the project::

    $ cd develop
    $ touch testfile
    $ git add testfile
    $ git commit -m 'first checkin'
    [master (root-commit) 37e4ff1] first checkin
     1 file changed, 0 insertions(+), 0 deletions(-)
     create mode 100644 testfile

#. clone the development repo into the deploy location::

    $ cd ..
    $ git clone -q develop deploy

#. do our first release (using 'init' since no prior tags)::

    $ cd -
    /tmp/project/develop

    $ git reltag init
    tag master-0.0.0 successfully created, commit message:
    branch: master
    commit: 37e4ff1c5e78662d2c67d4f88265dd5e00417ebd
    tagname: master-0.0.0

#. sync the deploy now that we've made a release tag::

    $ pushd ../deploy
    /tmp/project/deploy /tmp/project/develop

    $ git reltag sync
    deploying over checked out branch 'master'
    branch head matches desired tag, detaching...
    branch: master
    commit: 37e4ff1c5e78662d2c67d4f88265dd5e00417ebd
    tagname: master-0.0.0

#. add another file to the dev repo and release as fix::

    $ pushd
    /tmp/project/develop /tmp/project/deploy

    $ touch anotherfile
    $ git add anotherfile
    $ git ci -m 'add another file'
    $ git reltag
    tag master-0.0.1 successfully created, commit message:
    branch: master
    prior: master-0.0.0
    changes: 1
    desc: master-0.0.0-1-gf741b48f96bfbfcbfb9259a6c6c208ef34c3e838
    commit: f741b48f96bfbfcbfb9259a6c6c208ef34c3e838
    prefix: master
    tagname: master-0.0.1

#. now that we're detached, we can deploy without args too::

    $ pushd
    /tmp/project/deploy /tmp/project/develop

    $ git reltag
    branch: master
    prior: master-0.0.0
    changes: 1
    desc: master-0.0.0-1-gf741b48f96bfbfcbfb9259a6c6c208ef34c3e838
    commit: f741b48f96bfbfcbfb9259a6c6c208ef34c3e838
    prefix: master
    tagname: master-0.0.1

#. prepare 1.0, major version update::

    $ pushd
    /tmp/project/develop /tmp/project/deploy

    $ echo 'ready for version 1' > README
    $ git add README
    $ git ci -m 'prep readme for GA release'
    $ git reltag major
    tag master-1.0.0 successfully created, commit message:
    branch: master
    prior: master-0.0.1
    changes: 1
    desc: master-0.0.1-1-g84b102c45c5d6b06d9e03bd958f7a13e4f564472
    commit: 84b102c45c5d6b06d9e03bd958f7a13e4f564472
    prefix: master
    tagname: master-1.0.0

#. and again the deploy will pull it in with no args::

    $ pushd
    /tmp/project/deploy /tmp/project/develop

    $ git reltag
    branch: master
    prior: master-0.0.1
    changes: 1
    desc: master-0.0.1-1-g84b102c45c5d6b06d9e03bd958f7a13e4f564472
    commit: 84b102c45c5d6b06d9e03bd958f7a13e4f564472
    prefix: master
    tagname: master-1.0.0

#. found a bug, take the deploy back to prior (working) release::

    $ git reltag prev
    branch: master
    prior: master-0.0.0
    changes: 1
    desc: master-0.0.0-1-gf741b48f96bfbfcbfb9259a6c6c208ef34c3e838
    commit: f741b48f96bfbfcbfb9259a6c6c208ef34c3e838
    prefix: master
    tagname: master-0.0.1

and so on.


TODO
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- allow user-specified tag prefix instead of branch name
- allow suffix string to be specified as well
- forward the gpg verify, avoiding need to trust deploy host's keystore


Status
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- used by author to develop and deploy
- most codepaths debugged, some remain, should write tests
- please inform if using
