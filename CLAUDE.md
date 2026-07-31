# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## What this repository is

Desired state for `walter-oci`: one Oracle Cloud development machine, OpenTofu
state in Cloudflare R2. Nothing here is source code. What is tracked is
`colors.yml`, the installed launcher, the skill package behind it, and the
dev-environment files.

```text
colors.yml                        the desired state — the only file you normally edit
walter                            the installed launcher (a COPY of the payload)
.agents/skills/package-walter-green   the installed skill package
.claude/skills/package-walter-green   a symlink into .agents/skills, so Claude Code finds it
.envrc                            secret-free; sources the gitignored .envrc.private
devenv.nix                        the toolchain
```

Everything else is generated (`.colors/`) or secret (`.envrc.private`).
`.gitignore` is `.*` with narrow negations, so check `git ls-files` rather than
assuming from the working tree.

## Commands

```sh
./walter build              # render .colors/walter-oci/ — contacts nothing
./walter create --dry-run   # print the graph — touches nothing
./walter create             # provision, and write the ssh alias
./walter stop               # power off
./walter start              # power on, and refresh the alias
./walter delete             # destroy (guarded — see below)
```

After a successful `create`, `ssh walter-oci` reaches the machine: the local
Ansible stage writes a managed block into `~/.ssh/config` with agent forwarding
on, so you can push to git from the machine without copying a private key onto
it.

## The relationship with once-colors

This project deliberately shares four things with `../once-colors`: the OCI
tenancy, the compartment, the **subnet** and availability domain, the `DEFAULT`
session profile, and the R2 bucket.

Two consequences, stated rather than discovered:

- The machine sits in the production website's subnet and inherits its security
  list. It can also reach the website server over private IPs, which is useful
  for debugging against the real stack.
- One `oci session refresh` serves both projects.

What keeps them apart is **`profile`**, and only that. Remote state is keyed
`<profile>/<stage>.tfstate`, so this writes `walter-oci/walter-compute.tfstate`
while once-colors writes `once-colors/tofu-compute.tfstate`. Both halves differ —
walter names its stage `walter-compute` rather than `tofu-compute` for exactly
this reason — so sharing one bucket is safe.

**Never export `COLORS_PAR_PROFILE`.** `profile` is a flat key and `COLORS_PAR_*`
overlays any flat key, so one variable in the wrong shell would point walter at
once-colors' state — same bucket, same compartment, same subnet. Walter refuses
to start when it is set. Do not suggest a workaround; that is the guard working.

## OCI credentials

The `DEFAULT` profile is session-token based, so `.envrc` exports
`OCI_CLI_AUTH=security_token` — the `oci` CLI rejects that profile otherwise.
OpenTofu's `oracle/oci` provider detects `security_token_file` by itself and does
not need the variable.

That asymmetry matters here more than it does in once-colors, because **walter's
everyday verbs drive the CLI**. `stop` and `start` never reach OpenTofu — power
state is not desired state — so they depend on a live session where `create` does
not. Sessions last 60 minutes.

When walter reports an expired session it names the fix:

```sh
bb ~/.claude/skills/refresh-oci-token/refresh-oci-token.clj
```

That skill is installed globally on this machine, so it works from any
directory. Within a session's window it extends the token in place with no
browser; only a fully expired session needs the login flow, which needs an SSH
port forward on 8181.

This is a deliberate choice of temporary credentials over a long-lived API key,
accepting the refresh friction in exchange.

## Gotchas

**The root `walter` is a copy, not a symlink.** `npx skills update -p` rewrites
`.agents/skills/` and leaves the root file untouched, so a project that skips the
re-copy keeps running the old pin while the lockfile claims the new one:

```sh
npx skills update -p
cp .agents/skills/package-walter-green/walter walter
```

**`stop` does not restart with `create`.** With no power state in the
configuration there is no diff, so an apply leaves a stopped machine stopped.
`start` is the only way up.

**Pin `oci-image-id` after the first create.** Left unset the newest compatible
Canonical image is used, and the image id forces replacement — so a later apply
proposes destroying the machine because Canonical published something new. With
`compute-prevent-destroy: true` that apply fails instead, which is safe and
confusing. Read the id back with `tofu state show oci_core_instance.ampere_vm`.

**Fill in `oci-instance-id` once it exists.** It is what makes `stop` and `start`
work when the R2 backend is unreachable, rather than leaving you with a running
machine you cannot power off.

**`delete` takes the boot volume with it.** This is a development machine; what
is on it is uncommitted work. The guard is on by default and lifted with
`COLORS_PAR_COMPUTE_PREVENT_DESTROY=false` for one intentional run.

## Provenance

The launcher here is pinned and self-resolving: `./walter` fetches
`io.github.getcolors/walter` at the stamped commit on first run, into
`~/.gitlibs`, and needs no checkout, no `WALTER_LIB_ROOT` and no install step.

It was **copied by hand** from `../walter/skills/package-walter-green/`, not
installed with `npx skills add getcolors/walter`, so there is deliberately no
`skills-lock.json` — a lockfile records the source and content hash that an
actual install computed, and writing one by hand would be a claim this project
did not earn. Run the real install when you want it:

```sh
npx skills add getcolors/walter
cp .agents/skills/package-walter-green/walter walter    # the copy, again
```

The lockfile appears with it, and from then on `npx skills update -p` is the
way this project moves forward.

## Git

Do not commit or push unless explicitly asked.
