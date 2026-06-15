# Git Workflow: From `git init` to a Merged Pull Request

A hands-on, step-by-step walkthrough of a complete Git workflow. You'll use the **command line** for the Git side — creating a repository, making and inspecting changes, committing, and pushing — and **GitHub's website** (the web UI) for the GitHub side: creating the repo, opening a pull request, reviewing, and merging. Together they make up the branch → pull request → merge cycle that real teams use every day.

This guide assumes you've read the Git and GitHub section of [01-basics.md](01-basics.md), have `git` installed, and have a free GitHub account you can sign into in your browser. Work through it in a throwaway folder — everything you do locally is safe to run and easy to delete afterwards.

> **Why command line for Git but the website for GitHub?**
> They're two different things (see [01-basics.md](01-basics.md)): **Git** is the version-control program on your laptop, and the command line is the most direct way to drive it. **GitHub** is a website that hosts a copy of your repo and adds collaboration features on top. Everything GitHub does has a point-and-click interface in the browser, which is the friendliest way to learn it — so that's what we'll use here. (There's also a `gh` command that does the GitHub steps from the terminal, but we'll stick to the web UI.)

> **Before you start: who are you?**
> Git stamps your name and email onto every commit. If you've never used Git on this machine, set them once (globally, for all repos):
> ```
> git config --global user.name "Your Name"
> git config --global user.email "you@example.com"
> ```
> Use the same email as your GitHub account so your commits link back to your profile.

---

## 1. Create a folder and turn it into a repository

First, make a folder for the project and move into it:

```
mkdir git-practice
cd git-practice
```

Now turn that ordinary folder into a Git repository:

```
git init -b master
```

> **What just happened?**
> `git init` creates a hidden `.git` subfolder inside `git-practice`. That folder *is* the repository — it holds the entire history. Your visible files are just the current snapshot; everything Git knows lives in `.git`.
>
> The `-b master` flag names the starting branch `master`. Without it, the branch name depends on your Git version's default, which may be `master` or `main`. Naming it explicitly means the rest of this guide works the same on any machine.

You can confirm it worked by listing hidden files — you should see `.git`:

```
ls -a
```

---

## 2. Create a file with `echo`

Instead of opening a text editor, you can create a small file straight from the command line. The `echo` command prints text, and the `>` operator redirects that text into a file instead of the screen:

```
echo "My name is John" > about.txt
```

> **`>` versus `>>`**
> A single `>` **overwrites** the file (creating it if needed). Two of them, `>>`, **append** to the end of the file. A common beginner mistake is using `>` when you meant `>>` and wiping out your file's contents. When in doubt, `>>` is the safer one.

---

## 3. Check the file's contents with `cat`

`cat` prints a file's contents to the screen. Use it to confirm the file says what you expect:

```
cat about.txt
```

You should see:

```
My name is John
```

> **Why "cat"?**
> The name is short for *concatenate*. Its original job was to join multiple files together (`cat part1.txt part2.txt`), but its most common everyday use is exactly this: a quick peek at what's inside a single file without opening an editor.

---

## 4. Commit the file to the repository

Right now Git can *see* `about.txt`, but it isn't tracking it yet. Saving a snapshot is a two-step move: **stage** the changes you want, then **commit** them.

Check the current state:

```
git status
```

Git reports `about.txt` as an *untracked* file. Stage it:

```
git add about.txt
```

Then commit it with a message describing the change:

```
git commit -m "Add about file with my name"
```

> **Why two steps instead of one?**
> The **staging area** (also called the "index") is a holding pen for the changes you're about to commit. It lets you commit *some* of your changes and leave others for later — for example, committing a bug fix now while saving an unfinished experiment for a separate commit. For small changes it feels like extra typing, but on real projects the control is invaluable.

> **What makes a good commit message?**
> Write it in the imperative mood — "Add about file," not "Added" or "Adds" — as if completing the sentence *"This commit will…"*. It reads naturally in Git's history and matches the messages Git generates itself (e.g. "Merge branch…").

