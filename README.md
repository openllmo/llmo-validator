# llmo-validator

Reference validator for [LLMO specification](https://llmo.org/spec/v0.1) documents.

Deployed at **[validate.llmo.org](https://validate.llmo.org)**.

## What this is

A single-file, client-side HTML validator for `llmo.json` documents. It:

- Parses pasted JSON, or fetches from a user-provided domain's `/.well-known/llmo.json`
- Validates against the [LLMO v0.1 JSON Schema](https://llmo.org/spec/v0.1/schema.json)
- Evaluates the three conformance tiers (Minimal, Standard, Strict) per §5.1 through §5.3
- Reports results with spec section citations, both as a rendered panel and as grep-friendly plain text
- Runs entirely in the user's browser; nothing is uploaded

## What it does not do

- Cryptographic signature verification. The validator reports whether a `signature` field is structurally present and well-formed, but does not verify the signature against a public key. Implementers verify signatures using the procedure in §4 of the spec and the test vectors at [llmo.org/spec/v0.1/test-vectors](https://llmo.org/spec/v0.1/test-vectors).
- Server-side execution. The entire validator is the static HTML file in this repo.

## Run locally

```bash
git clone https://github.com/openllmo/llmo-validator.git
cd llmo-validator
open index.html   # macOS
# or: xdg-open index.html  (Linux), start index.html  (Windows)
```

The validator works offline against a pasted document. URL mode requires network access to the target domain's `/.well-known/llmo.json`.

## Inlined schema and test vector

The LLMO JSON Schema and the strict test vector are embedded in `index.html` as `<script type="application/json">` blocks with stable `id` attributes (`llmo-schema`, `llmo-strict-vector`). The validator reads them at startup with `JSON.parse(document.getElementById(...).textContent)`.

Rationale: `llmo.org` does not set `Access-Control-Allow-Origin` headers on `/spec/v0.1/*` assets, so a runtime cross-origin fetch from `validate.llmo.org` is blocked by the browser. Inlining both assets removes the CORS dependency and lets the validator run fully offline from `file://` as well.

Trade-off: when the spec repo updates either the schema (`openllmo/llmo.org/spec/v0.1/schema.json`) or the strict test vector (`openllmo/llmo.org/spec/v0.1/test-vectors/signed-strict.json`), a paired PR in this repo must update the inlined copies. See "Validator version and update cadence" below for the workflow.

## Deployment

Hosted on Cloudflare Pages. Auto-deploys from `main` on push. Pull requests produce preview deployments at `<pr-number>.llmo-validator.pages.dev`.

Full deployment and infrastructure documentation: [openllmo/llmo.org/infrastructure/VALIDATOR.md](https://github.com/openllmo/llmo.org/blob/main/infrastructure/VALIDATOR.md).

## Contributing

Bug reports and improvements welcome via GitHub Issues and pull requests.

All commits must be DCO-signed. Add `Signed-off-by: Your Name <your@email>` to commit messages, or use `git commit -s`.

Code is licensed under MIT (see [LICENSE](LICENSE)). Contributions are accepted under the same license.

## Validator version and update cadence

The validator displays its version in the footer of every rendered page. The version corresponds to the git commit SHA at deploy time, plus a human-readable tag.

Rule-set changes that add or remove conformance checks require corresponding updates to the spec and its changelog at [llmo.org/spec/changelog](https://llmo.org/spec/changelog). Open an issue in `openllmo/llmo.org` first, ship the spec change, then update the validator to match.

Bug fixes and UX improvements that do not change rule semantics do not require spec coordination.

Inlined-asset updates: whenever the spec repo modifies `spec/v0.1/schema.json` or `spec/v0.1/test-vectors/signed-strict.json`, open a PR in this repo that replaces the corresponding `<script type="application/json">` block and bumps the validator version. The footer should also note the source spec-repo commit SHA for each inlined asset so deployed state is traceable.

## Related

- [LLMO Specification v0.1](https://llmo.org/spec/v0.1)
- [Test Vectors](https://llmo.org/spec/v0.1/test-vectors)
- [JSON Schema](https://llmo.org/spec/v0.1/schema.json)
- [Governance](https://llmo.org/about/governance)
