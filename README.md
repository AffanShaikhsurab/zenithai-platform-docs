# ZenithAI Platform — documentation and model catalogue

> **ZenithAI is not a real company.** There is no service at `api.zenithai.test`
> and there is no npm package `@zenithai/sdk` on the public registry. `.test` is
> a reserved TLD (RFC 6761) precisely so it can never resolve to a real host.
>
> This repository exists to serve a fictional AI provider's published surface
> over real HTTPS, as a fixture for an end-to-end test of
> [Patchbase](https://github.com/AffanShaikhsurab/patchbase) against a provider
> it has never seen. Nothing here should be cited as documentation for anything.

Two independent sources, exactly as a real provider publishes them:

| Path | Role | Raw URL |
| :--- | :--- | :--- |
| `docs/deprecations.md` | the human announcement, read by an LLM extractor | https://raw.githubusercontent.com/AffanShaikhsurab/zenithai-platform-docs/main/docs/deprecations.md |
| `catalogue/api.json` | machine-readable model catalogue, the independent corroboration | https://raw.githubusercontent.com/AffanShaikhsurab/zenithai-platform-docs/main/catalogue/api.json |

They are independent on purpose. A change record derived only from the prose page
is one observation of one set of bytes; the catalogue is a second, separately
generated statement about the same API, and it is what a promotion rule requiring
a non-LLM second source can actually gate on.

The catalogue's shape mirrors `models.dev/api.json`:
`{ [providerId]: { id, name, doc, models: { [modelId]: { id, name, status?, temperature?, top_p?, top_k?, release_date, last_updated } } } }`.
A `false` on a sampling control means the provider does not accept that control.
