# Git Workflow Tutorial

In general, our `git` workflow is a variant of the _feature branch workflow_, which is described [here](https://www.atlassian.com/git/tutorials/comparing-workflows#feature-branch-workflow).
Basically, the main idea of this workflow is that there is one mainline repository (`master`) that should always work, and all bug fixes, code updates, and feature additions are worked on in separate `git` branches.
These branches are made off of `master` and worked on by one person, and once the update is ready, the code is reviewed, rebased onto `master`, and then merged into `master`.

Here's an overview of how this workflow would work in practice.

## Step 1: Create a new feature branch

Run the following commands in the `git` repository:

```bash
git fetch
git checkout master
git pull
git checkout -b my-feature-branch
git push --set-upstream origin my-feature-branch
```

Here, replace `my-feature-branch` with the name of your branch.
The name should be short and descriptive, succinctly describing what your branch is intended to do.
Example names include `trimesh-ray-tracer` or `kinect-interface-bugfix`.

Then, attach the corresponding Trello card to your branch on Github.
Click on the ticket, and in the right bar, find the GitHub Power-Up button (if you don't see it, enable it in the main sidebar power-ups tab).
Click "Attach Branch", and then navigate to the repo and branch you want to attach to the card.

## Step 2: Do your work

Make your changes on your feature branch, committing and pushing as necessary.
To commit to your branch, simply use the following commands:

```bash
git add <filenames here>
git commit
git push
```

## Step 3: Incorporate changes from master regularly

As you work, other people might be making changes to `master`.
In order to keep code current while you work on your feature branch, it's good practice to regularly rebase your branch on top of `master`.
Essentially, this unwinds all of your commits to your branch back to where it split from `master`, applies all new commits to `master`, and then replays your commits back over this updated `master`.
More information about rebasing can be found [here](https://www.atlassian.com/git/tutorials/merging-vs-rebasing).
After committing and pushing your changes to your feature branch, run the following commands:

```bash
git fetch
git checkout master
git pull
git checkout my-feature-branch
git rebase master
git push --force
```

## Step 3: Revise your commit history

Once you're done with your work, it's good practice to revise your commit history to clean up extraneous commits and create a clean, cohesive history with logical, self-contained commits.
First, make sure you've pushed all of your changes and rebased onto `master`.
Then, run the following command:

```bash
git rebase -i $(git merge-base my-feature-branch master)
```

You should see a `vi` window that pops up that looks like this:

TODO ADD PHOTO

You can pick, reword, edit, or squash commits by editing the word in front of each of the commits on your feature branch.
Once you write the file, `git` will guide you through the process of making the changes you wanted.

Alternatively, you can rewrite your entire commit history freshly by unstaging all of your changes and then making logical commits (this is my preferred method).
To do this, run the following command.

```bash
git reset $(git merge-base my-feature-branch master)
```

Then, add changes and commit them in logical, linear commits.
If you want to split changes in one file into multiple commits, just add the parts you want to each commit using `git add -p`.

Once you've rebased or reset and changed all your commits, you have to force-push onto your branch:

```
git push --force
```

Each commit in your branch should be self-contained (i.e. we should be able to undo any one of them without breaking the code).

## Step 4: Create a pull request on Github

Once your history is clean and your branch is ready to be merged, rebase onto master again and then create a pull request for your branch on Github.
Navigate to the "Pull requests" tab, and create a new request. The base branch should be `master`, and the compare should be your feature branch.

Then, add the pull request to the Trello card in the same way that you added the feature branch.

## Step 5: Merge into master

Once you get approval to merge your pull request, do it via the following commands.

```bash
git fetch
git checkout master
git pull
git checkout my-feature-branch
git rebase master
git push --force
git checkout master
git merge my-feature-branch
git push
```

This will merge your branch into master cleanly, with no merge commit.

