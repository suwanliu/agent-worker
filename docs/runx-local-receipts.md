# From a local skill to a verifiable runx receipt

An agent-tooling integration needs an inspectable execution record. This example
uses [runx](https://github.com/runxhq/runx) to execute a deterministic Node.js script,
check its declared output, and seal a receipt tree. It is small enough to inspect
before running: [`run.mjs`](../examples/runx-hello-world/run.mjs) reads one supplied
message and writes a JSON object; [`run.yaml`](../examples/runx-hello-world/run.yaml)
declares a required string input, a process command, and a string output.

The original walkthrough and recorded execution here were made on 2026-10-22,
using runx CLI **0.10.2**, Node **v22.23.3**, and Linux arm64. The three example
source files are copied unchanged from the upstream `hello-world` example at
[`bb0f6856d55aedaa1509bd7c5c8cb1e566cbee20`](https://github.com/runxhq/runx/tree/bb0f6856d55aedaa1509bd7c5c8cb1e566cbee20/examples/hello-world),
under [Apache-2.0](../examples/runx-hello-world/LICENSE). The public support here
is the tested walkthrough and reusable example, not authorship of upstream code.

## Run your own example

Prerequisites: Git, Node.js 20 or newer, and runx CLI 0.10.2. One installation
option is `npm install --global @runxhq/cli@0.10.2`. For the recorded execution,
I downloaded the published Linux-arm64 0.10.2 npm archive and used its native
binary in an isolated directory, with a local Node 22 runtime on `PATH`.

After cloning this repository, run these commands from its root. They reproduce
the successful run's inputs; use fresh local directories and a fresh run id if
you have already run it. The temporary-directory setup below is a portable way
to replace my machine-specific artifact paths.

```sh
runx --version
node --version
export RUNX_HOME="$(mktemp -d)"
export RUNX_RECEIPT_DIR="$(mktemp -d)"
runx skill ./examples/runx-hello-world \
  --run-id walkthrough-verified-20261022 \
  -i message='Hello from an agent-worker example.' --json
```

Selected fields from the [actual result](../examples/runx-hello-world/recorded/result.json):

```json
{
  "status": "sealed",
  "run_id": "walkthrough-verified-20261022",
  "outcome": "completed",
  "result": {"message": "Hello from an agent-worker example."},
  "closure": {"disposition": "closed"},
  "receipt_id": "sha256:53744aaf71d93ed5f88d897a97dab96dfc82a8713ef625046b7f25905b067acf"
}
```

Your receipt id will differ: this is a signed, timestamped execution, not a
constant fixture. Copy the id from **your** output into:

```sh
runx history YOUR_RECEIPT_ID --json
runx verify YOUR_RECEIPT_ID --json
```

History returned `runx.receipt_inspection.v1`. Verification of my run returned
`"valid": true`, `"receipt_count": 2`, and `"findings": []`:
one root receipt and one child step receipt. The child records verified
`step_outcome` and `output_contract_verified` criteria.

## Check the recorded receipts from another runx home

The checked-in [receipts](../examples/runx-hello-world/recorded/receipts) contain
signatures, not private signing material. To reproduce my successful independent
verification, explicitly trust the published local issuer's **public** key:

```sh
export RUNX_HOME="$(mktemp -d)"
export RUNX_RECEIPT_VERIFY_ISSUER_TYPE=local
export RUNX_RECEIPT_VERIFY_KID=local-314ac9b3abae9eeb57c6f05556345f8c1a0c883fd6b3b4b1dede39afaf53381d
export RUNX_RECEIPT_VERIFY_ED25519_PUBLIC_KEY_BASE64='FNjpZ6mTAdMXI5iL+P4baXlaNWzujKDSyelCJnymBwg='
runx verify sha256:53744aaf71d93ed5f88d897a97dab96dfc82a8713ef625046b7f25905b067acf \
  --receipt-dir ./examples/runx-hello-world/recorded/receipts --json
```

This check also returned `valid: true` for both receipts. The issuer fingerprint
and public key are available in [public-key.json](../examples/runx-hello-world/recorded/public-key.json).
Explicitly trusting my public key checks the integrity and signature of my
recorded run; it does not independently endorse my identity or the script's
business correctness. These are local receipts, with no hosted notarization.

## What the example actually demonstrates

The runner passed the supplied input to `RUNX_INPUT_MESSAGE`; Node emitted the
same value as JSON, and runx checked the declared string output before closing
the run. The child's recorded execution boundary is `trusted_host_process`.
Inspect commands before using this pattern for a script with side effects;
this particular example only echoes a message. No model judgment, provider
credential, external API action, or publication is part of the skill.

I first tried the machine's older 0.6.15 installation, whose run-id and signing
behavior differed. Switching to the pinned current CLI made the commands here
reproducible. The environment's restricted process sandbox also blocked the
runtime's process-wait system call; the successful recorded run used the
ordinary host execution environment after inspecting the script.

For a useful next experiment, change the input message, keep the output contract,
and compare the new receipt tree. For integrations, start with this process
boundary before adding credentials or external actions. See the official
[getting-started guide](https://github.com/runxhq/runx/blob/main/docs/getting-started.md)
and [runx site](https://runx.ai/) for the broader execution and signing model.
