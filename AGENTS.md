# AGENTS.md

Read [README.md](README.md) and [GUIDE.md](GUIDE.md) first. `main` holds the readable edition of
the proof, and it already builds in reasonable time. What we want now is a proof that people can
read and check. A change is welcome when it makes the proof clearer or connects it to Mathlib and
Tau Ceti. It is not welcome when it makes the code shorter or faster and harder to read.

If you have agent time to spare and no particular change in mind, the GRH roadmap of
[Tau Ceti](https://github.com/TauCetiProject/TauCeti) is a better place for it. Tau Ceti rebuilds
this proof as a library to Mathlib's standards, and its review process is set up for agent
contributions.

## Building

Fetch the cache before anything else:

```bash
lake exe cache get
lake build
```

Lake rebuilds the files you changed and every file that imports them, so this also catches a
lemma you removed that is still used somewhere else. It must finish without errors. Don't report
your own build or Comparator results in a pull request: we build pull requests ourselves.

## Changes we want

- Using a result from Mathlib or Tau Ceti in place of our own proof of it, and deleting ours.
  Check that the statements really agree, hypotheses included.
- Deleting declarations that nothing uses.
- Proving once, as a lemma, an argument that appears several times, and using it everywhere.
- Splitting a long proof at its natural steps into lemmas, each with a name and a docstring that
  say what it proves.
- A sharper or more general version of a lemma, if its proof is about as easy as the old one.
  Move the callers to it and delete the old statement; don't keep it as a corollary.
- Fixing linter warnings.
- Correcting a docstring or comment that says something wrong about the code.
- Fixing an open issue.

## Changes we don't want

### Inlining lemmas

Don't inline a lemma into its only caller, and don't fold helper lemmas into one long proof. Being
used once is not a reason to remove a lemma: its name and docstring tell the reader what that step
proves, and the file's header often lists it. Like Mathlib, we would rather have a long proof
split into lemmas than the reverse.

### Cutting statements down to what the callers use

Don't cut an `iff` down to the direction the proof uses, or drop conclusions from a lemma, only
because no caller needs them. The full statement is usually the one a reader expects to find.
Deleting a lemma that nothing uses at all is fine.

### Trading readability for speed or length

Don't replace `simp` with a long `simp only [...]` list, unfold `nlinarith` or `grind` into many
hand-written steps, or squeeze several readable steps into one opaque tactic call, to save a
second or a few lines. A speed-up is welcome when the time saved is large and the proof reads as
well as before. When a step is slow, find the cause, such as a slow instance, and fix it where it
is declared.

### Hiding a problem in one file

These make one proof go through, or go faster, and leave the cause in place:

- `set_option backward.…` options. They bring back how an older Lean behaved, and later versions
  remove them.
- Tactics written with `elab` or `run_tac` that close a goal in place of `exact` or `rfl`.
- Switching instances off with `attribute [-instance]`, adding a local instance or a shortcut such
  as `instance : CommRing R := inferInstance` only to make one file faster, or proving private
  copies of Mathlib lemmas so that an instance search never runs.

If you can't fix the cause, open an issue with the profile.

## Rules that always apply

- Don't change `Challenge.lean`, `src/Statements.lean` or `comparator.json`. CI rejects any pull
  request that does.
- Don't add axioms or `sorry`. `src/Main.lean` may use only `propext`, `Quot.sound` and
  `Classical.choice`.
- Don't rewrite the proof along a substantially different path, or add files or lemmas the proof
  does not need.
- Don't add or change a reference to the literature unless you have checked it against the
  source. Say in the pull request which edition you checked, and give the theorem number. A wrong
  reference is worse than none.

## Pull requests

Keep each pull request to one change. Fixing linter warnings in the files you edit doesn't count
as a second change.

If you want to make the same kind of change in many places, open one pull request first and wait
until a maintainer has said we want it before you open the others. That way a pattern we don't
want costs one pull request, not a hundred.

Don't open a pull request until it compiles. Before then, open it as a draft
(`gh pr create --draft`).

Titles follow [Mathlib's convention](https://leanprover-community.github.io/contribute/commit.html),
`<type>(<optional-scope>): <subject>`, where the scope is a directory or file without `src/`. For
example:

```
refactor(Lib/Adelic/Hecke): use Tau Ceti for stabiliser orders
refactor(New/Spanning/RemainingTerms): share the second factor rank bound
```

In the description, say in two or three sentences what the change does and why it makes the proof
easier to read. Name any declarations it removes, renames or moves.

## Review

A pull request is merged only after a reviewer has approved it on its current head commit.
Reviewers are the people with the Triage role on this repository, or more.

When a reviewer hands a pull request back to you, it gets the `awaiting-author` label. Once you
have made the changes or answered the question, comment `ready` on the pull request, with nothing
else in the comment, to put it back in the queue.

## Batches

If you assemble a batch, take only pull requests that a reviewer has approved on their current
head commit. Leave out those labelled `awaiting-author`, those a maintainer has asked to hold, and
any that break a rule in this file. Merge each with its own merge commit, such as
`Merge PR #123 into batch`. If a batch already holds a pull request that should be left out,
rebuild the batch without it and force-push; don't revert the merge commit, or GitHub will mark
the pull request merged although its changes did not land. List the pull requests you left out in
the batch's description, with the reason for each.
