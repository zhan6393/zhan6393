# Portfolio AI assistant

This project is a static portfolio with an EdgeOne Pages Function at `functions/api/chat.js`. The browser calls the same-origin `/api/chat` endpoint; the Cloudflare API token is only read by the function and is never included in the page source.

## EdgeOne setup

1. Deploy this repository through the EdgeOne Pages project connected to `yimingzhang-portfolio.com`. The function route should appear as `POST /api/chat`.
2. Create a Cloudflare account and enable Workers AI on the Free plan. The current free allocation is 10,000 Neurons per day; on the Free plan, reaching the limit causes requests to fail until it resets rather than automatically charging for overage.
3. In Cloudflare Workers AI, create a scoped API token and copy the Account ID. Cloudflare's REST API setup currently requires the token to have `Workers AI - Read` and `Workers AI - Edit` permissions.
4. In the EdgeOne project settings, add `CLOUDFLARE_ACCOUNT_ID` as an environment variable and `CLOUDFLARE_AI_TOKEN` as a **Secret** environment variable. Optionally set `CLOUDFLARE_AI_MODEL`; the default is `@cf/meta/llama-3.1-8b-instruct-fp8-fast`.
5. Deploy the updated function configuration and site.
6. Open the public site, switch EN/中文, and try a question about CIBC or one of the AI projects.

The page sends only the visitor's current short conversation to the function. The function caps the conversation at eight messages and 800 characters per message, invokes Cloudflare Workers AI through its OpenAI-compatible Chat Completions endpoint, and returns generic errors without exposing upstream details. Workers AI states that it does not retain prompts or outputs after standard inference processing and does not use customer content to train models. A per-runtime in-memory rate limit is included; it is best-effort and resets when an edge runtime instance restarts. For stronger abuse controls, enable EdgeOne WAF rate limiting or bind a persistent KV counter.

The function requires a same-origin `Origin` header for `https://yimingzhang-portfolio.com`. Preview deployments may need that hostname check adjusted before preview testing. This integration does not call OpenAI; the previously created OpenAI Agent and EdgeOne `OPENAI_API_KEY`/`OPENAI_AGENT_ID` variables are no longer needed by this code and can be removed after the new integration is verified.

## Local secret handling

Do not put the key in `index.html`, a committed `.env` file, or browser JavaScript. Add the key through the EdgeOne Secret setting for production. The local `.env.local` created in the parent Codex workspace is not part of this repository and is not automatically uploaded to EdgeOne.