You can see your new commit in the project's history:

```
git log --oneline
```

---

## 5. Edit the file manually

Now change the file the normal way — open `about.txt` in any text editor (TextEdit, VS Code, whatever you like), replace **John** with **Joe**, and save. After saving, the file should read:

```
My name is Joe
```

Confirm the change landed with `cat` again:

```
cat about.txt
```

> **`echo` vs. editing by hand**
> Step 2 used `echo` because it's fast for tiny one-liners. For anything real you'll edit files in an editor. Git doesn't care *how* a file changed — it only compares the file's current contents to the last committed snapshot. The tool you used to make the edit is irrelevant.

---

## 6. See exactly what changed with `git diff`

Before you commit an edit, it's good practice to review precisely what you changed. That's what `git diff` is for:

```
git diff
```

You'll see something like this:

```
diff --git a/about.txt b/about.txt
index 1c2d3e4..5f6a7b8 100644
--- a/about.txt
+++ b/about.txt
@@ -1 +1 @@
-My name is John
+My name is Joe
```

> **How to read a diff**
> The lines that matter most are at the bottom:
> - A line starting with `-` (often shown in red) was **removed**.
> - A line starting with `+` (often shown in green) was **added**.
>
> So this diff says: the line "My name is John" was removed and "My name is Joe" was added — i.e. you changed one word. Git has no concept of "edited a line"; it represents every change as a removal plus an addition.
>
> The `@@ -1 +1 @@` part is a location marker telling you *where* in the file the change happened (line 1, in this case).

> **A key subtlety: `git diff` shows *unstaged* changes**
> By default, `git diff` compares your working files against what's currently *staged*. Once you `git add` a file, plain `git diff` shows nothing for it — because there's no difference between the file and the staging area anymore. To see what you've staged and are about to commit, use:
> ```
> git diff --staged
> ```
> Get in the habit of running `git diff` before staging and `git diff --staged` before committing. It's the cheapest way to avoid committing something by accident.

---

## 7. Commit the edit

Now that you've reviewed the change, stage and commit it:

```
git add about.txt
git commit -m "Change name from John to Joe"
```

Check the history again — you should now have two commits:

```
git log --oneline
```

> **Commits build a chain**
> Each commit points back to the one before it, forming an unbroken chain of snapshots. This is why Git can show you the project at any point in time and why you can safely undo changes: nothing is ever really lost, it's just earlier in the chain.

---

## 8. Create a matching repository on GitHub (web UI)

So far everything lives only on your laptop. To back it up and collaborate, you need a copy on GitHub. Create the empty repo on the website:

