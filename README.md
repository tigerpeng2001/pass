# Pass Credential Store

Encrypted credential store managed with [`pass`](https://www.passwordstore.org/) and synced via a private GitHub repository.

## Structure

- `personal/` — personal credentials
- `family/` — shared family credentials
- `work/individual/` — your work-only credentials
- `work/team/` — team-shared credentials (use subfolders per team)

Example team paths:

- `work/team/engineering/`
- `work/team/ops/`

## Initialize `pass` for this store

From this folder:

```bash
pass init <YOUR_GPG_KEY_ID>
```

For folder-level sharing with specific recipients:

```bash
pass init -p family <YOUR_KEY> <FAMILY_MEMBER_KEY>
pass init -p work/team/engineering <YOUR_KEY> <TEAMMATE_KEY_1> <TEAMMATE_KEY_2>
```

## GitHub Sync

1. Create a **private** GitHub repository.
2. Add remote and push:

```bash
git remote add origin git@github.com:<you>/<repo>.git
git push -u origin main
```

## Usage examples

```bash
pass insert personal/example.com/alice
pass insert family/netflix/account
pass insert work/individual/github.com
pass insert work/team/engineering/vault
```

## Notes

- Never store unencrypted secrets outside this repository.
- Access control is managed by GPG recipients per folder/subtree.
- To revoke access, remove recipient(s) from subtree and re-encrypt with `pass init -p <path> ...`.
