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

The Text Generation API is the primary surface of the ZenithAI Platform. A request carries the model, the prompt, and the generation controls at the top level of the request body:

```http
POST https://api.zenithai.test/v1/completions
content-type: application/json

{
  "model": "zen-flux-3",
  "prompt": "Summarise the following incident report.",
  "max_tokens": 512,
  "temperature": 0.7
}
```

The response carries the generated text in `output_text`:

```json
{
  "id": "cmpl_01HZ",
  "model": "zen-flux-3",
  "output_text": "The outage began at 04:12 UTC ...",
  "usage": { "input_tokens": 214, "output_tokens": 96 }
}
```

The official Node client (`@zenithai/sdk`) exposes the same shape through `completions.generate`:

```ts
import { ZenithAI } from "@zenithai/sdk";

const zenith = new ZenithAI({ apiKey: process.env.ZENITHAI_API_KEY });

const res = await zenith.completions.generate({
  model: "zen-flux-3",
  prompt: "Summarise the following incident report.",
  max_tokens: 512,
  temperature: 0.7,
});

console.log(res.output_text);
```

The Python client (`zenithai`) exposes `client.completions.generate(...)` with the same keyword arguments and the same `output_text` attribute on the response.

### Supported generation controls

| Parameter     | Type    | Default | Notes                                                        |
| ------------- | ------- | ------- | ------------------------------------------------------------ |
| `max_tokens`  | integer | 256     | Upper bound on generated tokens. Sent at the top level.       |
| `temperature` | number  | 1.0     | Sampling temperature between 0 and 2. Sent at the top level.  |
| `top_p`       | number  | 1.0     | Nucleus sampling mass. Sent at the top level.                 |
| `top_k`       | integer | unset   | Restricts sampling to the k most likely tokens.               |
| `stop`        | array   | unset   | Up to four stop sequences.                                    |

## Deprecation history

All deprecations are listed in the following sections, with the most recent announcements first.

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

There are no deprecated request parameters on the ZenithAI Platform API at this time. Every parameter listed under **Supported generation controls** above is accepted at the top level of the request body by every active model.

## Machine-readable catalogue

The model catalogue is published separately as JSON, independent of this page, and is the source of truth for which model IDs exist and which generation controls each model accepts:

```
https://raw.githubusercontent.com/AffanShaikhsurab/zenithai-platform-docs/main/catalogue/api.json
```

Each model entry carries a boolean per sampling control (`temperature`, `top_p`, `top_k`). A `false` means the model does not accept that control.
