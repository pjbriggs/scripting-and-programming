``git`` workflow notes
=======================

Background
**********

Atlassian has a nice tutorial comparing different git workflow models:

* https://www.atlassian.com/git/tutorials/comparing-workflows

There is this key post on **git-flow**:

* http://nvie.com/posts/a-successful-git-branching-model/

and one on **github-flow** (a simplified version used at github):

* http://scottchacon.com/2011/08/31/github-flow.html

Workflow
********

I think I want something a bit like git-flow, where:

* ``master`` stores the official release history, and
* ``devel`` serves as an integration branch for features.

There are 3 aspects:

* Adding new features
* Making releases
* Making bug fixes (i.e. maintenance)

To add a feature
----------------

Features branch off from ``devel`` and are merged back into ``devel``
when they're finished.

This would look something like::

    git clone REPO
    git branch feature-NAME devel
    git checkout feature-NAME

or more concisely::

    git checkout -b feature-NAME devel

Then do your work on the feature and commit to this branch. It can be
pushed to ``origin`` using::

    git push -u origin feature-NAME

To merge the feature into ``devel``::

    git pull origin devel
    git checkout devel
    git merge feature-NAME
    git push origin devel
    git branch -d

(The last one deletes the feature branch).

To make a release
-----------------

Releases branch off from ``devel`` and are merged back into both
``devel`` and ``master``. The release branches serve as a place to
prepare a release, specifically:

* Clean up and test the release
* Do testing
* Update the documentation, change log etc

It also establishes the version number and essentially
feature-freezes a release::

    git checkout -b release-1.0 devel
    git push -u origin release-1.0

Once the release is ready::

    # Merge into master
    git checkout master
    git merge release-1.0
    git push
    # Merge into devel
    git checkout devel
    git merge release-1.0
    git push
    # Delete release branch
    git branch -d release-1.0

Finally tag the release (good idea to do this every time you push
the ``master`` branch)::

    git tag -a 1.0 -m "Initial public release"
    git push --tags

To make bug fixes (maintenance fixes)
-------------------------------------

Bug fixes (known as 'hotfixes' in git-flow parlance) branch off from
``master`` and are merged back into ``master`` and ``devel``. These
are the only branches allowed from ``master``.

According to the git-flow methodology the first step when making the
maintenance branch is to update the version number.

For example::

   # Make the bug fix branch
   git checkout -b issue-NAME master
   # Update the version number e.g. 1.0 to 1.0.1

.. note::

   Consider using the ``bumpversion`` Python package? See
   https://pypi.python.org/pypi/bumpversion

Fix the bug then merge into ``master`` and ``devel``. In ``master``
you'll also need to tag the new 'release':

   # Merge into master
   git checkout master
   git merge issue-NAME
   git tag -a 1.0.1 "Fixed bug"
   git push
   git push --tags
   # Merge into devel
   git checkout devel
   git merge issue-NAME
   git push
   # Delete bug fix branch
   git branch -d issue-NAME

.. note::

   This procedure is for bugs in released versions. If the bug
   is found while a release is being prepared and there is an
   existing release branch then merge into that rather than into
   ``devel``.

   (If the bug fix is also needed in ``devel`` then you can merge
   into ``devel`` as well.)

Pushing branches to github
**************************

The github-flow article suggests that pushing the branches to github
means that you automatically get a list of features and bugs that
are being worked on, along with status reports.

Also if you're using something like Travis CI then this can trigger
testing for your changes.

Using ``--no-ff`` for merging
*****************************

The git-flow article uses ``--no-ff``, which I think means you always
get a commit associated with the merge - so there is always a record
left in the history that the feature or bug fix was added.

