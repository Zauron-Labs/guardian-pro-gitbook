# Verifying images

Guardian Pro is delivered as container images from Zauron's container registry, `zauron.azurecr.io`. This page explains how release images are protected and how you can check that an image is the one Zauron released.

## How releases are protected

- **Signed by digest.** Every Guardian Pro release image (the Guardian application, the viewer, the web entry point, DataForge and the report-search model) is signed with [cosign](https://github.com/sigstore/cosign). The signature covers the image digest, its content hash, and carries the release version it was signed for. Releases after 6.31.0 are signed; 6.31.0 and earlier releases are not.
- **The signing key never leaves Zauron's key vault.** Signing happens inside Microsoft Azure Key Vault; only the public key is shared.
- **Release tags are locked.** After signing, each release tag (for example `6.32.0`) is locked in the registry: it cannot be overwritten or deleted, so a version number always names the same image. `latest` is a moving tag and is never locked; the updater refuses it on production installations.
- **Pinned by digest on your host.** Upgrades pin each image by version **and** digest, so the image that runs is exactly the one that was pulled and checked.
- **Release notes list every digest.** Zauron sends the image names and digests with each release.

## Get the public key

The public key (`cosign.pub`) is available from Zauron on request, and will also be published on this page. Before you rely on it, compare its fingerprint with the one Zauron gives you through a second channel (for example your Zauron contact, or the security exhibit of your agreement):

```bash
openssl pkey -pubin -in cosign.pub -outform DER | sha256sum
```

## Verify an image

Verify the **digest**, not the tag: a digest is the content, a tag is only a name for it.

```bash
docker login zauron.azurecr.io     # with the registry credentials Zauron issued for your installation

cosign verify --key cosign.pub --insecure-ignore-tlog=true -a tag=6.32.0 \
  zauron.azurecr.io/<image>@sha256:<digest from the release notes>
```

- Exit code 0 and `The signatures were verified against the specified public key` means the image is the one Zauron released. Any other result: do not run the image, and tell Zauron.
- `-a tag=6.32.0` checks the release version signed into the signature, so an older signed image cannot be passed off as a newer release.
- `--insecure-ignore-tlog=true` skips the public Sigstore transparency log. Zauron does not upload signatures to it, because the registry is private and many sites verify without internet access. The check against the public key is unchanged.
- To find the digest a tag points to: `docker buildx imagetools inspect zauron.azurecr.io/<image>:6.32.0` (or `crane digest`). Compare it with the release notes.

Signatures are stored in the same registry repository as the image, under the tag `sha256-<digest>.sig` (`cosign triangulate <image>@<digest>` prints it). No special registry feature is needed, so verification works on any OCI registry.

## Verify every upgrade on your Guardian host

The host updater can verify the signature of every image it pulls, before anything is pinned or restarted. When this is on, the check **fails closed**: a missing signature, a signature from another key, a missing cosign program or an image without a registry digest fails the upgrade and leaves the running version unchanged.

To turn it on, cosign v2 is installed on the Guardian host, the public key is placed in a root-owned file, and the check is switched on in the updater's host configuration, which the Guardian web consoles cannot change. Zauron does this with you on request. Upgrade to a signed release before you turn the check on.

## Mirroring into your own registry

Copy the image **and** its signature, by digest. Verification then works against your mirror with the same key.

With cosign (copies the image with its signatures):

```bash
cosign copy zauron.azurecr.io/<image>@sha256:<digest> \
            registry.example.org/guardian/<image>:6.32.0
cosign verify --key cosign.pub --insecure-ignore-tlog=true -a tag=6.32.0 \
  registry.example.org/guardian/<image>@sha256:<digest>
```

With [ORAS](https://oras.land) (the image, then its signature tag):

```bash
oras cp zauron.azurecr.io/<image>@sha256:<digest> \
        registry.example.org/guardian/<image>:6.32.0
oras cp zauron.azurecr.io/<image>:sha256-<digest>.sig \
        registry.example.org/guardian/<image>:sha256-<digest>.sig
```

Copy by digest so the digest does not change: rebuilding or recompressing an image changes its digest, and the signature would no longer match. Then ask Zauron to point your installation at the mirror; the digest pins stay the same.

## Kubernetes and admission control

Policy engines that support cosign key signatures (Sigstore policy-controller, Kyverno `verifyImages`, Ratify) can enforce the same check with `cosign.pub`. Configure key-based verification with the transparency log check off (Kyverno: `rekor: { ignoreTlog: true }`).

## Questions

To request the public key or a release's digests, contact [service@zauronlabs.com](mailto:service@zauronlabs.com).
