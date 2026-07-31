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

**No machine exists yet.** `build` and `create --dry-run` work and contact
nothing; nothing beyond them has been run.

The launcher is pinned and self-resolving — `./walter` fetches its library on
first run and needs no checkout or install step, so the commands above work as
written.

Two keys in `colors.yml` are commented out until the first `create`, and both
are worth filling in afterwards: `oci-image-id`, so a later apply does not
propose replacing the machine because Canonical published a new image, and
`oci-instance-id`, so `stop` and `start` keep working when the R2 backend does
not.

See `CLAUDE.md` for what this shares with `../once-colors` and why that is safe.
