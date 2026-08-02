# Model and API deprecations

See which ZenithAI models are active, deprecated, or retired, and find retirement dates and recommended replacements for models and API parameters.

---

As safer and more capable models launch, ZenithAI regularly retires older ones. Applications relying on ZenithAI models may need occasional updates to keep working. Impacted customers are always notified by email and in the documentation.

This page lists all API deprecations for the ZenithAI Platform, along with recommended replacements.

## Overview

ZenithAI uses the following terms to describe the model lifecycle:

* **Active:** The model is fully supported and recommended for use.
* **Legacy:** The model will no longer receive updates and may be deprecated in the future.
* **Deprecated:** The model is still functional but no longer recommended. ZenithAI provides a recommended replacement and assigns a retirement date.
* **Retired:** The model is no longer available for use. Requests to retired models will fail.

The dates on this page apply to ZenithAI-operated platforms: the ZenithAI Platform API at `api.zenithai.test` and the ZenithAI Console. Partner-operated deployments set their own retirement schedules, so a model's lifecycle status and dates can differ there.

## Migrating to replacements

Once a model is deprecated, migrate all usage to a suitable replacement before the retirement date. Requests to models past the retirement date will fail with a `404 model_not_found`.

To help measure the performance of replacement models on your tasks, consider thorough testing of your applications with the new models well before the retirement date.

## Notifications

ZenithAI notifies customers with active deployments for models with upcoming retirements, providing at least 90 days' notice before model retirement for publicly released models.

## Auditing model usage

To help identify usage of deprecated models and deprecated request shapes, customers can access an audit of their API usage. Follow these steps:

1. Go to the **Usage** page in the ZenithAI Console.
2. Click **Export**.
3. Review the downloaded CSV to see usage broken down by API key, model, and request shape.

This audit will help you locate any instances where your application is still using deprecated models, allowing you to prioritize updates before the retirement date.

## Best practices

1. Regularly check the documentation for updates on model and parameter deprecations.
2. Test your applications with newer models well before the retirement date of your current model.
3. Update your code to use the recommended replacement as soon as possible.
4. Contact the support team if you need assistance with migration or have any questions.

## Model status

Current and recently retired models are listed in the following table with their status:

| API model name | Current state | Deprecated | Tentative retirement date          |
| -------------- | ------------- | ---------- | ---------------------------------- |
| zen-flux-3     | Active        | N/A        | Not sooner than October 1, 2027     |
| zen-flux-2     | Active        | N/A        | Not sooner than June 12, 2027       |
| zen-nano-1     | Active        | N/A        | Not sooner than April 30, 2027      |
| zen-spark-1    | Retired       | July 9, 2025 | January 14, 2026                  |

## The Text Generation API

The Text Generation API is the primary surface of the ZenithAI Platform. A request carries the model and the prompt at the top level, and every generation control inside the nested `generation` object:

```http
POST https://api.zenithai.test/v1/responses
content-type: application/json

{
  "model": "zen-flux-3",
  "prompt": "Summarise the following incident report.",
  "generation": {
    "limits": { "output_tokens": 512 },
    "sampling": { "temperature": 0.7 }
  }
}
```

The response carries the generated text in the first element of `content`:

```json
{
  "id": "resp_01HZ",
  "model": "zen-flux-3",
  "content": [{ "type": "output_text", "text": "The outage began at 04:12 UTC ..." }],
  "usage": { "input_tokens": 214, "output_tokens": 96 }
}
```

The official Node client (`@zenithai/sdk`) exposes the same shape through `responses.create`:

```ts
import { ZenithAI } from "@zenithai/sdk";

const zenith = new ZenithAI({ apiKey: process.env.ZENITHAI_API_KEY });

const res = await zenith.responses.create({
  model: "zen-flux-3",
  prompt: "Summarise the following incident report.",
  generation: {
    limits: { output_tokens: 512 },
    sampling: { temperature: 0.7 },
  },
});

console.log(res.content[0].text);
```

The Python client (`zenithai`) exposes `client.responses.create(...)` with a `generation` mapping argument of the same shape and the same `content` list on the response.

### Supported generation controls

Every control below is sent inside `generation`, never at the top level of the request body.

