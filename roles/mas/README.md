# mas

Sets up [matrix-authentication-service](https://github.com/element-hq/matrix-authentication-service)
(MAS) as a podman container managed by systemd through a quadlet unit
(`/etc/containers/systemd/matrix-mas.container`, started as `matrix-mas.service`
and part of `matrix.target`).

The role does nothing unless `mas_enabled` is true.

## Requirements

- Debian or Ubuntu with podman >= 4.4 (quadlet). The role installs `podman` and
  `python3-cryptography`.
- A postgres database for MAS, via `psql_dbs['mas']` or `mas_psql_*`.
- `community.crypto` and `containers.podman` collections.

## Required variables

- `mas_domain`: public hostname for MAS, e.g. `auth.example.org`. nginx proxies it to `mas_port`.
- `mas_shared_secret`: shared with synapse. The synapse role writes the same value into `sso.yaml`.

`mas_encryption_secret` is optional. When empty the role keeps the one already in
`/etc/matrix-mas/config.yaml`, or generates one on a fresh install. Signing keys are
kept the same way, or one RSA key is generated into `/etc/matrix-mas/private/`.

See `meta/argument_specs.yml` for everything else.

## Networking

The container runs with host networking by default so MAS binds on
`127.0.0.1:8007`, synapse reaches it on `localhost:8007` and MAS reaches synapse
and postgres on `localhost`, just like the other services in this collection.

## Migrating an existing non-container install

When the role finds a MAS it did not deploy (a `config.yaml` without the role's
header line, or a unit named in `mas_legacy_service_names` such as
`matrix-mas.service` in `/etc/systemd/system`), it migrates it in place:

1. Backs up the old `config.yaml`, unit files and drop-ins to `/var/backups/matrix-mas-legacy`.
2. If `/etc/matrix-mas/private` is a symlink (e.g. to `/var/lib/matrix-mas/private`),
   replaces it with a real directory holding copies, because the container only
   mounts `/etc/matrix-mas`. The original directory is left untouched.
3. Keeps the encryption secret (also from `encryption_file`), all signing keys
   (inline or `key_file`, with paths rewritten to the new directory) and the issuer.
4. Keeps every config section the role does not manage (`clients`, `upstream_oauth2`,
   `passwords`, `email`, `account`, `branding`, `policy.data`, ...) in
   `/etc/matrix-mas/legacy.yaml`. They are merged under the role's config on every run.
   Variables like `mas_passwords` or `mas_upstream_oauth2_providers` override them.
5. Drops `http`, `matrix`, `templates` and `policy.wasm_module`, which pointed at the
   binary's share directory. The image ships its own.
6. Keeps the old `database` section unless `mas_psql_user` / `psql_dbs['mas']` is set.
7. Stops and disables the old units, removes their unit files and drop-ins (a
   `/etc/systemd/system/matrix-mas.service` would shadow the quadlet unit), then
   starts the container and waits for `/health`.

Safety checks, on migration and on every later run:

- The role refuses to change the issuer (`mas_issuer_allow_change`).
- The role refuses to change the encryption secret (`mas_encryption_secret_allow_change`).
- The role fails if the kept config references files outside `/etc/matrix-mas`
  that are not mounted with `mas_container_extra_volumes`.
- Every config is validated with `mas-cli config check` in the target image before
  it is written.

Before migrating, set `mas_version` to the running version or newer. MAS runs
database migrations on start and cannot downgrade. Check the old version with
`mas-cli --version`. The old binary and `/var/lib/matrix-mas` are not removed.

Note that `mas_bind_address` and `mas_port` replace the old listeners, so nginx must
proxy to `mas_port` (the collection's nginx vars already do).

## Migrating users from synapse

Moving existing synapse users into MAS (`mas-cli syn2mas`) is a separate one-time
step this role does not run. See the
[MAS migration guide](https://element-hq.github.io/matrix-authentication-service/setup/migration.html).
