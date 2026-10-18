# mas

Sets up [matrix-authentication-service](https://github.com/element-hq/matrix-authentication-service)
(MAS) as a podman container managed by systemd through a quadlet unit
(`/etc/containers/systemd/matrix-mas.container`, started as `matrix-mas.service`
and part of `matrix.target`).

The role does nothing unless `mas_enabled` is true.

## Requirements

- podman >= 4.4 on the target (quadlet support). The role installs the `podman` package.
- A postgres database for MAS (`mas_psql_*`).
- `community.crypto` collection, used to generate the signing key.

## Required variables

- `mas_domain`: public hostname for MAS, e.g. `auth.example.org`. nginx proxies it to `mas_port`.
- `mas_shared_secret`: shared with synapse, the role's synapse `sso.yaml` uses the same variable.
- `mas_encryption_secret`: 32 bytes hex, e.g. `openssl rand -hex 32`. Must never change once set.

See `meta/argument_specs.yml` for everything else. Signing keys are generated into
`/etc/matrix-mas/private/` on first run.

## Networking

The container runs with host networking by default so MAS binds on
`127.0.0.1:8007`, synapse reaches it on `localhost:8007` and MAS reaches synapse
and postgres on `localhost`, just like the other services in this collection.

## Migrating from synapse auth

Existing users must be synced into MAS with `mas-cli syn2mas`; this role does
not do that. See the
[MAS migration guide](https://element-hq.github.io/matrix-authentication-service/setup/migration.html).
