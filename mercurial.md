Mercurial Crib Sheet
====================

Basics
------

`hg help COMMAND` Get help on COMMAND

`hg version` Get version information for mercurial

`hg clone https://bitbucket.org/...`

`hg clone -U REPO REPO.CLONE` Create a clone of REPO in REPO.CLONE without a working copy (e.g. for backup)

`hg status`

`hg commit`

`hg tip` Show the tip revision

Configuration
-------------

Configuration is stored in `~/.hgrc` (on Linux). This file doesn't exist by default. For details of the format
and content see [hgrc](http://www.selenic.com/mercurial/hgrc.5.html).

Discarding local changes in working copy
----------------------------------------

`hg update [ -r BRANCH ] -C` reverts to most recent version, with `-C` discarding local changes.

`hg purge` removes untracked files in the working directory.

Merging updates from remote repository
--------------------------------------

`hg pull` fetches changes from the remote repository and adds them to the local repository. By default this
doesn't update the copy in the local working directory.

`hg update` updates the repository's working directory to the most recent changeset.

Note that `update` may resulting in conflicts that will need to be resolved. To set the editor to use when
resolving conflicts on merge use e.g.:

    [ui]
    merge=vi
    
Set `merge=internal:merge` to avoid going into an interactive editor session when conflicts are detected on
merge. Instead the conflicts are marked with "conflict markers". NB if conflicts are resolved manually then
use

    hg resolve -m [FILE]
    
to tell Mercurial that the conflicts have been resolved (see (help:resolve)[http://www.selenic.com/hg/help/resolve])
