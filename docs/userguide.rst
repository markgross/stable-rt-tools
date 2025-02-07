User Guide
==========

Dependencies & installation
---------------------------

This project depends on

- Python 3
- distro Python 3 package

To install kas into your python site-package repository, run::

    $ sudo pip3 install .


Installation
------------

- Install it locally via pip to get the ``stable-rt-tools`` command.

Assumptions
-----------

SRT needs kup installed and working.  This will mean you have write access to
some parts of kernel.org's directory structures an thus have a kernel.org
trusted gpg key and account.

Find kup here: `git://git.kernel.org/pub/scm/utils/kup/kup.git
<git://git.kernel.org/pub/scm/utils/kup/kup.git>`_.

There are some branch name assumptions you need to satisfy:
Follow the following pattern:
v<lts-id>-rt, and v<lts-id>-rt-rebase, and v<lts-id>-rt-next
For example; v4.9-rt, v4.9-rt-rebase, and v4.9-rt-next for example.

Also these branches must be tracking branches from the respective upstream
stable-rt branches of the same names.

Your git project needs an "origin", use the gitolite origin below to be safe.

SRT also assumes your work spaces for each branch is setup using git worktree
based off your parent local git project.  Set up your working directories this
way::

  git worktree add --track -b <branch> <path> <remote>/<branch>
  i.e. git worktree add --track -b v4.9-rt ../v4.9-rt k.org/v4.9-rt
  i.e. git worktree add --track -b v4.9-rt-rebase ../v4.9-rt k.org/v4.9-rt-rebase
  i.e. git worktree add --track -b v4.9-rt-next ../v4.9-rt k.org/v4.9-rt-next

Where k.org in this case
`git://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-stable-rt.git
<git://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-stable-rt.git>`_
It likely would have worked find if I just used "origin" instead of
having a separate read only remote.

Configuration
-------------

srt will look for a config file called 'srt.conf' following directories::

   .
   ~/.config
   ~
   /etc/srt

It is a INI type of configuration file. The group encodes
the upstream tree name and branch name. For example for

origin git@gitolite.kernel.org:pub/scm/linux/kernel/git/rt/linux-stable-rt.git
branch v4.4-rt

The group is called::

	[linux-stable-rt/origin/v4.4-rt]

srt expects the following keys::

	LOCALVERSION
	GPG_KEY_ID
	PRJ_GIT_TREE
	PRJ_DIR
	SENDER
	NAME

  LOCALVERSION: The filename of the localversion file, localversion-rt
  GPG_KEY_ID: ID of the GPG which should be used to sign and upload to korg
  PRJ_GIT_TREE: URL of the git upstream repo
  PRJ_DIR: kup folder to upload the release
  ANNOUNCE: Email template for release
  RC_TEXT: Email template for release candicate
  MAIL_TO: Email addresses to which the announces/patches should be send
  SENDER: Your name and email address
  NAME:  Your first name or nickname

Note for each branch you need to define a group (-rt, -rebase, -next)


Examples Configuration
----------------------
.. code-block:: ini

   [DEFAULT]
   MAIL_TO = LKML <linux-kernel@vger.kernel.org>,linux-rt-users <linux-rt-users@vger.kernel.org>,Steven Rostedt <rostedt@goodmis.org>,Thomas Gleixner <tglx@linutronix.de>,Carsten Emde <C.Emde@osadl.org>,John Kacur <jkacur@redhat.com>,Sebastian Andrzej Siewior <bigeasy@linutronix.de>,Daniel Wagner <dwagner@suse.de>,Tom Zanussi <tom.zanussi@linux.intel.com>,Clark Williams <williams@redhat.com>

   [linux-stable-rt/origin/v4.4-rt]
   LOCALVERSION = localversion-rt
   GPG_KEY_ID = 5BF67BC5082672CABB45ACAE587C5ECA5D0A306C
   PRJ_GIT_TREE = git@gitolite.kernel.org:pub/scm/linux/kernel/git/rt/linux-stable-rt
   PRJ_DIR = /pub/linux/kernel/projects/rt/4.4

   [linux-stable-rt/origin/v4.4-rt-rebase]
   LOCALVERSION = localversion-rt
   GPG_KEY_ID = 5BF67BC5082672CABB45ACAE587C5ECA5D0A306C
   PRJ_GIT_TREE = git@gitolite.kernel.org:pub/scm/linux/kernel/git/rt/linux-stable-rt
   PRJ_DIR = /pub/linux/kernel/projects/rt/4.4

   [linux-stable-rt/origin/v4.4-rt-next]
   LOCALVERSION = localversion-rt
   GPG_KEY_ID = 5BF67BC5082672CABB45ACAE587C5ECA5D0A306C
   PRJ_GIT_TREE = git@gitolite.kernel.org:pub/scm/linux/kernel/git/rt/linux-stable-rt
   PRJ_DIR = /pub/linux/kernel/projects/rt/4.4


Workflow Example
----------------
.. code-block:: console

  $ cd v4.4-rt
  $ git tag -l 'v4\.4\.*' --sort=v:refname | tail
  $ git merge v4.4.120      [fixup conflicts]

  $ git push lxcvs -f --follow-tag HEAD:stable-maintenance-4.4.y-rt

  $ srt commit
  $ srt tag

  $ cd v4.4-rt-rebase
  $ git rebase -i v4.4.120  [fixup conflicts]
  $ srt commit
  $ srt tag

  $ srt create v4.4.115-rt130 v4.4.120-rt135
  $ srt sign v4.4.115-rt130 v4.4.120-rt135
  $ srt upload v4.4.115-rt130 v4.4.120-rt135
  $ srt push v4.4.115-rt130 v4.4.120-rt135

  # XXX push missing tags
  $ git push origin v4.4.115-rt131 v4.4.116-rt132 v4.4.118-rt133 v4.4.119-rt134

  $ srt announce v4.4.115-rt130 v4.4.120-rt135 > ../announce-rt
  $ cat ../announce-rt | msmtp -t --
  or
  $ mutt -H ../announce-rt


Release candicates series
-------------------------
.. code-block:: console

  $ cd v4.4-rt-next
  $ git reset --hard v4.4-rt

  [ backport patches ]

  $ srt commit -r 1
  $ srt tag

  $ srt create v4.4.148-rt165 v4.4.148-rt166-rc1
  $ srt sign v4.4.148-rt165 v4.4.148-rt166-rc
  $ srt upload v4.4.148-rt165 v4.4.148-rt166-rc1
  $ srt push v4.4.148-rt165 v4.4.148-rt166-rc1
  $ srt announce v4.4.148-rt165 v4.4.148-rt166-rc1


Trouble shooting
----------------

srt announce fails:  it attempts to figure out your name and email from
specific configurations you might not have (say if you using Ubuntu) if
srt announce fails try adding SENDER and NAME's to your srt.config groups.

srt commit fails: check for missing remote named "origin" if so add one using
git remote add origin pointing anywhere.  One maintainer used a local mirror to
Linus' tree.  Its used as a handle to disambiguate multiple workflows.

srt upload attempts fail: verify kup works following the README in kup, then
check your str.config has the appropriate PRJ_DIR for your LTS version of
stable-rt you are working with.  And if that fails check you have proper access
to the PRJ_DIR on kernel.org through the helpdesk.

There is a debug option you can add to the srt command line that turns on some
tracing,  "-d".  One new maintainer bumped into each of the above and the
tracking branch assumption and used the -d option to debug their issues.

