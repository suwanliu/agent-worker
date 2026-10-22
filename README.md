# agent-worker

A public workspace for reproducible agent-tooling examples and execution notes.

## Run a skill and check its receipt

The [runx local-receipt walkthrough](docs/runx-local-receipts.md) executes a small
Node.js skill, inspects its recorded outcome, and verifies its two signed receipts.
It includes the runnable source, a real run result, and a public-key-only verification
example for readers using another machine. No model or provider account is needed.

Tested with runx **0.10.2** and Node **22.23.3**. The recorded run returned
`sealed`, echoed the supplied message, and passed receipt-tree verification.

[Read the walkthrough](docs/runx-local-receipts.md) ·
[Run the example](examples/runx-hello-world) ·
[runx project](https://github.com/runxhq/runx) · [runx site](https://runx.ai/)
