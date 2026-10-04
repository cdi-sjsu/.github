# Contributing to CDI projects

## Access and teams

Organization owners are **Nativity8904** and **nicojeda189**. New members receive
repository access through explicit team grants; organization membership alone
does not grant write access or access to private repositories.

| Team | Repositories with write access |
| --- | --- |
| [`gpu`](https://github.com/orgs/cdi-sjsu/teams/gpu) | `tiny-fixed-function-gpu` |
| [`learning`](https://github.com/orgs/cdi-sjsu/teams/learning) | `demo-design-guide`, `svlings`, `grimoire` |
| [`website`](https://github.com/orgs/cdi-sjsu/teams/website) | `.github`, `assets`, `cdi-sjsu.github.io`, `grimoire` |
| [`events`](https://github.com/orgs/cdi-sjsu/teams/events) | Private `club-events` |

The `gpu` team is nested under `projects`. The parent has no repository grants;
future projects can receive independent permissions. GPU subsystem teams may be
added beneath `gpu` later. Current membership is shown on each team's GitHub page.

## Pull requests and merging

1. Create a feature branch. If you lack write access, fork the public repository
   and create your branch there.
2. Follow the repository README's development and test instructions.
3. Open a pull request into `main` with a clear title, the resulting behavior,
   and the relevant validation.
4. Resolve review conversations and pass any required CI checks.
5. Squash and merge. The PR title and description become the commit message;
   merged feature branches are deleted automatically.

All seven public repositories require PRs and protect `main` from force pushes
and deletion. `tiny-fixed-function-gpu` and `demo-design-guide` require the GitHub
Actions `check` job and an up-to-date branch. Other public repositories do not
currently require CI checks; their deployment workflows continue to run normally.

**GPU only:** PRs require one approval, refreshed after reviewable changes. The
latest reviewable push must be approved by someone other than its pusher.
Nativity8904 has an explicit PR-only review exception, while still needing CI,
resolved conversations, and the other protections. No other user has a configured
review exception. Other public repositories require no approvals.

The private `club-events` repository cannot enforce these branch rules on the
organization's current Free plan. Use the same PR workflow there voluntarily.

External fork contributors need a maintainer to approve Actions workflow runs.
Cryptographic commit signing is recommended and optional. DCO sign-off is also
optional and is separate from cryptographic signing.

## Git signatures

Existing GPU commits include PGP signatures. `cannot run gpg: No such file or
directory` means the local verifier is missing; it does not establish a bad
signature. You can verify PGP history while using SSH to sign your own commits.

### Verify historical PGP signatures on macOS

```sh
brew install gnupg
gpg --version
key_dir=$(mktemp -d)
curl --fail --location --silent --show-error \
  https://github.com/KaitoTLex.gpg --output "$key_dir/contributor.asc"
gpg --show-keys --with-fingerprint "$key_dir/contributor.asc"
```

Before importing, compare the displayed fingerprint against the key that signed
GPU commit `ba41fd583b18fa462ca5923989b8328a951a59ca`:

```text
42F5 2D76 F1B1 5B8D 997E 2AEE 8AB9 3474 6F47 5D0B
```

Confirm key identity with the contributor when establishing trust. If the
matching historical key is no longer published, request its public key from them.
Then, from your GPU checkout:

```sh
gpg --import "$key_dir/contributor.asc"
git -c gpg.openpgp.program="$(command -v gpg)" verify-commit ba41fd583b18fa462ca5923989b8328a951a59ca
rm -r "$key_dir"
```

### Verify historical PGP signatures on Windows

Install [Git for Windows](https://gitforwindows.org/) and
[Gpg4win](https://www.gpg4win.org/), then reopen PowerShell and VS Code. Make sure
Gpg4win's executable directory is on your user `PATH`.

```powershell
$gpgProgram = (Get-Command gpg).Source
$keyFile = Join-Path $env:TEMP "gpu-contributor-public-keys.asc"
Invoke-WebRequest -Uri "https://github.com/KaitoTLex.gpg" -OutFile $keyFile
& $gpgProgram --show-keys --with-fingerprint $keyFile
# Compare the fingerprint above before importing.
& $gpgProgram --import $keyFile
git -c "gpg.openpgp.program=$gpgProgram" verify-commit ba41fd583b18fa462ca5923989b8328a951a59ca
Remove-Item $keyFile
```

Use the same GPG installation for importing and verifying keys. A **good
signature / unknown trust** result means the signature verifies but your keyring
has not established the owner's identity. Do not assign ultimate ownertrust to
another person's key merely to remove that warning. GitHub verification is
independent of your local keyring and trust settings.

WSL and containers have separate executables and keyrings. In Ubuntu, install
GnuPG with `sudo apt-get update` and `sudo apt-get install -y gnupg`. The GPU Dev
Container includes it already. Use the shell download/import commands there,
without Homebrew; public-key imports may need repeating after a container rebuild.

### Sign new commits with SSH

Use Git 2.34 or later and OpenSSH with signing support (8.2 or later; avoid 8.7's
known signing issue). Use an existing suitable key or create a dedicated key.
Do not overwrite an existing key file.

On macOS or WSL:

```sh
mkdir -p ~/.ssh
ssh-keygen -t ed25519 -C "YOUR_VERIFIED_GITHUB_EMAIL" -f ~/.ssh/id_ed25519_signing
# If needed, start an agent: eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_signing
cat ~/.ssh/id_ed25519_signing.pub
```

On Windows, enable the **OpenSSH Client** optional feature. Start its agent from
an elevated PowerShell terminal:

```powershell
Set-Service ssh-agent -StartupType Automatic
Start-Service ssh-agent
```

Then use normal PowerShell:

```powershell
$sshTools = Join-Path $env:WINDIR "System32\OpenSSH"
$signingKey = Join-Path $env:USERPROFILE ".ssh\id_ed25519_signing"
New-Item -ItemType Directory -Force (Split-Path $signingKey) | Out-Null
& "$sshTools\ssh-keygen.exe" -t ed25519 -C "YOUR_VERIFIED_GITHUB_EMAIL" -f $signingKey
& "$sshTools\ssh-add.exe" $signingKey
git config --local gpg.ssh.program "$sshTools\ssh-keygen.exe"
Get-Content "$signingKey.pub"
```

On either platform, configure your checkout using the public key's first two
fields. A literal public key avoids host file paths that do not exist in containers:

```sh
git config --local user.email "YOUR_VERIFIED_GITHUB_EMAIL"
git config --local gpg.format ssh
git config --local user.signingkey "key::ssh-ed25519 YOUR_PUBLIC_KEY_DATA"
git config --local commit.gpgsign true
```

Register the public key in **GitHub Settings > SSH and GPG keys > New SSH key**,
selecting **Signing Key**. SSH authentication and signing registrations are
distinct, even when the same key is used. Keep your private key on your machine.

For local SSH verification, create an allowed-signers file outside the repository
containing identities and public keys you intend to trust:

```text
YOUR_VERIFIED_GITHUB_EMAIL namespaces="git" ssh-ed25519 YOUR_PUBLIC_KEY_DATA
```

```sh
git config --local gpg.ssh.allowedSignersFile "/absolute/path/to/allowed_signers"
git verify-commit HEAD
```

PowerShell uses the same Git commands with a Windows path. This file controls
SSH verification and does not import PGP public keys.

### Signing in the Dev Container

Load your key into the host agent before opening the container. VS Code forwards
a reachable SSH agent and copies host Git configuration. Inside the container:

```sh
printf '%s\n' "$SSH_AUTH_SOCK"
ssh-add -l
git config --show-origin --get-regexp '^(user\.|gpg\.|commit\.gpgsign)'
```

Use the literal `key::` public-key setting above. If Windows configured an
absolute `gpg.ssh.program` path, override it for a Linux-container commit:

```sh
git -c gpg.ssh.program=ssh-keygen commit -S
```

For PGP verification inside Linux, use `git -c gpg.openpgp.program=gpg verify-commit
<commit>`. For SSH verification, create a public-only allowed-signers file inside
the container and override its path with `git -c gpg.ssh.allowedSignersFile=/path/in/container/allowed_signers
verify-commit HEAD`.

If the agent is unavailable or has no keys, check it on the host, restart VS Code
with the agent environment, and reopen the container. A WSL-launched VS Code
session needs an agent reachable from WSL; use a WSL agent for that workflow.
Do not copy private keys into images or disable signing to fix forwarding.

### References

- [Git signing and verifier configuration](https://git-scm.com/docs/git-config)
- [GitHub signing setup](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key)
- [GitHub signature verification](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification)
- [VS Code container credentials and agents](https://code.visualstudio.com/remote/advancedcontainers/sharing-git-credentials)
