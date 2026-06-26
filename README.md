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

## Update Local Master Password (GPG Passphrase)

`pass` does not have a separate app-level master password.
Your local protection comes from your GPG private key passphrase.

Use these steps to change it locally:

1. Go to your home directory and confirm which key is used by your local store:

```bash
cd ~
pwd
ls .password-store
cat .password-store/.gpg-id
```

Expected key ID:

```text
9CD76D267E408A2B
```

2. Create backups before making any change:

```bash
cd ~
ts=$(date +%Y%m%d-%H%M%S)

# Backup password store
tar -czf "/tmp/password-store-backup-$ts.tar.gz" .password-store

# Backup secret key
gpg --export-secret-keys --armor 9CD76D267E408A2B > "/tmp/gpg-secret-9CD76D267E408A2B-$ts.asc"

# Verify backup files
ls -lh "/tmp/password-store-backup-$ts.tar.gz" "/tmp/gpg-secret-9CD76D267E408A2B-$ts.asc"
```

3. List your secret keys and verify that key ID exists:

```bash
gpg --list-secret-keys --keyid-format LONG
```

4. Change the passphrase for that key:

```bash
gpg --edit-key 9CD76D267E408A2B
```

Inside the GPG prompt, run:

```text
passwd
save
```

5. Clear cached passphrase from gpg-agent (so you can test the new one):

```bash
gpgconf --kill gpg-agent
```

6. Test decryption with `pass` and enter the new passphrase when prompted:

```bash
pass ls
pass show <AN_EXISTING_ENTRY_FROM_LIST>
```

7. (Optional) Re-encrypt entries only if you also changed recipients/keys.
If you changed only the passphrase, no repository content changes are required.

## Restore If Something Goes Wrong

If you cannot decrypt entries after the change, restore from the backup files created earlier.

1. Go to your home directory and confirm backup files are present:

```bash
cd ~
ls -lh /tmp/password-store-backup-*.tar.gz /tmp/gpg-secret-9CD76D267E408A2B-*.asc
```

2. Restore `.password-store` from your chosen backup archive:

```bash
cd ~
mv .password-store ".password-store.broken.$(date +%Y%m%d-%H%M%S)"
tar -xzf /tmp/<YOUR_PASSWORD_STORE_BACKUP_FILE>.tar.gz
```

3. Re-import your secret key backup (if needed):

```bash
gpg --import /tmp/<YOUR_SECRET_KEY_BACKUP_FILE>.asc
```

4. Restart gpg-agent:

```bash
gpgconf --kill gpg-agent
```

5. Verify access:

```bash
pass ls
pass show <AN_EXISTING_ENTRY_FROM_LIST>
```

6. If restore still fails, check that the restored key ID matches `.password-store/.gpg-id`:

```bash
cat ~/.password-store/.gpg-id
gpg --list-secret-keys --keyid-format LONG
```
