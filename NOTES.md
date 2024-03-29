
# PHILOSOPHY

Everything gi *does* is compatible with git.

But it will be pedagogically simple and conceptually clear.

# MOTTO

“If you want git-restore-spatulated-blastospheres, you know where to find it.”

Gi doesn't have to support every possible use case, because Git already does that.

Gi is not intended to replace Git.  It is intended to provide a smoother learning curve to get people to Git.

# FOR EXAMPLE

Maybe `git commit file` is convenient.
But it's internally complex and pedagogically confusing.
So we aren't going to do it.
`gi commit` commits THE INDEX.
If you really want "git commit file", you know where to find it.

Similarly, "gi commit" is forbidden when the HEAD is detached.
But sometimes it's useful to commit when the HEAD is detached.
Fine. If you want to do that, you can use git-commit.

Git interactive rebase is powerful, but complex. There are a lot of
options and it's not clear ahead of time what it will do.  For
example, while you're learning what it is for, you _also_ have to
learn to deal with resolving conflicted merges.

We can provide several limited versions that address common individual
use cases, thus splitting the complexity into smaller pills that can
be swallowed one at a time.

# BASIC CONCEPTS THAT WE EMPHASIZE

* HEAD

  * headref     -- this is the thing that doesn't exist when you have a detached head.
  * head commit -- there is always a head commit, unless you tampered with `.git/HEAD`.

* The index

  * We commit THE INDEX.
  * We DO NOT commit "changes".
  * We DO NOT commit "a file".

  The terminology here is a mess:

  * Do not refer to the index as "the stage" or "the cache".
  * Do not refer to indexed changes as "staged" or "cached"?
  * Or if you do, pick one and stick with it!

  Don't say "the staged changes" unless you have recently used the longer form
  "The changes staged in the index".

* Branches

  * A branch is a named commit plus all its ancestor commits.
  * Thus some branches contain others.

# Stuff that confuses beginners

## Detached head

We'll fix this with explicit `gi-detach` and `gi-attach` commands.
Instead there's some suite for copying a commit to the index, for
copying the index to the working tree, and `gi-go` for moving HEAD to a
new commit.

(If `gi-go` is for switching branches, why do we need `gi-attach`?
Maybe the difference is, when HEAD is detached, `gi-go` is forbidden,
you have to use `gi-attach`?)

There is always a headref unless you *explicitly* use `gi-detach`.

