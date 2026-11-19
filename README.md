<!-- Generated from the SocQ API catalog and docs. Do not edit by hand. -->
# Creator API

[![SocQ Creator API](https://img.shields.io/badge/API-SocQ%20Creator%20API-F64C31)](https://socq.ai/apis/creator?utm_source=github&utm_medium=readme&utm_campaign=socq-creator-api)
[![API documentation](https://img.shields.io/badge/Docs-docs.socq.ai-111827)](https://docs.socq.ai/api-manual/creator/overview)
[![License: MIT](https://img.shields.io/badge/License-MIT-2563EB)](LICENSE)
[![Check examples](https://github.com/SocQAPI/creator-api/actions/workflows/check.yml/badge.svg)](https://github.com/SocQAPI/creator-api/actions/workflows/check.yml)

3 endpoints for public Creator data through one SocQ API key, with runnable cURL, Node.js and Python examples and a valid request body for every endpoint.

[Browse Creator APIs](https://socq.ai/apis/creator?utm_source=github&utm_medium=readme&utm_campaign=socq-creator-api) · [Get an API key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-creator-api) · [Documentation](https://docs.socq.ai/api-manual/creator/overview) · [All SocQ examples](https://github.com/SocQAPI/socq-examples)

## Endpoints

Prices are in SocQ credits; 1 credit = $0.005.

| Endpoint | publicId | What it returns | Price | Launched |
| --- | --- | --- | --- | --- |
| [Influencer Email Finder API](#influencer-email-finder-api) | `creator/find-emails` | Find the public business emails of a creator from an Instagram, TikTok, YouTube, X or Facebook profile, each with the pages it was found on. | 0.5 credits ($0.0025) per call + 7.5 credits ($0.0375) per result; maximum 8 credits ($0.04) per call | 2026-11-17 |
| [Social Media Profile Finder API](#social-media-profile-finder-api) | `creator/find-social-profiles` | Find the accounts a creator runs on other networks from one profile URL, with a 0 to 1 confidence and the evidence behind each match. | 8 credits ($0.04) per call | 2026-11-17 |
| [Age and Gender Detection API](#age-and-gender-detection-api) | `creator/age-gender` | Estimate a creator's age range and gender from the profile photo on Instagram, TikTok, YouTube, X or Threads, with a confidence score. | 1 credit ($0.005) per call | 2026-11-17 |

## Quick start

1. Create an API key at [socq.ai/dashboard/api-key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-creator-api).
2. Export it. The examples read only `SOCQ_API_KEY`.

```bash
git clone https://github.com/SocQAPI/creator-api.git
cd creator-api
export SOCQ_API_KEY="your-api-key"
```

3. Run any endpoint by its publicId. Without a second argument the script sends `payloads/<slug>.json`; pass a JSON file or an inline JSON object to change the input.

```bash
# Node.js 20+
node examples/node/socq.mjs creator/find-emails

# Python 3.8+ (standard library only)
python3 examples/python/socq.py creator/find-emails payloads/find-emails.json

# cURL + jq
bash examples/curl/socq.sh creator/find-emails '{"url":"https://www.instagram.com/miguelangeles"}'
```

Each script submits the task, polls it until it succeeds or fails, follows every result page, and writes the records to `output/<slug>-<task_id>.json`. Run `npm run check` to validate the payloads and scripts offline.

Never expose `SOCQ_API_KEY` in browser code, mobile apps, public repositories, screenshots, fixtures or logs.

## How the API works

Base URL `https://api.socq.ai`. Every request sends the API key as a bearer token:

```http
Authorization: Bearer <SOCQ_API_KEY>
Content-Type: application/json
```

All endpoints are asynchronous:

```text
POST /v1/<publicId>  ->  data.task_id  ->  GET /v1/tasks/{task_id}  ->  queued | running | succeeded | failed
```

1. **Submit** the JSON body to `POST https://api.socq.ai/v1/<publicId>`. The response returns `data.task_id` immediately; it means the task was created, not that data is ready.
2. **Poll** `GET https://api.socq.ai/v1/tasks/{task_id}` until `data.status` is `succeeded` or `failed` (a failed task carries `data.error_message`).
3. **Read results.** A succeeded task returns normalized records in `data.results.items`. Use `limit` (default 50, max 500) and pass `data.results.next_cursor` as `cursor` while `data.results.has_more` is true. `fields` selects up to 50 comma-separated fields or dot paths.
4. **Raw files (optional).** `GET https://api.socq.ai/v1/tasks/{task_id}/files` lists raw JSONL exports.

```bash
curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100" \
  -H "Authorization: Bearer $SOCQ_API_KEY"

curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100&cursor=$NEXT_CURSOR" \
  -H "Authorization: Bearer $SOCQ_API_KEY"
```

`callback_url` (where an endpoint accepts it) makes SocQ POST to your URL when the task reaches a terminal state; keep polling as a fallback.

**Billing.** A submit may reserve credits from `results_limit` or the number of submitted inputs; when the task finishes SocQ settles the charge from the actual result count and returns unused reserved credits. `402` means insufficient credits.

**Errors.** `400` invalid body or unsupported field, `401` missing or invalid key, `402` insufficient credits, `403` IP allowlist or task owned by another account, `404` endpoint or task not found, `422` input validation failed, `429` rate or credit-window limit (respect `Retry-After`), `500` submission failed. A client-side timeout does not cancel a task: resume polling the same `task_id` instead of submitting again.

Reference: [Authentication](https://docs.socq.ai/api-manual/agent-api/authentication) · [Task status and results](https://docs.socq.ai/api-manual/agent-api/status) · [Task files](https://docs.socq.ai/api-manual/agent-api/files) · [Callbacks](https://docs.socq.ai/api-manual/agent-api/callbacks) · [Errors](https://docs.socq.ai/api-manual/agent-api/errors)

## Endpoint reference

### Influencer Email Finder API

Find the public business emails of a creator from an Instagram, TikTok, YouTube, X or Facebook profile, each with the pages it was found on.

- **Request:** `POST https://api.socq.ai/v1/creator/find-emails`
- **publicId:** `creator/find-emails`
- **Price:** 0.5 credits ($0.0025) per call + 7.5 credits ($0.0375) per result; maximum 8 credits ($0.04) per call
- **Launched:** 2026-11-17
- **Example body:** [`payloads/find-emails.json`](payloads/find-emails.json)

Give it one creator profile and it goes looking for the emails that creator makes public: in the bio, behind link-in-bio pages, on linked websites and on the creator's accounts on other networks. Every email comes with the pages it was found on, so your outreach team can see where an address came from before using it.

Provide one Instagram, TikTok, YouTube, X or Facebook profile URL in url; a YouTube channel can be given by @handle or channel ID. When at least one email is found the result is one row and the lookup costs 8 credits. When none is found the task still succeeds, returns no rows and costs 0.5 credits. A profile that does not exist fails with a refund.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `url` | string | Yes | Public profile URL, such as https://www.instagram.com/<username> or https://www.tiktok.com/@<username>. |

```bash
curl -X POST "https://api.socq.ai/v1/creator/find-emails" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://www.instagram.com/miguelangeles"
}'
```

Full docs: [https://docs.socq.ai/api-manual/creator/find-emails](https://docs.socq.ai/api-manual/creator/find-emails) · Landing page: [socq.ai/apis/creator/find-emails](https://socq.ai/apis/creator/find-emails?utm_source=github&utm_medium=readme&utm_campaign=socq-creator-api)

### Social Media Profile Finder API

Find the accounts a creator runs on other networks from one profile URL, with a 0 to 1 confidence and the evidence behind each match.

- **Request:** `POST https://api.socq.ai/v1/creator/find-social-profiles`
- **publicId:** `creator/find-social-profiles`
- **Price:** 8 credits ($0.04) per call
- **Launched:** 2026-11-17
- **Example body:** [`payloads/find-social-profiles.json`](payloads/find-social-profiles.json)

Know a creator from Instagram and want their TikTok and YouTube too? Pass the profile you have and get one row for every other account found for the same person. Each match carries a confidence from 0 to 1 and the evidence behind it, such as the two profiles linking to each other or sharing a website, so you can keep only the matches you trust.

Provide one Instagram, TikTok, YouTube, X or Facebook profile URL in url. Each lookup costs 8 credits and returns one row per account found on another network; the profile you sent is not repeated in the results. A profile that does not exist fails with a refund.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `url` | string | Yes | Public profile URL, such as https://www.instagram.com/<username> or https://www.tiktok.com/@<username>. |

```bash
curl -X POST "https://api.socq.ai/v1/creator/find-social-profiles" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://www.instagram.com/miguelangeles"
}'
```

Full docs: [https://docs.socq.ai/api-manual/creator/find-social-profiles](https://docs.socq.ai/api-manual/creator/find-social-profiles) · Landing page: [socq.ai/apis/creator/find-social-profiles](https://socq.ai/apis/creator/find-social-profiles?utm_source=github&utm_medium=readme&utm_campaign=socq-creator-api)

### Age and Gender Detection API

Estimate a creator's age range and gender from the profile photo on Instagram, TikTok, YouTube, X or Threads, with a confidence score.

- **Request:** `POST https://api.socq.ai/v1/creator/age-gender`
- **publicId:** `creator/age-gender`
- **Price:** 1 credit ($0.005) per call
- **Launched:** 2026-11-17
- **Example body:** [`payloads/age-gender.json`](payloads/age-gender.json)

Sort a creator list by demographic before you spend time reviewing content. The estimate is read from the face in the profile photo and comes back as an age range, a gender and a confidence score. It works best on personal accounts with a clear headshot; logos and group photos leave the age and gender fields empty.

Provide one Instagram, TikTok, YouTube, X or Threads profile URL in url. Age and gender are estimated from the face in the profile photo. Each lookup costs 1 credit and returns one row. A profile that does not exist fails with a refund.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `url` | string | Yes | Public profile URL, such as https://www.instagram.com/<username> or https://www.tiktok.com/@<username>. |

```bash
curl -X POST "https://api.socq.ai/v1/creator/age-gender" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://x.com/levelsio"
}'
```

Full docs: [https://docs.socq.ai/api-manual/creator/age-gender](https://docs.socq.ai/api-manual/creator/age-gender) · Landing page: [socq.ai/apis/creator/age-gender](https://socq.ai/apis/creator/age-gender?utm_source=github&utm_medium=readme&utm_campaign=socq-creator-api)

## Repository contents

| Path | Purpose |
| --- | --- |
| [`examples/curl/socq.sh`](examples/curl/socq.sh) | Submit, poll and paginate with cURL and jq |
| [`examples/node/socq.mjs`](examples/node/socq.mjs) | Same workflow in Node.js 20+ (no dependencies) |
| [`examples/python/socq.py`](examples/python/socq.py) | Same workflow in Python 3.8+ (standard library) |
| [`payloads/`](payloads) | One documented request body per endpoint |
| [`endpoints.json`](endpoints.json) | Endpoint list, prices and request schemas used by the scripts and checks |
| [`fixtures/`](fixtures) | Synthetic submit and task response shapes |
| [`docs/production-notes.md`](docs/production-notes.md) | Retries, timeouts, pagination and data handling |
| [`scripts/check.mjs`](scripts/check.mjs) | Offline check: payloads, syntax and secrets |

## Responsible use and platform scope

- Use only publicly accessible Creator data and the fields supported by each endpoint.
- Do not use the examples to access restricted content, login-only surfaces, authentication controls or non-public personal information.
- SocQ is not an official API of the represented platform and is not affiliated with or endorsed by it. Platform names and trademarks belong to their respective owners.
- Before production use, assess the laws, platform terms, privacy obligations and retention requirements that apply to your organization and use case.
- Collect only the fields needed for a defined purpose, restrict access, set retention periods, and support correction or deletion workflows where required.

This section describes the public-data boundary; it is not legal advice.

## License

[MIT](LICENSE)