| Parameter      | Sent as                              | Default | Notes                                            |
| -------------- | ------------------------------------ | ------- | ------------------------------------------------ |
| `output_tokens`| `generation.limits.output_tokens`    | 256     | Upper bound on generated tokens.                  |
| `temperature`  | `generation.sampling.temperature`    | 1.0     | Sampling temperature between 0 and 2.             |
| `top_p`        | `generation.sampling.top_p`          | 1.0     | Nucleus sampling mass.                            |
| `top_k`        | `generation.sampling.top_k`          | unset   | Restricts sampling to the k most likely tokens.   |
| `stop`         | `generation.stop`                    | unset   | Up to four stop sequences.                        |

## Deprecation history

All deprecations are listed in the following sections, with the most recent announcements first.

### 2026-08-02: flat generation parameters and the completions.generate method

On August 2, 2026, ZenithAI notified developers that the flat top-level generation parameters on the Text Generation API are deprecated in favour of the nested `generation` object, that the client method `completions.generate` is renamed, and that the generated text moves out of `output_text`.

Requests that set `temperature` or `max_tokens` at the top level of the request body return a `400 invalid_request_error` on `zen-flux-3`, `zen-flux-2` and `zen-nano-1`.

The top-level form stops being accepted on March 2, 2027. Until that date a top-level value is accepted and ignored, which means a request that looks like it is setting a temperature is in fact running at the default.

ZenithAI publishes the migration as a table of mappings. Every row is one edit; the four rows must be applied together, because a request that nests `temperature` while still calling `completions.generate` is rejected by the old route and a request that calls `responses.create` with a top-level `max_tokens` is rejected by the new one.

| From                   | To                                | Transform | New location   |
| ---------------------- | --------------------------------- | --------- | -------------- |
| `temperature`          | `generation.sampling.temperature` | `move`    | request body   |
| `max_tokens`           | `generation.limits.output_tokens` | `move`    | request body   |
| `completions.generate` | `responses.create`                | `rename`  | client method  |
| `output_text`          | `content[0].text`                 | `move`    | response body  |

Note that `max_tokens` is both moved and renamed: the nested field is called `output_tokens`, not `max_tokens`. There is no `generation.limits.max_tokens`.

`@zenithai/sdk` 3.x keeps `completions.generate` as a deprecated alias that forwards to the new route only when no top-level generation control is present. The alias is removed in 4.0.

### 2025-07-09: ZenithAI Spark 1 model

<Note>
  This model was retired January 14, 2026.
</Note>

On July 9, 2025, ZenithAI notified developers using the ZenithAI Spark 1 model of its upcoming retirement on the ZenithAI Platform API.

| Retirement date  | Deprecated model | Recommended replacement |
| ---------------- | ---------------- | ----------------------- |
| January 14, 2026 | `zen-spark-1`    | none published          |

ZenithAI published no direct replacement for this model. Customers were asked to evaluate the currently active models against their own workloads and choose for themselves.

## API parameter deprecations

ZenithAI occasionally deprecates request parameters that no longer apply to current models. Deprecated parameters remain in the SDK request types so existing code continues to type-check, but their behavior changes per model.

| Parameter                    | Status                        | Behavior                                                                                        | Recommended replacement                        |
| ---------------------------- | ----------------------------- | ----------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| `temperature` (top level)    | Deprecated (all active models) | Accepted and ignored today; returns a `400 invalid_request_error` from March 2, 2027.            | Send `generation.sampling.temperature` instead. |
| `max_tokens` (top level)     | Deprecated (all active models) | Accepted and ignored today; returns a `400 invalid_request_error` from March 2, 2027.            | Send `generation.limits.output_tokens` instead. |
| `top_p`, `top_k` (top level) | Deprecated (all active models) | Accepted and ignored today; returns a `400 invalid_request_error` from March 2, 2027.            | Send them inside `generation.sampling` instead. |

## Machine-readable catalogue

The model catalogue is published separately as JSON, independent of this page, and is the source of truth for which model IDs exist and which generation controls each model accepts:

```
https://raw.githubusercontent.com/AffanShaikhsurab/zenithai-platform-docs/main/catalogue/api.json
```

Each model entry carries a boolean per sampling control (`temperature`, `top_p`, `top_k`). A `false` means the model does not accept that control at the top level of the request body.
