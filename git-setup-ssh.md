# Git + GitHub SSH Setup Template (Reusable)

Use this guide whenever you set up Git and GitHub SSH access for a new machine or repository.

Target environment: macOS (`zsh`)  
Works for: any GitHub repository using SSH remotes

---

## 0) Personal Info Placeholders (Replace These First)

Anywhere you see these placeholders, replace them with your own values:

| Placeholder | What it means | Example |
|---|---|---|
| `<YOUR_NAME>` | Your Git commit display name | `Wei Dapan` |
| `<YOUR_EMAIL>` | Email used for Git commits / GitHub account | `you@example.com` |
| `<YOUR_GITHUB_USERNAME>` | Your GitHub handle | `weida8` |
| `<YOUR_REPO_NAME>` | Repository name on GitHub | `useful-guides` |
| `<HOME_DIR>` | Your macOS home directory path | `/Users/your-mac-username` |
| `<PATH_TO_REPO>` | Local folder path of your project | `/Users/wei-dapan/git/guides` |

---

## 1) Verify Required Tools

```bash
git --version
ssh -V
```

If `git` is missing, install Xcode Command Line Tools:

```bash
xcode-select --install
```

---

## 2) Configure Git Identity (Global)

Set your personal commit identity:

```bash
git config --global user.name "<YOUR_NAME>"
git config --global user.email "<YOUR_EMAIL>"
```

Verify:

```bash
git config --global --get user.name
git config --global --get user.email
```

---

## 3) Check for Existing SSH Keys

```bash
ls -al <HOME_DIR>/.ssh
```

If you already have both files below, you can reuse them and skip to Step 4:

- `<HOME_DIR>/.ssh/id_ed25519`
- `<HOME_DIR>/.ssh/id_ed25519.pub`

---

## 4) Generate a New SSH Key (Recommended: Ed25519)

```bash
ssh-keygen -t ed25519 -C "<YOUR_EMAIL>"
```

When prompted:

- File location: press **Enter** to use default (`<HOME_DIR>/.ssh/id_ed25519`)
- Passphrase: optional but recommended

---

## 5) Start SSH Agent and Add Your Key

Start the agent:

```bash
eval "$(ssh-agent -s)"
```

Create/update SSH config for macOS keychain support:

```bash
touch <HOME_DIR>/.ssh/config
chmod 600 <HOME_DIR>/.ssh/config
cat >> <HOME_DIR>/.ssh/config <<'EOF'
Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile <HOME_DIR>/.ssh/id_ed25519
EOF
```

Add key to agent + keychain:

```bash
ssh-add --apple-use-keychain <HOME_DIR>/.ssh/id_ed25519
```

---

## 6) Copy Public Key and Add It to GitHub

Copy key to clipboard:

```bash
pbcopy < <HOME_DIR>/.ssh/id_ed25519.pub
```

In GitHub:

1. Go to **Settings** → **SSH and GPG keys** → **New SSH key**
2. **Title**: use a device label (example: `MacBook-Pro-2026`)
3. **Key type**: `Authentication Key`
4. **Key**: paste clipboard contents
5. Click **Add SSH key**

---

## 7) Test SSH Authentication to GitHub

```bash
ssh -T git@github.com
```

Expected first-time prompt: confirm GitHub host fingerprint (`yes`).

Expected success message:

```text
Hi <YOUR_GITHUB_USERNAME>! You've successfully authenticated, but GitHub does not provide shell access.
```

---

## 8) Initialize Local Repository (If Needed)

Go to your project folder:

```bash
cd <PATH_TO_REPO>
```

If the folder is **not** already a Git repo:

```bash
git init -b main
```

Check status:

```bash
git status
```

---

## 9) Connect to GitHub Remote and Push

Create an empty repository on GitHub first (no README/license/gitignore initialization).

Add remote:

```bash
git remote add origin git@github.com:<YOUR_GITHUB_USERNAME>/<YOUR_REPO_NAME>.git
```

First commit:

```bash
git add .
git commit -m "Initial commit"
```

Push:

```bash
git push -u origin main
```

Verify remote:

```bash
git remote -v
```

---

## 10) Change or Replace Remote URL Later

Show current remotes:

```bash
git remote -v
```

Replace existing `origin` URL:

```bash
git remote set-url origin git@github.com:<YOUR_GITHUB_USERNAME>/<YOUR_REPO_NAME>.git
```

If `origin` does not exist yet:

```bash
git remote add origin git@github.com:<YOUR_GITHUB_USERNAME>/<YOUR_REPO_NAME>.git
```

Verify:

```bash
git remote -v
```

---

## 11) Troubleshooting

### A) `Permission denied (publickey)`

Run:

```bash
ssh-add -l
ssh -T git@github.com
```

If no identities are listed, add key again:

```bash
ssh-add --apple-use-keychain <HOME_DIR>/.ssh/id_ed25519
```

Also confirm the public key was added to the correct GitHub account.

### B) `remote origin already exists`

Use:

```bash
git remote set-url origin git@github.com:<YOUR_GITHUB_USERNAME>/<YOUR_REPO_NAME>.git
```

### C) `src refspec main does not match any`

You likely have no commit yet. Run:

```bash
git add .
git commit -m "Initial commit"
git push -u origin main
```

### D) Wrong Git identity on commits

Check current values:

```bash
git config --global --get user.name
git config --global --get user.email
```

Reset if needed:

```bash
git config --global user.name "<YOUR_NAME>"
git config --global user.email "<YOUR_EMAIL>"
```

---

## 12) Reuse Checklist (Quick)

- [ ] Replace all placeholders with personal values
- [ ] Confirm SSH auth: `ssh -T git@github.com`
- [ ] Confirm remote URL: `git remote -v`
- [ ] Confirm first push succeeded

This template is ready to reuse for future repositories.
