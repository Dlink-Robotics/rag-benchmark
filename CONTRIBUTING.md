# Contributing to rag-benchmark

This guide explains how we work together in this repository. It sets the rules, and it also explains why each rule exists and which commands to type. Read it once from top to bottom. After that, section 2 is the part you will come back to.

If a word is new to you, look it up in the [glossary](#11-glossary) near the end.

## 1. The idea in one paragraph

We use a method called trunk-based development. There is one permanent branch, `main`, and it always holds the current agreed state of the benchmark. Nobody changes `main` directly. For every change you create a short-lived branch, do the work there, and open a pull request (a request to bring your branch into `main`). Another contributor reviews it. When it is approved, the pull request is squash merged: all its commits become one clean commit on `main`, and the branch is deleted. Then you start the next change from the new `main`.

## 2. The daily loop

These nine actions are the whole normal workflow. `<name>` is your own short name, `<topic>` says what the branch is for, and `<branch>` is the two together, `<name>/<topic>`.

1. **Pick or open an issue.** An issue is a written description of one piece of work. Assign it to yourself so nobody else starts the same thing. See [section 7](#7-issues).

2. **Update `main`.** Always start from the newest state, or your branch begins out of date.

   ```sh
   git switch main
   git pull
   ```

3. **Create a branch.** This gives you a private line of work that cannot disturb anyone else.

   ```sh
   git switch -c <name>/<topic>
   ```

4. **Commit small and often.** A commit is a saved snapshot with a message. Small commits are easy to describe and easy to undo.

   ```sh
   git status
   git add <files>
   git commit
   ```

   Add files by name. Avoid `git add .`, because it can pick up files that must not be in the repository (see [section 9](#9-what-never-goes-in-the-repository)).

5. **Push.** This uploads your branch to GitHub. The first time, `-u` links your local branch to the one on GitHub, so later a plain `git push` is enough.

   ```sh
   git push -u origin <branch>
   ```

6. **Open a pull request.** Use `gh`, the GitHub command-line tool, or the web page. After a push, the repository page on GitHub shows a "Compare & pull request" button.

   ```sh
   gh pr create
   ```

   If the work is not finished, open it as a draft: `gh pr create --draft`. When it is ready for review, run `gh pr ready`.

7. **Respond to review.** Answer every comment. Make the fixes as new commits on the same branch and push again. Resolve each conversation once it is settled. See [section 6](#6-reviews).

   ```sh
   git add <files>
   git commit
   git push
   ```

8. **Squash merge.** When the pull request has one approval and all conversations are resolved, merge it. Use the "Squash and merge" button on the pull request page, or:

   ```sh
   gh pr merge --squash
   ```

   GitHub deletes the branch on GitHub for you.

9. **Delete the local branch and pull `main`.**

   ```sh
   git switch main
   git pull
   git branch -D <branch>
   ```

   The capital `-D` is needed here. A squash merge creates one new commit on `main`, so git cannot see that your branch was merged and the lowercase `-d` refuses. Check that the pull request shows "Merged" before you run it.

### If `main` moved while you were working

Someone else's pull request may be merged before yours. Bring those changes into your branch like this:

```sh
git switch main
git pull
git switch <branch>
git merge main
git push
```

If git reports a conflict, it means both sides changed the same lines. Open the files it lists, keep the correct lines, then `git add` those files and `git commit`.

## 3. Branch names

Write every branch name as `<name>/<topic>`.

- `<name>` is your own short name. `<topic>` says what the branch is for.
- Use lowercase letters, digits and hyphens. No spaces, no underscores.
- When the work belongs to a roadmap step, start the topic with it.

Examples:

- `<name>/step-3-run-and-record`
- `<name>/step-1-label-check-script`
- `<name>/fix-timer-units`

Why the prefix: it shows at a glance whose branch it is, and it keeps the branch list easy to read. The rule is the same for everyone: use `<name>/<topic>`.

Branches are short-lived. A branch should live for days, not weeks. It is deleted after the merge. Never reuse a merged branch: its history no longer matches `main`, and new work on it causes confusing conflicts. Start a new branch from the updated `main` instead.

## 4. Commits

A commit message has a subject line and, when needed, a body.

- **Subject:** `<type>: <short description>`, under 72 characters. The `type` is one of `feat`, `fix`, `docs`, `test`, `perf`, `ci`, the same list as for issue titles (see [section 7](#7-issues)). The description is lowercase English in the imperative, the form you use to give an order: "add", "fix", "remove". A test: the subject should complete the sentence "If applied, this commit will ...". No full stop at the end.
- **Body:** leave one blank line after the subject, then explain WHY the change is made when that is not obvious. The diff already shows what changed. It cannot show the reason.
- **One logical change per commit.** Do not mix a bug fix with a rename.

| Good subject | Bad subject | What is wrong with the bad one |
|---|---|---|
| `docs: add checksum file for the Japanese documents` | `update` | Says nothing about what changed, no type |
| `fix: report per-stage timer in milliseconds` | `fixed stuff` | Not specific, not imperative |
| `feat: record chunk rank and score for each question` | `WIP` | Describes your state, not the change |
| `feat: pin the embedding model to an exact version` | `Changes after review and also some cleanup and a new script` | Several changes in one commit, no type |

An example with a body:

```text
feat: pin the embedding model to an exact version

A rolling alias can change the model without notice. Results from
two runs would then not be comparable.
```

Two more rules:

- Do not add an agent or AI name as co-author in a commit message.
- If a `CHANGELOG.md` file exists, never edit it by hand.

Because pull requests are squash merged, the commits on your branch do not reach `main` one by one. They still matter: the reviewer reads them, and they are your own undo points.

## 5. Pull requests

A pull request is where a change is explained, checked and agreed. The template fills in the headings for you.

- **One change per pull request.** Normally one pull request relates to one roadmap step or one sub-step from the [README](README.md).
- **Keep it small.** Aim for under about 300 changed lines when possible. If it grows beyond that, split it.
- **Open it as a Draft while it is unfinished.** A draft pull request is visible to everyone but is marked as not ready to merge. It is a good way to show early work and ask for direction.
- **Link the issue.** Write `Closes #N` in the description, where N is the issue number. GitHub then closes the issue automatically when the pull request is merged.
- **Give the numbers.** When the change affects a measured result, put the quality and speed numbers in the description, and name the machine they were measured on. Numbers from a laptop and from the target device are not comparable.
- **Say how to check it.** Describe what a reviewer can run or read to confirm the change does what you say.

The title of the pull request becomes the subject of the single commit on `main`. Write it like a commit subject (see [section 4](#4-commits)).

## 6. Reviews

Everyone reviews everyone. Review is not a test of the author. It is a second pair of eyes on the change, and it is how all three of us stay informed about the whole project.

### When you are the reviewer

- **Review against the issue and the numbers.** Read the issue's "Done when" line first, then check that the pull request meets it. If the pull request reports numbers, check that they are there, that the machine is named, and that they support the claim. Do not block a pull request over style preferences.
- **Ask questions freely.** "Why is this value 3?" is a useful review comment. To comment, open the "Files changed" tab and click the `+` next to a line.
- **Finish with a clear decision.** Click "Review changes" and choose one:
  - **Comment:** you have questions or notes, and no decision yet.
  - **Approve:** the change does what the issue asks and you see no problem.
  - **Request changes:** something must be fixed before merging. Say exactly what.
- From the command line, `gh pr checkout <number>` gets the branch onto your machine so you can run it. `gh pr review <number> --approve` approves it.

### When you are the author

- **You cannot approve your own pull request.** Open it, ask a teammate to review, and wait. GitHub does not count the author's approval, and the rule on `main` needs 1 approval from someone else. Pushing a commit to a teammate's branch and then approving it does not work either, because the person who pushed last cannot be the approver.
- **Reply to every comment**, even if only with "Done" or "I left it, because ...".
- **Push fixes as new commits** on the same branch.
- **Resolve each conversation** once it is settled, using the "Resolve conversation" button. GitHub will not merge while any conversation is open.
- **Ask for review again after you push.** A new push dismisses the earlier approval. This is on purpose: an approval covers only the code the reviewer actually saw.

### Response time

Aim to respond within two working days, as reviewer and as author. If you cannot, say so on the pull request, so the other person can plan or ask someone else.

### How to disagree

Disagreement is normal and useful. Keep it about the change, not the person. Decide quickly: if two or three replies do not settle it, talk directly, bring in the third contributor if needed, and pick one option. Then write the decision and the reason on the pull request, so the record is there for later readers.

## 7. Issues

An issue describes one piece of work before it starts. Writing it first forces a clear goal, and gives the reviewer something to check against.

- **One issue per roadmap step or sub-step.** If an issue covers more, split it.
- **Title format `<type>: <short description>`.** The `type` is lowercase and one of `feat` (new feature or capability), `fix` (bug fix), `docs` (documentation), `test` (adding or updating tests), `perf` (performance improvement) or `ci` (automation workflows). Put one space after the colon. Write the description in English, in the imperative, with no full stop, and keep the whole title around 72 characters or fewer. Good: `fix: correct recall at k calculation`. Too vague: `fix: bug`. Pull requests and commits use the same format.
- **Sections.** The issue form asks for them:
  - **Goal and why:** what should exist afterwards, and why it is needed.
  - **Done when:** a statement someone else can check. "The script exits with an error if any answer text is not at its labelled position" is checkable. "The script works well" is not.
  - **Out of scope:** what this issue does not cover. This stops the work from growing.
  - **Size:** half a day, 1 day, 2 days. If it is more than 2 days, split it.
- **Assign yourself when you start.** An assigned issue means "someone is working on this".
- **Mark it blocked when it is.** Add the `blocked` label and write what it is blocked by, for example another issue number or an open decision in [docs/roadmap.md](docs/roadmap.md).

Open a new issue with the "New issue" button and choose the task form or the bug form.

## 8. Labels

A label is a small tag on an issue or pull request. Labels make the list easy to filter.

| Label | Kind | Meaning |
|---|---|---|
| `task` | type | A planned piece of work on the benchmark |
| `bug` | type | Something that is there but behaves wrongly |
| `question` | type | An answer is needed before work can continue |
| `blocked` | status | Work cannot go on until something else is done. Say what |
| `step-0` ... `step-8` | step | The roadmap step this belongs to, numbered as in the README |

The issue forms add `task` or `bug` for you. Add the `step-N` label yourself.

## 9. What never goes in the repository

This repository is public. Anything committed can be read and copied by anyone, and git keeps it in the history even after you delete the file.

Never commit:

- **The held-out test questions.** Only their checksum (a short fingerprint of the file) is committed. If the questions become public, the final test no longer proves anything.
- **API keys.**
- **`.env` files.**
- **Personal data.**
- **Large model files.** Write the model name and exact version in the config instead.
- **Machine-specific paths**, such as a path inside your own home folder. They break on every other machine.

Before each commit, run `git status` and `git diff --staged` and read what you are about to save. If something from this list was pushed by mistake, tell the other contributors at once. Deleting the file in a later commit does not remove it from the history, and a key that was pushed must be treated as leaked.

## 10. What GitHub enforces

Some rules are not left to memory. GitHub applies them automatically, and nobody can bypass the ones below. They are set in the repository settings under Rules. If this section and the settings ever disagree, the settings are right and this guide should be fixed.

**On `main`:**

- Changes arrive only through a pull request.
- The pull request needs 1 approval from someone other than the author. You cannot approve your own pull request.
- The person who pushed last cannot be the approver.
- A new push dismisses earlier approvals.
- All review conversations must be resolved before merging.
- Force pushes to `main` and deletion of `main` are blocked.

**On merging:**

- Only squash merging is allowed: one clean commit per pull request.
- The branch is deleted automatically after the merge.

### If a push or a merge is refused

Read the message. It tells you which rule applied. It is almost always one of these:

| What you tried | What it means | What to do |
|---|---|---|
| Push to `main` | "Open a pull request" | Create a branch, push it, open a pull request |
| Push a branch with another name | The name must be `<name>/<topic>` with your own name | Create a correctly named branch and push that |
| Merge a pull request | "You need a review" | Ask a reviewer, or ask again if you pushed after the approval |
| Merge an approved pull request | A conversation is still open | Reply to it and resolve it |

Never try to work around a refusal. The rules are the reason `main` can be trusted. If you think a rule is wrong, open an issue and discuss it.

## 11. Glossary

| Word | Meaning |
|---|---|
| Branch | A separate line of work with its own name. Changes on it do not affect other branches |
| Commit | A saved snapshot of your changes, with a message that describes it |
| Push | Upload your commits from your machine to GitHub |
| Pull request | A request to bring the commits of one branch into another, here into `main`. Often shortened to PR |
| Review | Another contributor reads the pull request, asks questions and gives a decision |
| Approve | The reviewer's decision that the pull request can be merged |
| Squash merge | Merging a pull request as one single commit, whatever number of commits the branch had |
| Issue | A written description of one piece of work or one problem |
| Label | A small tag on an issue or pull request, used to sort and filter |
| Milestone | A named group of issues and pull requests that belong to one goal, with a progress bar |
| Fork | Your own copy of a repository on GitHub. People without write access use one to propose changes. With write access you do not need one |
| Draft pull request | A pull request marked as not ready. It can be read and commented on, but not merged |

## 12. Learning notes

> **Why these habits matter**
>
> - **Small pull requests get better reviews.** A reviewer can really read 100 lines. At 1000 lines, people skim and approve.
> - **Squash keeps history readable.** Each line in the history of `main` is one complete, reviewed change.
> - **Protected `main` means the baseline never silently changes.** Every benchmark number is compared to an earlier one. That only works if nobody can alter the base without a review.
> - **Short-lived branches avoid conflicts.** The longer a branch lives, the further it drifts from `main`.
> - **Issues with a "Done when" line end arguments early.** Author and reviewer agree on the target before the work starts.
> - **Commit messages that say why save time later.** The code shows what was done. Only the message can say what the reason was.
> - **Numbers with a machine name can be compared.** A timing without its machine cannot be checked or repeated.