1. Sign in at [github.com](https://github.com).
2. Click the **+** button in the top-right corner and choose **New repository**.
3. For **Repository name**, type `git-practice`.
4. Choose **Private** (only you can see it) or **Public** (visible to everyone) — either is fine for practice.
5. **Leave every "Initialize this repository" option unchecked** — do *not* add a README, `.gitignore`, or licence.
6. Click the green **Create repository** button.

> **The most important checkbox: leave the repo empty**
> It's tempting to tick "Add a README." Don't. You already have commits on your laptop, and you're about to connect them to this repo. If GitHub also creates a commit (a README), the two histories will have diverged and your first push will be **rejected** with an error about unrelated histories. Starting with a completely empty repo on GitHub avoids that headache entirely.

After you click Create, GitHub shows a setup page with several commands. The section you want is titled **"…or push an existing repository from the command line"** — it contains the exact two commands used in the next steps, pre-filled with *your* repository's address. Keep that page open.

---

## 9. Connect your local repo to GitHub (the remote)

You now have two separate repositories — one on your laptop, one (empty) on GitHub — and they don't know about each other yet. A **remote** is a named bookmark that points your local repo at the GitHub copy. The conventional name for it is `origin`.

Back on the GitHub setup page from the previous step, copy the address shown for your repo (it looks like `https://github.com/your-username/git-practice.git`), then run:

```
git remote add origin https://github.com/your-username/git-practice.git
```

Confirm it worked:

```
git remote -v
```

You'll see two lines (a fetch URL and a push URL) both pointing at your new GitHub repo.

> **`origin` is just a nickname**
> There's nothing magic about the word "origin" — it's a convention, an alias for that long URL so you can type `git push origin` instead of the full `https://github.com/...` address every time. A repo can have several remotes (for example, your own fork plus the original project you forked from).
>
> The GitHub setup page literally prints this exact `git remote add origin …` line for you — you can copy it instead of typing the URL by hand.

---

## 10. Push your commits to GitHub

Now send your local commits up to GitHub:

```
git push -u origin master
```

> **What's the `-u` for?**
> `-u` (short for `--set-upstream`) links your local `master` branch to the `origin/master` branch on GitHub. After doing it once, Git remembers the connection, so from then on you can just type `git push` and `git pull` with no extra arguments. You only need `-u` the first time you push a new branch.

Refresh your repo's page on GitHub in the browser and you'll see `about.txt` and both commits now live online.

---

## 11. Create a new branch

Real work doesn't happen directly on `master`. Instead, you make a **branch** — a parallel line of development where you can make changes safely. Create one and switch to it in a single command:

```
git switch -c add-greeting
```

> **`switch -c` creates *and* moves you**
> The `-c` flag means "create." `git switch -c add-greeting` creates a branch called `add-greeting` and immediately makes it your current branch. Any commits you make now go onto `add-greeting`, leaving `master` untouched. You can confirm which branch you're on at any time with `git branch` — the current one is marked with a `*`.
>
> (You may see older guides use `git checkout -b` for this. `git switch` is the newer, clearer command that does the same thing.)

---

## 12. Add a new file and commit it to the branch

Create a second file on this branch:

```
echo "hello" > greeting.txt
```

Stage and commit it just like before:

```
git add greeting.txt
git commit -m "Add greeting file"
```

> **This commit lives only on `add-greeting`**
> If you switched back to `master` right now (`git switch master`) and ran `ls`, you would *not* see `greeting.txt` — it exists only on the branch where you committed it. This isolation is the whole point of branches: you can experiment freely without affecting the stable `master` version. Switch back with `git switch add-greeting` before continuing.

---

## 13. Push the branch to GitHub

Publish your new branch so it exists on GitHub too:

```
git push -u origin add-greeting
```

> **Branches are pushed independently**
> Pushing `master` earlier did *not* automatically push `add-greeting` — each branch is published separately. Again you use `-u` because this is the first time this particular branch has been pushed. GitHub now has both your `master` and your `add-greeting` branches.

---

## 14. Open a pull request (web UI)

A **pull request** (PR) is a formal proposal to merge one branch into another — here, to merge `add-greeting` into `master`. It's where code review and discussion happen before changes become official. Open one on the website:

1. Go to your repo's page on [github.com](https://github.com) and refresh it. Because you just pushed a new branch, GitHub shows a yellow banner: **"add-greeting had recent pushes"** with a green **Compare & pull request** button. Click it.
   - *(No banner? Click the **Pull requests** tab, then the green **New pull request** button, and choose `master` as the base and `add-greeting` as the branch to compare.)*
2. Check the two dropdowns at the top: **base: `master`** ← **compare: `add-greeting`**. This means "merge `add-greeting` *into* `master`."
3. Give it a **title** (e.g. `Add greeting file`) and a short **description** explaining the change.
4. Click the green **Create pull request** button.

> **Read the merge direction carefully**
> The two dropdowns are the heart of a PR: **base** is the branch you're merging *into* (the destination, `master`), and **compare** is the branch holding your changes (the source, `add-greeting`). Getting them backwards is the single most common PR mistake. GitHub shows a preview of the diff right below them, so you can confirm you're proposing exactly the changes you expect before creating the PR.

> **Why bother with a PR for your own repo?**
> On a solo project you *could* merge branches directly. But pull requests give you a tidy record of every change, a place to run automated checks (tests, linting) before merging, and the exact workflow you'll need the moment a second person joins. Practising it now on a throwaway repo means it's second nature later.

---

## 15. View the pull request (web UI)

Creating the PR drops you straight onto its page. You can always get back to it later from the **Pull requests** tab at the top of your repo. The page has a few tabs worth knowing:

- **Conversation** — the description, comments, review status, and the merge button.
- **Commits** — the list of commits the PR would add (here, just "Add greeting file").
- **Files changed** — the line-by-line diff. This is the same `+` / `-` view you saw with `git diff` in step 6, now rendered in the browser. It's where a reviewer reads and comments on the actual changes.

> **The PR page is a living document**
> If you push more commits to the `add-greeting` branch while the PR is open, they appear here automatically — the PR always reflects the latest state of the branch. That's why you open the PR early and keep working: reviewers see your updates as you go.

---

## 16. Approve and merge the pull request (web UI)

In a team, a colleague reviews and **approves** your PR before it's merged. They'd do it like this:

1. Open the PR and click the **Files changed** tab.
2. Click the **Review changes** button (top-right).
3. Choose **Approve**, optionally leave a comment, and click **Submit review**.

> **You usually can't approve your own PR**
> GitHub greys out the **Approve** option on a pull request you opened yourself — approval is meant to be a second pair of eyes. On a solo repo you'll simply skip the approval step and merge directly (a personal repo doesn't require a review unless you've turned on **branch protection rules** that demand one). The steps above are exactly what a teammate would do to approve *your* work.

Once the PR is approved (or, on a solo repo, whenever you're ready), merge it from the **Conversation** tab:

1. Click the **Merge pull request** button. (Use the small ▾ arrow beside it to pick a merge style first — see below.)
2. Confirm by clicking **Confirm merge**.
3. After merging, GitHub shows a **Delete branch** button — click it to remove the finished `add-greeting` branch.

> **The merge styles worth knowing (the ▾ dropdown)**
> - **Squash and merge** combines all the commits on the branch into a single tidy commit on `master`. This is the most popular default because it keeps `master`'s history clean and readable.
> - **Create a merge commit** keeps every commit plus an extra "merge commit" that ties the branch back in.
> - **Rebase and merge** replays the branch's commits onto `master` with no merge commit.
> For this guide, **Squash and merge** is the one to pick.

> **Why delete the branch?**
> Once merged, `add-greeting` has done its job — its changes now live in `master`. Branches are cheap and meant to be temporary, so deleting it keeps your repo's branch list uncluttered. Nothing is lost: the commits are safely part of `master`'s history.

---

## 17. Bring your local `master` up to date

The merge happened on GitHub, so your laptop's `master` doesn't know about it yet. Switch back to `master` and pull the merged changes down:

```
git switch master
git pull
```

> **Closing the loop**
> After this `pull`, run `ls` and you'll finally see `greeting.txt` on your local `master` — its journey is complete. You deleted the branch *on GitHub* via the web UI in the previous step; tidy up the leftover local copy with `git branch -D add-greeting`. (Use a capital `-D` here: because you squash-merged, Git doesn't recognise the local branch as "fully merged" and a lowercase `-d` would refuse to delete it.) Your local repo and GitHub are now back in sync, and you're ready to branch off and do it all again for the next change.

---

## The full cycle at a glance

Once it's familiar, the everyday loop is short and rhythmic:

1. `git switch -c my-change` — branch off `master`. *(command line)*
2. Edit files. Use `git diff` to review. *(command line)*
3. `git add` and `git commit` — snapshot your work. *(command line)*
4. `git push -u origin my-change` — publish the branch. *(command line)*
5. On **github.com**, click **Compare & pull request** to propose the change. *(web UI)*
6. **Review**, **approve**, then **Squash and merge** and **Delete branch**. *(web UI)*
7. `git switch master && git pull` — sync up, then repeat. *(command line)*

The pattern to remember: **Git work happens on the command line; GitHub work happens in the browser.**

That's the same cycle used on projects with thousands of contributors. The commands don't change as the team grows — only the number of people running them does.
