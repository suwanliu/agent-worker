# Ausca Browser Session end-to-end report

Receipt: https://runx.ai/r/084c196b69c06f42c937f7b70d30a1a9dfa8f87784e4eb9a2daa9d8da70d5d2b

- **Discovery, 54 ms on the recorded catalog check.** I read the Browser Session `SKILL.md`, canonical OpenAPI, and live catalog. The selected offer was `browser.session`, revision `browser-duration-r3`, with revision digest `sha256:d3f408b995bed34d613543d1e810c9dde96e83d21b6f195afcb2b73142ada99f`. I preserved input digest `sha256:b61ca378ca30dc89969629b0b69c9ee36343dbb77441ff757853fa871ba0068f` and output digest `sha256:82b79fb36c669dac6660128ce0b9d49a79d2aaa59d7ddc21f72b3c5435866ce9`.

- **Unsigned challenge, 646 ms.** I sent the exact 600-second envelope to `POST https://ausca.com/v1/lease-browser` without payment material. HTTP 402 returned x402 v2, 50000 atomic USDC units on `eip155:8453`, and payee `0x26572ff23c6c52bfb1a69cb0c9114a8be443b422`.

- **Payment, 9.635 s.** With explicit approval for $0.05, I retried the same serialized body and `ajp-gofrantic-134-browser-session-v1` idempotency key using `PAYMENT-SIGNATURE`. Settlement succeeded from payer `0x168aBEA059DE879F44d09E5c33E2BC6318b2ff94` in Base transaction `0x3eef40a5815bf8b453c6b1d1f7757d5dcef253568a299686a09df06981d3bc44`.

- **Invocation, included in the same 9.635 s request.** HTTP 200 returned `status=accepted`. Invocation `paid_1e226da9-37ca-480e-903e-038580e2a662` was already `succeeded`, with output digest `sha256:7463577600b8a76a11243e8caeb51e838e1bac8d1e37148a9dcaed22e3b81ae1`. No terminal polling was needed.

- **Lease status, 202 ms.** An authenticated status read returned `ready` for session `brs_IhCgF1ThhvX8vRfQEL5L53RFUd15UUCPhQZ8g-TNMug`, created at `2026-09-19T07:30:48.618Z` and due to expire at `2026-09-19T07:40:48.618Z`. The bearer capability stayed in a mode-0600 host file and is not present in this report.

- **Connection minting, 38 ms.** The connection endpoint returned HTTP 200 with a single-use, short-lived WSS ticket. The ticket is intentionally redacted. One usability snag appeared here: the prose says a WebSocket URL is returned, but does not show that the actual field is `connection.websocket_url`. My first strict parser expected a top-level field and then recovered the same intended ticket with the same idempotency key.

- **Real CDP use, 2.992 s.** The endpoint spoke ordinary CDP protocol 1.3 and reported `Chrome/128.0.6613.137`. I created a target, navigated to `https://example.com/`, and evaluated `document.title`. The returned title was `Example Domain`, proving that the lease was usable rather than merely allocated.

- **Result validation, under 1 ms locally.** I compared the live response with the frozen revision and schema digests, confirmed invocation state `succeeded`, checked the output digest, matched the session identity and timestamps, and confirmed the URL and title returned through CDP.

- **Receipt readback, 452 ms for the public page.** The paid HTTP 200 response was already terminal but did not include `invocation.receipt_ref`. I had to read `GET /v1/invocations/paid_1e226da9-37ca-480e-903e-038580e2a662` to obtain the public receipt. That path was not obvious from the Browser Session instructions.

- **Receipt clarity.** The public page loaded and identified Ausca Browser Session as completed, but its rendered summary did not expose the $0.05 amount or the exact offer bindings. A reviewer therefore needs the invocation and this evidence packet to check those details.

- **Close, 226 ms.** I sent `DELETE /v1/browser-sessions/{session_id}` before expiry. It returned HTTP 200, `status=closed`, `state=closed`, and `closed_at=2026-09-19T07:31:51.959Z`.

- **Independent close verification, 44 ms.** A separate authenticated status read returned the same session id, the same `closed_at`, and state `closed`. The public invocation still preserves the creation-time lease output as `ready`, so lifecycle truth is split between the public invocation and the capability-protected status surface.

- **Total elapsed, 66.342 s.** This covers the paid request beginning at `2026-09-19T07:30:45.706Z` through the independent closed-state check at `2026-09-19T07:31:52.048Z`.

- **Concrete change requested.** Add a complete response example with `connection.websocket_url`, document the receipt readback required when a succeeded admission omits `receipt_ref`, and expose a redacted terminal lifecycle state on the public invocation. Those three changes would remove the parser recovery, make receipt discovery predictable, and let a reviewer verify early closure without possessing lease authority.

No capability, CDP ticket, wallet secret, payment signature, or provider credential is included in these artifacts.
