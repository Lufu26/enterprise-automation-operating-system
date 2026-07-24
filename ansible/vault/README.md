# Ansible Vault Setup (EAOS)

## Why this exists
Ansible needs real secrets (DB passwords, JWT signing keys, API keys) to deploy the platform.
Those secrets must never exist in plaintext inside this git repo, on GitHub, or anywhere version-controlled.
Ansible Vault encrypts them at rest so the *encrypted* file is safe to commit — only someone with
the vault password can read it.

## One-time setup (per machine)
1. Create a vault password and store it OUTSIDE this repo:
   ```
   echo "your-strong-password-here" > ~/.eaos_vault_pass
   chmod 600 ~/.eaos_vault_pass
   ```
   This file must never be committed. It lives only on your machine (or a password manager).

2. Tell Ansible where to find it, so you don't type it every run:
   ```
   export ANSIBLE_VAULT_PASSWORD_FILE=~/.eaos_vault_pass
   ```
   (Add this line to your `~/.bashrc` / `~/.zshrc` so it persists.)

## Creating the encrypted secrets file
From the repo root:
```
ansible-vault create ansible/vault/secrets.yml
```
This opens an editor. Add secrets as plain YAML — Ansible encrypts the whole file on save:
```yaml
db_password: "real-password-here"
jwt_secret: "real-jwt-signing-key"
```

## Editing it later
```
ansible-vault edit ansible/vault/secrets.yml
```

## Verifying it's actually encrypted (do this once, trust nothing)
```
cat ansible/vault/secrets.yml
```
You should see `$ANSIBLE_VAULT;1.1;AES256` and unreadable ciphertext — not your password in plaintext.
If you ever see plaintext here, STOP and do not commit.

## Using secrets in a playbook
```yaml
- name: Configure database
  vars_files:
    - vault/secrets.yml
  tasks:
    - name: Set DB password
      debug:
        msg: "Using password: {{ db_password }}"
```

## Rule
`ansible/vault/secrets.yml` is the ONLY file in `ansible/vault/` that ever gets created with real
values — and it is gitignored anyway (see repo root `.gitignore`) as a second layer of protection.
This README is the only file in this directory meant to be committed.
