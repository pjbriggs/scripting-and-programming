Mercurial Crib Sheet
====================

`hg help COMMAND` Get help on COMMAND

`hg version` Get version information for mercurial

`hg clone https://bitbucket.org/...`

`hg clone -U REPO REPO.CLONE` Create a clone of REPO in REPO.CLONE without a working copy (e.g. for backup)

`hg status`

`hg commit`

`hg tip` Show the tip revision

`hg update [ -r BRANCH ] -C` Revert to most recent version, `-C` discards local changes (`hg purge`
removes untracked files)
