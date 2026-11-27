# Software Bill of Materials

A Software Bill of Materials (SBOM) is a machine-readable inventory of the components and dependencies included in a software release. It helps you understand what is included in a build and assess potential security or compliance risks.

Starting with Percona Search for MongoDB 1.79.1-4, each release includes a CycloneDX 1.6 SBOM in JSON format. The tarball and package SBOMs describe the mongot bundle and its dependencies. The SBOM attached to a Docker image also covers the image's operating system packages.

## Why it matters

An SBOM helps you:

- Identify the components and dependencies included in a Percona Search for MongoDB release.
- Assess known vulnerabilities with SBOM-compatible security scanners.
- Support security reviews, compliance processes, and software supply chain requirements.
- Verify the contents of deployed software artifacts.

## Where to find the SBOM

| Distribution method | SBOM location |
|---|---|
| Binary tarball | `percona-search-mongodb/percona-search-mongodb-<version>.cdx.json` |
| RPM package | `/usr/share/doc/percona-search-mongodb/percona-search-mongodb-<version>.cdx.json` |
| DEB package | `/usr/share/doc/percona-search-mongodb/percona-search-mongodb-<version>.cdx.json` |
| Docker image | Attached as an OCI artifact. See [Docker images](#docker-images). |

## Locating and scanning the SBOM

The examples below use [Grype](https://github.com/anchore/grype).

!!! note
    The bundle SBOM does not include operating system packages. To include image operating system packages in Trivy results, scan the container image itself with `trivy image`.

### Binary tarball

Check for the SBOM in the tarball, extract it, and scan it with Grype:

```bash
# Confirm the SBOM is bundled
tar tzf percona-search-mongodb-<version>-linux_x86_64.tar.gz \
    | grep '\.cdx\.json$'

# Extract and scan
tar xzf percona-search-mongodb-<version>-linux_x86_64.tar.gz \
    -C /tmp percona-search-mongodb/percona-search-mongodb-<version>.cdx.json
grype sbom:/tmp/percona-search-mongodb/percona-search-mongodb-<version>.cdx.json
```

### RPM package

```bash
# Confirm the package installs the SBOM
rpm -ql percona-search-mongodb | grep '\.cdx\.json$'

# Scan it
grype sbom:/usr/share/doc/percona-search-mongodb/percona-search-mongodb-<version>.cdx.json
```

### DEB package

```bash
# Confirm the package installs the SBOM
dpkg -L percona-search-mongodb | grep '\.cdx\.json$'

# Scan it
grype sbom:/usr/share/doc/percona-search-mongodb/percona-search-mongodb-<version>.cdx.json
```

### Docker images

To include operating system packages in vulnerability results, scan the image directly with Trivy:

```bash
trivy image --severity HIGH,CRITICAL \
    docker.io/percona/percona-search-mongodb:<tag>
```

#### Advanced: Inspect OCI-attached SBOMs with ORAS

The image includes a separate SBOM for each architecture. Choose `linux/amd64` or `linux/arm64` to match your image.

Use the [ORAS CLI](https://oras.land/) to inspect the attached SBOM.

1. Discover the attached artifacts for the image:

    ```bash
    oras discover --platform linux/amd64 --format tree \
        docker.io/percona/percona-search-mongodb:<tag>
    ```

2. Select the CycloneDX SBOM referrer in the output. Copy its hexadecimal digest after `sha256:`. Replace `<sbom_artifact_digest>` with that value:

    ```bash
    oras pull docker.io/percona/percona-search-mongodb@sha256:<sbom_artifact_digest>
    ```

3. List the downloaded CycloneDX SBOM file:

    ```bash
    ls -1 percona-search-mongodb-*.cdx.json
    ```
