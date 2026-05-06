# Why You Cannot Make This Fork Private — and What To Do About It

## The Problem

When you try to change the visibility of a public fork (e.g. `your-username/your-fork-name`) from public to private, GitHub shows an error similar to:

> *"You cannot make your fork of a public repository private for security reasons."*

This is a **GitHub platform policy**, not a bug, and it applies to any public fork of any public repository.

---

## Why GitHub Enforces This Restriction

GitHub's restriction exists to prevent a subtle but real security problem:

1. **Public forks are already public.** Once a repository is forked publicly, its content (including all commit history) is indexed, cached, and accessible to anyone. Making the fork "private" after the fact does **not** remove any data from the internet — it only changes who can see it going forward on GitHub.

2. **Network visibility.** GitHub maintains a "fork network" that links all forks of an original repository. A private fork within a public fork network could still expose metadata (existence of the repo, contributor activity, etc.) in ways that create a false sense of security.

3. **Preventing security theater.** If you accidentally commit a secret (API key, password) to a public fork, hiding it behind a private repo gives false assurance — the credential is already compromised. GitHub therefore blocks the action and encourages you to rotate the credential instead.

4. **Upstream repository is public.** As long as the upstream (`n8n-io/n8n`) is public, GitHub considers the entire fork network public and does not allow individual forks in that network to change their visibility to private.

**Reference:** [GitHub Docs — Changing a fork's visibility](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/setting-repository-visibility#changing-a-forks-visibility)

---

## Your Options / Workarounds

### Option A — Detach the Fork and Convert to a Standalone Private Repository (Recommended)

GitHub Support can **detach the fork relationship** between your repo and the upstream. Once detached, your repository is no longer part of the fork network and you can set its visibility to private.

**Steps:**

1. Go to [GitHub Support](https://support.github.com/contact) and open a request.
2. Select the topic **"Repository" → "I want to change this repository's settings"**.
3. Ask them to *"detach the fork relationship for `your-username/your-fork-name` from `n8n-io/n8n`"*.
4. Once detached, navigate to **Settings → Danger Zone → Change repository visibility** and set it to **Private**.

> **Note:** After detaching, you will lose the ability to easily submit pull requests to the upstream. You can still add the upstream as a git remote manually if you need to pull updates.

---

### Option B — Create a New Private Repository (Mirror)

Create a fresh private repository and push the current code there. This gives you a clean private copy with no fork relationship.

```bash
# 1. Create a new private repository on GitHub (via UI or gh CLI)
gh repo create your-username/your-private-repo --private

# 2. Clone your current fork locally (if not already)
git clone https://github.com/your-username/your-fork-name.git
cd your-fork-name

# 3. Add the new private repo as a remote and push everything
git remote add private https://github.com/your-username/your-private-repo.git
git push private --mirror

# 4. (Optional) Update your local default remote
git remote set-url origin https://github.com/your-username/your-private-repo.git
```

You can keep the original public fork as-is or delete it once the private mirror is set up.

---

### Option C — Use GitHub's "Import Repository" Feature

GitHub's importer creates a completely independent copy of a repository — no fork relationship.

1. Go to **https://github.com/new/import**.
2. In **"Your old repository's clone URL"**, enter `https://github.com/your-username/your-fork-name.git`.
3. Choose your account/org as the owner and give it a new name (e.g., `maia-n8n-private`).
4. Set visibility to **Private**.
5. Click **Begin Import**.

This imports the full commit history without linking back to the upstream fork network.

---

### Option D — Keep It Public (No Action Required)

If there are no secrets or sensitive customizations in the repository, the simplest path is to leave it public. The code it is forked from (`n8n-io/n8n`) is already public under its own license, so making the fork private does not provide meaningful additional protection for the code itself.

---

## Important: If You Committed Secrets

If the reason you want to make the repo private is that a secret (API key, password, token) was accidentally committed:

> **Making the repo private does NOT revoke or secure the credential.**  
> The secret has already been public and may have been scraped.

The correct response is:
1. **Immediately rotate/revoke the credential** in the relevant service.
2. Remove the secret from git history using [`git filter-repo`](https://github.com/newren/git-filter-repo) or [BFG Repo Cleaner](https://rtyley.github.io/bfg-repo-cleaner/).
3. Force-push the cleaned history.
4. Then change visibility (using one of the options above if needed).

---

## Summary Table

| Option | Fork Relationship | Private? | Preserves History | Effort |
|--------|------------------|----------|-------------------|--------|
| A — Detach via Support | Removed | ✅ Yes | ✅ Yes | Low (wait for support) |
| B — Mirror to new repo | None | ✅ Yes | ✅ Yes | Low |
| C — Import via GitHub | None | ✅ Yes | ✅ Yes | Low |
| D — Keep public fork | Maintained | ❌ No | ✅ Yes | None |

---

## License Reminder

Regardless of repository visibility, remember that this repository contains code licensed under the [Sustainable Use License](../LICENSE.md) and the [n8n Enterprise License](../LICENSE_EE.md). Making the repository private does not change your obligations under those licenses. In particular:

- Files with `.ee.` in their filename or `.ee` in their directory path require a valid **n8n Enterprise License** for production use.
- You may **not** remove or obscure license/copyright notices.
- Personal/non-commercial use is permitted under the Sustainable Use License.

See [LICENSE.md](../LICENSE.md) and [LICENSE_EE.md](../LICENSE_EE.md) for full terms.
