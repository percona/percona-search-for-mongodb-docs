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

## Verifying and scanning the SBOM

The examples below use [Grype](https://github.com/anchore/grype).

!!! note
    Trivy can scan the SBOM embedded in a tarball or container image when you scan the filesystem or image. For an OCI-attached image SBOM, scan the image itself with Trivy. A standalone SBOM scan may skip OS packages because the attached SBOM identifies the operating system as `rhel`, which Trivy does not recognize as the Red Hat distribution name.

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

The Percona Search for MongoDB image has an SBOM attached as an OCI artifact. Use Trivy to scan the image and its attached SBOM without downloading the SBOM separately:

```bash
trivy image --severity HIGH,CRITICAL --sbom-sources oci \
    docker.io/percona/percona-server-mongodb-mongot:<tag>
```

#### Advanced: Inspect OCI-attached SBOMs with ORAS

Use the [ORAS CLI](https://oras.land/) to discover and download an OCI-attached SBOM.

1. Discover the attached artifacts for the image:

    ```bash
    oras discover --format tree \
        docker.io/percona/percona-server-mongodb-mongot:<tag>
    ```

2. Copy the SBOM artifact digest from the output and replace `<sbom_artifact_digest>` with the value shown after `sha256:`:

    ```bash
    oras pull docker.io/percona/percona-server-mongodb-mongot@sha256:<sbom_artifact_digest>
    ```

3. Confirm that the SBOM file was downloaded:

    ```bash
    ls
    ```
