# Known limitations

Percona Search for MongoDB packages `mongot`, the same search engine that powers self-managed MongoDB Search and Vector Search. Because this technical preview builds directly on upstream `mongot`, it inherits the same underlying limitations and unsupported behavior as upstream self-managed deployments.

This page summarizes the limitations most relevant to Percona Search for MongoDB. For the complete list, see [Known Limitations for Self-managed mongot :octicons-link-external-16:](https://www.mongodb.com/docs/search/self-managed/current/limitations/){:target="_blank"} in the MongoDB documentation.

## Platform and deployment

- `mongot` runs on Linux only, on `x86_64` and `aarch64`. There are no native Windows or macOS builds.
- `mongot` isn't supported on IBM Power (`ppc64le`) or IBM Z (`s390x`).

<!-- TBD-ENG: confirm whether Percona's own tarball, RPM, DEB, or container packages of mongot differ from upstream Community Edition packaging limitations (no systemd unit in the tarball, no apt/yum installation). State Percona's actual packaging support here rather than assuming parity with upstream. -->

## Authentication and security

- `mongot` reads X.509 certificates and SCRAM credentials only at startup. Rotating either requires a `mongot` restart to take effect. Schedule rotations during a maintenance window.
- `mongot` doesn't validate certificate revocation through OCSP or CRLs. To revoke access immediately, remove the corresponding `$external` user on `mongod`; the next handshake rejects the revoked certificate.
- `mongot` doesn't provide native application-level encryption at rest. Encrypt the underlying filesystem or block device instead, for example with LUKS, `dm-crypt`, or a cloud provider's encrypted volumes.
- `mongot` doesn't support running its TLS stack in a FIPS-validated cryptographic mode.

## Observability

- `mongot` doesn't include a built-in metrics UI. Use Prometheus, Grafana, or another monitoring stack against the `mongot` `/metrics` endpoint.
- You're responsible for defining your own alerting, and for managing FTDC and log retention.

## Operations and upgrades

- A single `mongot` instance is bound to one replica set, or to one shard in a sharded cluster. It can't serve multiple unrelated MongoDB deployments. Deploy a separate `mongot` instance per replica set or shard.
- Downgrading `mongot` across major versions requires a re-sync between `mongod` and `mongot`, since major versions aren't rollback-safe.
- Upgrading `mongot`, or changing its configuration, both require a process restart. `mongot` reads its configuration only at startup.