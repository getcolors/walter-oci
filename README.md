# walter-oci

A remote development machine on Oracle Cloud, managed with
[walter](https://github.com/getcolors/walter).

```sh
./walter build              # render .colors/walter-oci/ — contacts nothing
./walter create --dry-run   # print the graph — touches nothing
./walter create             # provision it
ssh walter-oci              # get on it
./walter stop               # power off when you are done
./walter start              # power on tomorrow
```

`colors.yml` is the only file you normally edit. Credentials never go in it —
they come from the gitignored `.envrc.private` as `COLORS_PAR_*` variables.

## Prerequisites

`direnv allow` brings up the toolchain through devenv: babashka, OpenTofu,
Ansible, the `oci` CLI, and the AWS CLI the R2 backend authenticates through.

`create` and `delete` need R2 credentials in `.envrc.private`:

```sh
export COLORS_PAR_R2_ACCESS_KEY_ID=...
export COLORS_PAR_R2_SECRET_ACCESS_KEY=...
```

OCI needs no variable — it authenticates from the `DEFAULT` profile in
`~/.oci/config`. That profile is session-token based and sessions last an hour;
when walter says it has expired, run what it tells you to.

## Status

**Created and running.** `VM.Standard.A2.Flex`, 2 OCPU / 12 GB / 100 GB, Ubuntu
24.04 on aarch64, in `XquT:EU-FRANKFURT-1-AD-1`. `ssh walter-oci` reaches it.

Both post-create keys are filled in, so `colors.yml` now plans clean:
`oci-image-id` is pinned to the image actually booted — an unpinned one is a
moving target that would eventually propose replacing the machine — and
`oci-instance-id` is set, so `stop` and `start` work even when the R2 backend
does not.

`tofu plan` reports no changes.

Nothing is installed on the machine. walter's remote stage is a connectivity
ping in v1; toolchains and dotfiles are yours to add, and a later playbook to
automate.

See `CLAUDE.md` for what this shares with `../once-colors` and why that is safe.
