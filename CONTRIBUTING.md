# Contributing to Oil Spill Detection

Thanks for helping out! This guide walks you through everything from first-time
setup to getting your changes merged.

**How it works:** you can't push to this repo directly. You work on your own
copy (a **fork**) and send changes as a **Pull Request (PR)**. A maintainer
(@ashketchum7182 or @adeeb-m-hub) reviews it and merges it into `main`.

---

## Part 1: One-time setup (about 10 min)

### Step 1: Install Git

- **Windows:** download it from https://git-scm.com/download/win and install
  with the default options. This also installs **Git Bash**, which you should
  use for all the commands below.
- **Mac:** open Terminal and run `git --version`. If Git isn't installed, it
  will prompt you to install it.

Check it worked:

```bash
git --version
```

### Step 2: Tell Git who you are

Use the same email as your GitHub account:

```bash
git config --global user.name "Your Name"
git config --global user.email "your-github-email@example.com"
```

### Step 3: Fork the repo

1. Log in to GitHub.
2. Open https://github.com/ashketchum7182/Oil-Spill-Detection
3. Click **Fork** (top right) → leave the defaults → **Create fork**.
4. You now have your own copy at
   `https://github.com/YOUR_USERNAME/Oil-Spill-Detection`.

### Step 4: Clone your fork to your computer

Replace `YOUR_USERNAME` with your GitHub username:

```bash
cd Desktop
git clone https://github.com/YOUR_USERNAME/Oil-Spill-Detection.git
cd Oil-Spill-Detection
```

The first time you push, a browser window will open asking you to log in to
GitHub. Log in, and Git will remember you.

### Step 5: Link to the original repo ("upstream")

This lets you pull in everyone else's merged work:

```bash
git remote add upstream https://github.com/ashketchum7182/Oil-Spill-Detection.git
git remote -v
```

You should see **origin** (your fork) and **upstream** (the main repo).

### Step 6 (optional): Set up Python

You need **Python 3.11, 3.12, or 3.13**. Python 3.14 will fail with
`No matching distribution found for scikit-learn==1.6.1`. Check your version
with `python --version`. If you have several versions installed on Windows,
pick one with `py -3.13 -m venv venv` instead of the first line below.

```bash
python -m venv venv
source venv/Scripts/activate   # Windows (Git Bash)
source venv/bin/activate       # Mac/Linux
pip install -r requirements.txt
```

The dataset (about 490 MB) is **not** in the repo. See the
[Dataset](README.md#dataset) section of the README to download it.

✅ **Setup done. You only do Part 1 once.**

---

## Part 2: Every time you work on something

### Step 1: Get the latest code

```bash
git checkout main
git pull upstream main
git push origin main
```

This updates your local `main` and your fork to match the main repo.

### Step 2: Create a new branch for your task

Never work on `main`. Name the branch after what you're doing:

```bash
git checkout -b feature/short-description
```

Examples: `feature/add-sample-images`, `fix/geolocation-bug`,
`docs/update-readme`

### Step 3: Make your changes

Edit files in VS Code or any editor. Check what changed:

```bash
git status
```

### Step 4: Save your changes (commit)

```bash
git add .
git commit -m "Short clear description of what you changed"
```

- Good message: `Add confidence score to AIS output`
- Bad message: `changes`, `update`, `final`

### Step 5: Push the branch to your fork

```bash
git push -u origin feature/short-description
```

### Step 6: Open a Pull Request

1. Go to your fork: `https://github.com/YOUR_USERNAME/Oil-Spill-Detection`
2. A yellow **"Compare & pull request"** banner will appear. Click it.
3. Check the top bar reads:
   - **base repository:** `ashketchum7182/Oil-Spill-Detection`, **base:** `main`
   - **head repository:** `YOUR_USERNAME/Oil-Spill-Detection`,
     **compare:** `feature/short-description`
4. Fill in:
   - **Title:** what the PR does
   - **Description:** what you changed, why, and how you tested it
5. Click **Create pull request**.

### Step 7: Wait for review

- A maintainer will review it.
- If they **approve**, they merge it. You're done 🎉
- If they **request changes**, go to Part 3.

---

## Part 3: If changes are requested on your PR

Stay on the **same branch**, fix the code, then:

```bash
git add .
git commit -m "Address review comments"
git push
```

The PR updates automatically. Don't open a new PR.

---

## Part 4: After your PR is merged

Clean up and get ready for the next task:

```bash
git checkout main
git pull upstream main
git push origin main
git branch -D feature/short-description
```

(PRs are squash-merged, so use `-D`; `-d` will refuse and say the branch
isn't merged.)

Then start the next task from **Part 2, Step 2**.

---

## Part 5: Fixing common problems

### "Your branch is behind" or a merge conflict

```bash
git checkout feature/short-description
git pull upstream main
```

If there's a conflict, open the files it lists and look for:

```
<<<<<<< HEAD
your version
=======
their version
>>>>>>> main
```

Keep the correct code, delete the `<<<<<<<`, `=======` and `>>>>>>>` lines,
then:

```bash
git add .
git commit -m "Resolve merge conflict"
git push
```

### "Permission denied" or 403 when pushing

You're probably pushing to the main repo instead of your fork. Check:

```bash
git remote -v
```

`origin` must point to `YOUR_USERNAME/Oil-Spill-Detection`. If it doesn't:

```bash
git remote set-url origin https://github.com/YOUR_USERNAME/Oil-Spill-Detection.git
```

### Accidentally made changes on `main`

Move them to a new branch before committing:

```bash
git checkout -b feature/my-work
```

Your changes come with you. Then commit as normal.

### "File too large" error

Don't commit dataset files (`Radar_data.rar`, `train/`). They're already
ignored by `.gitignore`, so keep them in the project folder and don't
force-add them.

---

## Team rules

1. **Never commit to `main`.** Always use a branch and a PR.
2. **Pull from upstream before starting new work.**
3. **One PR = one task.** Keep PRs small and focused.
4. **Write clear commit messages and PR descriptions.**
5. **Don't commit data, retrained `.pkl` files, or `venv/`.**
6. **Stuck?** Message the group before force-pushing or deleting anything.
