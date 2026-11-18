# Confidential Kimi K3.1 Pro production runtime

This repository is the public, signed Tinfoil release provenance for the
production Kimi K3.1 Pro confidential-inference CVM. The CVM runs on an
8x NVIDIA B300 host with Intel TDX and NVIDIA GPU confidential computing
(multi-GPU protected PCIe mode, encrypted NVLink).

## What is attested

`tinfoil-config.yml` is the exact measured runtime exported from the production
CVM specification. It pins the Kimi K3.1 Pro model reference and MPK, the DSpark
draft model and its MPK, the vLLM image digest, resource allocation, the
container environment, and the complete vLLM command line.

The vLLM image is Tinfoil's
[`confidential-kimi-k3-1-pro`](https://github.com/tinfoilsh/confidential-kimi-k3-1-pro)
v0.0.3 image, pinned by digest. Its environment and flags are Tinfoil's, with two
additions: vLLM returns `X-Request-Id`, and the model is also served as
`moonshotai/Kimi-K3.1-Pro`.

The release workflow uses Tinfoil's pinned measurement action to publish a
Sigstore-signed deployment record and expected TDX measurements. A verifier
compares those expected measurements with a fresh quote from the live CVM and
verifies the attested TLS/HPKE key binding.

## Release process

1. Update the production `cvmctl` spec.
2. Export its exact measured runtime:

   ```bash
   cvmctl export-runtime -f vm-kimi-k31-b300-prod.yml > tinfoil-config.yml
   ```

3. Review and commit the generated `tinfoil-config.yml`.
4. Run the **Tinfoil Release** workflow with a new immutable tag, for example
   `v0.0.1`.
5. Allow the new tag in the attestation collateral service before a guest boots on it.
6. Deploy that exact production specification, setting its metadata repository
   and tag to this repository and release.

Any runtime change, including the model, MPK, image digest, environment or vLLM
arguments, requires a new release before deployment.

## Security boundaries

The shim requires authentication on `/v1/chat/completions`, `/v1/completions`,
`/v1/models`, and `/metrics`. `/health` stays public. It is not proof of
attestation or of successful authenticated inference.

The shim checks keys against the backend-only control-plane namespace
(`control-plane: .../backend/`). That namespace rejects customer keys.
Customers reach this model through the attested router, which authenticates to
this CVM with a scoped backend credential.

This repository intentionally contains no credentials. The certificate
authorization token remains only in the protected host-side CVM specification.

The Tinfoil release attestation establishes that the live CVM matches this
signed runtime configuration. It does not independently establish upstream
model-weight-to-MPK provenance unless that mapping is separately provided and
verified by the model packaging process.