(Big question: Why do we need detached HEAD at all?  What is
`gi-detach` even for?  Right now I'm coming up with nothing.  We maybe
need some way to recover if the head somehow becomes detached.  But I
see no reason why `gi-go` wouldn't be sufficient.)

You said above that `gi-commit` should fail if the head is detached.
Suppose you detact the head and change the working tree.  How do you
get your changes onto the branch where you want them?

You said below that attempting to switch branches with a dirty working
tree should temporarily commit them so they can be restored when you
come back.  But if head is detached this doesn't exactly make sense.
I think a reasonable response would be:

> You said to switch to branch `topic`, but you have uncommitted
> changes that you would lose.  Would you like to:
>
> (F)orget the uncommitted changes and switch branches
> (C)ommit the changes as a new branch and switch branches
> (Q)uit the switch-branches operation so you can do something else

(C) might prompt you for the name of the new ref, with some not-crazy
default like `wip-Thursday-afternoon`.

After (Q) you could use something like

    gi branch --attach new-topic-name

to name the current branch, after which `gi go` would work, silently
saving the working tree as usual.

## Rejected PUSH

I think this is *unavoidable complexity* and that git's treatment of
this is correct. Beginners will have to deal with it.  But think about
it anyway; maybe there's some way we can clean it up.

Maybe something to offer helpful advice about what to do.

## Dirty working tree

If the working tree is dirty, an attempt to switch branches has one of
two effects: fail with an error message, or carry the uncommitted
changes to the new place.  Both are confusing, and worse, it is hard
to predict which will happen.

A better behavior would be: silently commit the changes, and note in
the commit message that this was a temporary commit of a dirty tree.
When the user comes back to this branch, the changes are still there
and the ref can be reset to the previous commit.

I thought maybe this was similar to what Git calls `autostash`, but
it's not.  That applies only to `rebase`.  Of course, we would want
this to apply to `rebase` _also_.

## Resolving conflicts

It's really not clear how to proceed with this the first time one sees
it, and fixing it requires knowing how to look for and decipher
conflict markers.  The frequency with which conflict markers actually
get into the repository is witness to how tricky this is.  What can we
do instead?  See below.

How about an interactive dialogue, a bit like `git-add -p`, but more
conversational and less deliberately obscure?  Something like:

> I tried to copy your commits to the new branch, but on that branch
> the file 'blah/blah' had already been modified in ways that seem to
> conflict with some of your changes.  There are 2 such conflicts.
>
> Would you like to: (A)bort the whole copy-branch operation
>                    (R)esolve the differences one at a time with my help
>                    (E)xit to the shell to (E)dit the files with conflicts

The (R) will start an interactive loop which presents chunks (or
sub-chunks?) one at a time:

> The file originally had:
>
> ...
> 
> and you changed it to:
>
> ...
>
> but in the meantime the branch "foo" changed it to:
>
> ...
>
> Would you like to:
>   (K)eep your change
>   (D)iscard your change
>   (E)dit the file to combine the changes
>   (A)bort the whole copy-branch operation

What if the chunk is really big?  How about another prompt?

> The next conflicted section of the file is 42 lines long.  Would you
> like to:
>
>   (S)ee it now and have me ask you again
>   (D)eal with it all at once
>   (B)reak it into 4 smaller chunks to deal with separately
>   (C)ome back to this after dealing with the other conflicts

## The index

[Ori Dean Bernstein's translation of Git to Plan
9](https://github.com/oridb/git9/blob/master/README) says:

> the entire concept of the staging area has been dropped, as it's both confusing and clunky

I can't really disagree with this; beginners often see the index as
being just an annoying extra step between them and what they want,
which is to commit the changes.  And the interface is definitely
clunky.  I said below “beginners don't need the index”.

I wonder, what if there were a mode in which the index was replaced
with a separate working tree?  Then adding something to the index
would be equivalent to copying it to the staging tree.  A commit
operation would simply commit the contents of the staging tree.
All the commands like `git-reset`, `git-add`, `git-diff --cached`
disappear, or become shorthands for simple file operations.  Instead
of `git add -p` you now just edit the index file directly, instead of
dealing with patches.

But the rest of your document seems to take for granted that the index
is important.  Why did you think this?  Can we just get rid of it?  We
lose `git-add -p`, which is amazingly powerful, but also hard to use.
Perhaps more important, we lose the ability to commit only a subset of
the changed files.

# COMMANDS

## `gi-go` <ref>

Attach HEAD to some other ref:

       HEAD now attached to <ref>

The commit pointed to by the old headref doesn't change.

If HEAD was detached, it is attached, that's OK.

## `gi-detach`

Basically git-checkout HEAD~0.  Message is "HEAD detached" or "HEAD
was already detached".

Need some warning here about possible lost commits.

## `gi-branch` <name>

Make a new ref at the current head commit.

`-a` flag attaches to the new ref.

## `gi-commit`

Commits the index and moves the

Forbidden when in a detached HEAD state.

## `gi-move` <ref> <commit-ish>

Moves an existing ref to a new commit.  Work could be lost.  Dangerous?
If you do supply this, have it leave behind OLDLOC or something.

## `gi-checkout` <ref>

Attaches the HEAD to the specified ref.
Then copy the head commit to the index, then to the working directory.

Forbidden if the index or working tree are dirty.

## `gi-discard-changes`

Discards working tree changes: copies the index to the working tree.

(Optional files: copy only those files?)

Ought to support an “undo” feature somehow.

## `gi-reset-index`

Discards index changes: copies the head commit to the index.

(Optional files: copy only those files?)

## git commands that we will pass through to gi unchanged

    gi-cherry-pick
    gi-status
    gi-status--short

    gi-merge
    gi-grep
    gi-log

Most of the remote repo commands:

    gi-fetch
    gi-clone

## git commands where we should change the interface or the options or something

### `gi-diff`

It's hard to guess which two things will be compared.  Try to rationalize the argmuents:

     gi diff commitish-A commitish-B
     gi diff             commitish-B    (commitish-A is taken to be HEAD)
     gi diff INDEX                      ("INDEX" is treated specially)

     gi ix                              (Synonym for 'gi diff INDEX'?
                                         probably not; if they want
                                         that they can make an alias.
                                         Do not multiply entities.)


### `gi-push`

I found the distinction between the ref and the remote name confusing.
We can probably clean up the interface here.  Also get a better syntax
for things like "localref:remoteref".  Maybe the remote is *always*
assumed to be `origin` unless you use a `--remote=foo` option or
something.

Major terminology problem: Git does not have a clear, simple piece of
jargon for distinguishing between:

 1. The ref `foo` in the remote repo
 2. The ref `remote/foo` in the local repo
 3. The ref `foo` in the local repo

These are all related but different.  To understand `git-fetch` and
`git-push` you have to understand that there are three entities here.
But there's not way to talk about them because they don't have names!
But we want to say that `git-fetch` copies 1 to 2, and `git-push`
copies 3 to 1 _and_ to 2.

The existing `git-push` has confusing notation for how to copy
branches from local to remote repo.  For example, why

    git push origin :topic

for deleting a remote branch?

Idea to think about: beginners often confuse the remote branch named
`topic` with the local branch named `origin/topic`.  What if gi
conflates them, so that we talk about remotes as little as possible?
For example, instead of:

* `git fetch origin topic` → `gi update origin/topic`.
* `git push origin topic` → `gi copy-branch topic origin/topic`

  (Does fetch plus rebase plus push)
  
* `git push origin topic:x-topic` → `gi copy-branch topic origin/x-topic`
* `git push origin :topic` → `gi delete-branch origin/topic`
* `git remote --remove origin` → `gi delete-branch origin/*`

  (Presented only to demonstrate the analogy.  We don't actually need
  a Gi version of `remote --remove`.)

Maybe this goes beyond making things as simple as possible, into the
realm of “too simple”.

### `gi-fetch`

The difference between `git-fetch`, `git-fetch remote`, `git-fetch`
remote branch', and maybe `git-fetch branch` is confusing.  What's the
common case here?  What's the thing that beginners most need to do?
My own common case is probably `git-push origin HEAD`.  Maybe do that.

Whatever you do about the remote for gi-push, do the same thing here
too.

### `gi-rebase`

There are at least two problems with rebase.  One is that it almost
inevitably leads to rejected pushes.  I don't think that can be fixed,
and I don't think it can be hidden. As discussed above, it must be
faced head-on.

Another problem is that it seems like a scary headfirst dive into
something complex.  We can mitigate this in a couple of ways.

1. `gi-copy-branch`

   This differs from rebase in two ways.  Suppose you have `topic` checked out.
   `git-rebase target` does the following:

     1. check out the `target` commit (not `target` itself)
     2. cherry-pick commits from the merge-base up to `topic`, stopping to resolve conflicts
     3. if the cherry-picks are successful, move the `topic` ref to the end of the new branch
     4. check out `topic`.

   If successful, it's not clear what happened to the original branch.
   Experienced git users can undo this easily with `git reset --hard
   ORIG_HEAD` or by using the reflog, but beginners are at sea.  Also,
   the jargon is worrisome.  "Rebase"?  What's that?

   Instead, we can provide `gi-copy-branch`.  In the same case,
   `gi-copy-branch target` does the following:

     1. check out the `target` commit  (exactly the same)
     2. cherry-pick commits the same, but if there is a merge conflict, abort the whole thing
     3. if all the cherry-picks succeed, create a new `topic-copy` ref at the end of the new branch
     4. check out `topic`, which has not moved

   End result: you are in exactly the same place, `topic`, which has not
   changed.  But you either got an error message (some elaboration of
   "couldn't copy all the commits") or else the only new thing is there
   is now a new branch, `topic-copy`, on the end of `target`, which you
   can check out or examine as usual.  If you decide you don't like it,
   you can use `gi-delete-ref topic-copy` to get rid of it.  (Note,
   delete *ref*, not delete *branch*.  The _branch_ includes all the
   commits back to the initial commit, including everything on `target`.)

   (Instead of the `gi-checkout` and `gi-delete-ref` maybe a separate
   command: `gi-abandon-original-branch` or something, that moves `topic`
   to `topic-copy` and deletes the `topic-copy` ref. Find a better name
   for this.)

   There can be an option to allow merge resolution instead of instant failure.

   The error message can describe the commits that failed, if there were
   any, and mention the merge resolution option.  If the user does permit
   failed merges, and there is a failure, `gi-wtf` has its work cut out
   for it.

2. `gi-reorder-commits`

   Like interactive rebase, but instead of commands, you just get a list
   of commits, and edit them into order, they are then rebased into the
   requested order.  Unless you provide the expert-level option, a failed
   cherry-pick aborts the whole thing.

   What are the arguments to this?  Maybe `--back-to commitish` or `-n 6`
   to reorder the last 6?  Maybe `--from commitish-A --to commitish-B`?
   No, that's no good, because we can't truly support `--to`.

   Maybe the result should still be in `foo-copy` and you use
   `gi-abandon-original-branch` as with gi-copy-branch.

3. `gi-combine-commits`

   (Or maybe `squash-commits`.  Do _not_ call it `merge-commits`.)

   Like `git-rebase`'s squash option.  What is the UI?  Maybe you can just say

         git combine-commits c1 c2 c3 ... cn

   and it does a combined reorder plus squash, invokes the editor once
   with the combined commit messages from c1...cn, and leaves everything
   else in order?  For example if you have commits 1 2 3 4 5 6 7 and you
   do `gi combine-commits 2 4 5` then you end up with 1 245 3 6 7.

   As usual, a failed merge aborts the whole thing unless you give the special option.

   Same thing as above about `gi-abandon-original-branch`.

4. `gi-amend`

   This is just a wrapper around `git commit --amend`.  Replaces the head
   commit with the current index.  Maybe instead of moving the headref,
   it creates headref`-emended` so you can compare?  This can't fail, but
   it could support the `gi-abandon-original-branch` semantics if we
   wanted.  It can also easily support an undo.

5. `gi-move-branch`

   I thought this was a good idea for a few minutes, but NO. It seems
   easy to understand, by analogy to gi-copy-branch.  But the analogy is
   wrong.  It _does not_ actually move a branch.  It makes a copy and
   discards the original.  If we have gi-move-branch, someone will move a
   branch and ask how to move it back again—which is the wrong question.
   So instead, perhaps

           gi copy-branch --forget-original

   or some such.  It can print an instruction about how to get the
   original back.

## git commands I'm still thinking about

### `git-branch`

`-m` is confusing. It looks like it moves branches, but it moves refs.
Maybe call it `gi-move-ref` or something.  This is a basic command that
git lacks; the interface on `git-update-ref` is horrendous.

### `git-mv`

### `git-rm`

### `git-show`

The idea is sound, but I think we can do better here.  But isn't 99%
of the use-case to just do git-log -1?  If so, git-show is probably
okay.

### `git-status`

The long output from `git-status` is helpful to a lot of people.  But I
think the output format could be made simpler or more useful or
something.

I wonder if gi should provide `gi-s` as an alias for `git-status -s`?  Is
`status -s` even important? I think maybe this is MJD personal bias
creeping in.

Getting rid of the index would simplify the output quite a bit. 

### `git-stash`

Idea: `gi-stash` uses the regular git stash, but instead of a pile of
stashes, there is only one.  If you have stashed changes already, you
may not `gi-stash` new changes.  If you need to do that, `git-stash` is still available.

Is this really helpful?  It's not at all clear that people find
multiple stashes confusing.  The idea, however, is that the stash is
for _short-term_ storage, like when you need to temporarily clean the
working tree so that you can rebase.

Maybe the main use-case for beginners for the stash is to move
uncommitted changes to a different branch?  In that case we could have

            gi-move-uncomitted-changes target

that does that, by detaching HEAD, committing, attaching HEAD to
`target`, and cherry-picking the changes.  What if the cherry-pick
fails?

Maybe all uses of `git-stash` are obviated by Dustin Boswell's idea for
autostashing.  If so, great!

Note: Dustin's idea was that if you try to switch branches with a
dirty working tree, instead of complaining, Git should just save the
state of the working tree, and restore it when you come back to the
branch.  (This is pretty much what you described above under “Dirty
working tree”.)

### `gi-add` <paths>...

Copies files from the working tree to the index, like `git-add`.  We
support `-u` also.

Beginners are confused by this.  Maybe eparate `gi-add`, for adding *new*
files, from `gi-stage`, for copying changes to the index.  Does `gi-add`
imply `gi-stage`, or is it like `git-add -N`?  Probably avoid `git-add -N`
since it's still a little bit weird and unfinished.

Calling this `gi-stage` violates the commandment above about confusing
index/stage/cache terminology.  Maybe `gi-index` instead?  I like this;
it makes clear that `gi-reset-index` is the opposite of `gi-index`.

## Totally new commands

All of these are firmly in the "maybe, maybe not" file.

### `gi-???`

Something to display the topology of the commit history without
the details.  It elides sequences of commits that are straight lines.

I desperately wanted this when I was a beginner, and I still want it
frequently, although I've gotten used to not having it.  Maybe this is
MJD bias again?

Maybe package it up with something like your current `git-vee`?  They
make sense separately and in conjunction.  I still don't know how
anyone survives day-to-day without `git-vee`.

“`gi-map`” maybe?

### `gi-wtf`

Print a more detailed explanation of the previous error/warning.

Do it journalist style: Most important paragraph first. If
there's more to say, end with “OK to stop here, or use `gi wtf`
again to continue”.  Last paragraph is always “For complete
details, see `http://gi-wtf.org/gi/blah`”.

### `gi-whatnext`

Suggestion for what to do next?  If the index is conflicted,
suggest `gi-reset-index` or editing the files followed by `gi-index`.

Samer Masterson suggests: “reminds me of (Emacs) `vc-next-action`, which
simply does the next thing instead of asking”.  Look into this.

### `gi-undo`

Each gi command can write a history file that includes a command
that will perfectly roll back its changes.  `gi-undo` can execute
the rollback.  This would also allow `gi-history` which describes
just your `gi` commands without all the intervening `ls`es and
such.

Potential difficulty: Changing working directory and other
environmental changes.  Maybe it's as simple as having each undo
command start with `cd Whatever-cwd`.

### `gi-derp`

This means "I don't know what is going on, or Gi said something that makes no sense."

It pops up the editor with a feedback form.

We'll unfortunately have to phase this out as Gi usage grows.  Or
maybe turn it into an entry into the ticketing system.

"Derp" may be offensive. Maybe find a better name.

## git commands that we will NOT pass through to gi

### `git-reset`  (what a fucking mess)

Instead we have gi-discard-changes, gi-reset-index, and gi-go.

### `git-pull`

Use `gi-fetch` plus `gi-merge`.  Have a `gi-pull` comand that
prints out a message that tells you to do that. (“Sorry, the
opposite of `push` is unfortunately `fetch`.”)

### `git-init`

Emphasizes that gi is not a different thing than git, but that
gi is a different way to manipulate *git* repositories.  To make
a repo, you use git-init, because you are making a *git* repo,
not some different thing that is called a gi repo.

### `git-tag`

Advanced users only.

# Other features:

## `gi mode explain-git-commands`

After each gi command, gi will explain which _Git_ commands it
was using to do the same thing.

## Other modes?

Are there other kinds of mode settings?  Yes!  Gi can have a
"beginner" mode and an "intermediate" mode.  In beginner mode, menus
are shorter and some behaviors are disabled.  (Pitfall to avoid: do
not have behaviors _change_ when going from beginner to intermediate
mode.)  For example, the "beginner" version of the "merge conflict"
prompt will omit the option of exiting to the shell and dealing with
the conflict markers.

I think beginner mode omits the index.  Beginners don't need the
index.  (This conflicts with your idea that the index is one of the
basic concepts that we emphasize.)

# Prior Art

## [Easy Git — git for mere mortals](https://people.gnome.org/~newren/eg/)

  > In short, Easy GIT is a single-file wrapper script for git, designed
  > to make git easy to learn and use.

  To do: read this and shamelessly exploit it.

  Has a list of “Other similar projects”, but they are maiunly focused
  on making Git look more like SVN (barf), Mercurial, or Darcs.

## [Reinventing Git interface](https://tonsky.me/blog/reinventing-git-interface/) (2014)

Prokopov and I see eye-to-eye on many issues:

> basic Git concepts can be explained in a matter of half an hour _on a
> whiteboard_, yet actually touching Git _on a computer_ takes you weeks to
> get used to.

> Git warns you _a lot_…  fact is, you cannot really destroy anything
> by doing any “potentially dangerous” operations…

> Understanding there’s no harm to be done eases things a lot.

> Under the hood, working copy may be treated differently, but for a user there’s no point to be aware of that distinction.

> Instead of staging changes to index, we’ll split WIP commit into two
> WIP and STAGED, and then rename STAGED to something official.

> proper UI should enable is to select, drag, copy and shuffle commits and branch pointers, including HEAD, _directly on the DAG tree_.

This article's vision is much clearer than mine on some issues:

* No warnings!  Just do what the user asked and provide an undo path.
  Git's basic structure should make this easy, since the previous
  state is still there.

* Deltas are intuitive, and delta manipulation is important.  But
  deltas are second-class citizens of Git.

* We should remove any manual branch syncing stuff.
  Always sync local branches state with remotes.

I should reread Prokopov's article carefully, and then reconsider
everything in this document to see if I have changed my mind.

## [Jujitsu](https://github.com/martinvonz/jj)

> Jujutsu is a Git-compatible DVCS. It combines features from Git
> (data model, speed), Mercurial (anonymous branching, simple CLI free
> from "the index", revsets, powerful history-rewriting), and
> Pijul/Darcs (first-class conflicts), with features not found in most
> of them (working-copy-as-a-commit, undo functionality, automatic
> rebase, safe replication via rsync, Dropbox, or distributed file
> system).
>
> The command-line tool is called `jj` for now because it's easy to
> type and easy to replace (rare in English). The project is called
> "Jujutsu" because it matches "jj".