# Houselights

An audience-finding agent for independent theatre teams. Small productions know what their show feels like, but not necessarily where to find people who will connect with it. Houselights turns two or three cultural references into a traceable audience research and outreach plan.

## Run locally
Node.js 22 or newer, no dependencies. Set server-only environment variables `QLOO_API_KEY` and `GEMINI_API_KEY`, then run `npm start` and visit http://127.0.0.1:8787. Optional `PORT` defaults to 8787. `.env.example` documents the variables; the app does not automatically load .env files. Never put keys in frontend code or commit them.

## How it works
Gemini (`gemini-3.8-flash`) uses native function calling to choose Qloo searches, cross-domain categories and nearby-place research. The backend validates IDs against search results and limits tool rounds. Qloo data is reduced to useful evidence fields before being returned to the agent. Final recommendations cite Qloo entity IDs; raw affinity values are not purchase probabilities. The UI exposes tool parameters, cache hits and usage.

- `GET /search?query=…&take=3`: entity resolution with candidate types and years.
- `GET /v2/insights?filter.type=urn:entity:{movie,artist,book,podcast,brand}&signal.interests.entities=…`: aggregate cross-domain taste affinity.
- `GET /v2/insights?filter.type=urn:entity:place&signal.interests.entities=…&filter.location=POINT(lon lat)&filter.location.radius=3000`: taste-ranked local outreach candidates. The current entity parameter guide does not list `signal.location`; documented geography filters are used instead.
- Base: https://hackathon.api.qloo.com, `X-Api-Key` header.

Without Qloo the app cannot obtain its cross-domain entities, affinities or taste-ranked local places. The LLM-only toggle provides a deliberately unvalidated generic comparison using the original brief only.

## Output
Audience hypothesis, three hooks, five local candidates with addresses, three social drafts and a two-week schedule. All are suggestions for human review. No outreach or publishing functionality exists. English interface supports Korean input.

## Safety and limits
Keys stay on the server. A daily cap (`DAILY_RUN_LIMIT`, default 200 runs per UTC day) returns a friendly message once reached; like the other limits it is an in-memory counter per serverless instance, so under heavy traffic the effective global cap can be a small multiple of this value. Six runs per IP per hour, five Gemini calls total maximum (including the independent baseline and at most one repair), bounded tool arguments, 6,000 output tokens per model call, 35-second Qloo and 90-second model timeouts and one-hour private in-memory Qloo cache (max 300 entries). Rate limits/cache reset on cold starts and are per instance, not a distributed production abuse boundary. Add a durable rate limiter before a high-traffic launch. Provider usage quotas should also be configured. Avoid personal data. No demographic estimates are fabricated. Coordinates are model-estimated neighborhood centers, not verified geocoding. Check the displayed location trace and verify maps; venue address or map-pin confirmation should be added before real-world use. Entity matching can be ambiguous; inspect candidate years/types in the trace. Affinities reflect aggregate associations, not causation, ticket demand, or individual identity. Data coverage varies; no fabricated replacement places are shown. An empty place search is retried once at twice the radius (up to 10 km). Incomplete model JSON or invalid evidence references trigger one repair request, with explicit missing-field feedback. If still incomplete, the UI shows only grounded recommendations and marks the result partial with a list of gaps. A 108-second request budget bounds provider waits; partial results can be returned on provider failure. This prototype is not an ad targeting or individual profiling system.

## Hosting
`api/plan.js` exports a Vercel-style handler; static frontend files require no build. Set both environment variables in the host and use Node 22. `vercel.json` configures a 120-second function duration (within Hobby Fluid Compute’s documented 300-second maximum); enable Fluid Compute in the Vercel project. Classic Hobby function limits without Fluid Compute may be lower. Do not publish private cache or Qloo response fixtures in source control. Public repository and live demo are pending owner approval; this local build alone is not a complete competition submission.

## Configuration and license
Change the product name in `config.js`. MIT, copyright WooYoung Choi; also visible in About. This code was written with an AI coding agent and reviewed/tested locally. The application itself uses Gemini to orchestrate Qloo tools.

## References
- https://docs.qloo.com/reference/qloo-llm-hackathon-developer-guide
- https://docs.qloo.com/reference/parameters
- https://docs.qloo.com/reference/available-parameters-by-entity-type
- https://github.com/qloo/qloo-hackathon-kit
- https://github.com/qloo/qloo-hackathon-kit/blob/main/docs/SAFE_USE.md
- https://www.are.na/ (connection-first research organization; no assets copied)

### Deployment checklist (owner approval required)
1. Publish only the `app` directory as repository root after owner approval. Never publish sibling files, private results, diagnostic logs or key files.
2. Import that repository into Vercel, Framework Preset: Other, no build command, repository root as output/static source. Native `api/plan.js` is detected as a Node function.
3. Enable Fluid Compute and verify Node.js 22.x and a 120-second function duration. Set QLOO_API_KEY and GEMINI_API_KEY in server environment settings for the desired deployment environments; redeploy after changes.
4. Smoke-test both samples and inspect partial-result notices, usage and evidence. Local tests are not deployment verification.

`PLAN_DEBUG=1` is optional local-only model-response logging; leave unset in production. It can log user briefs/model output, so never commit logs. Provider token counts reflect returned usage metadata; timed-out requests can still incur unreported provider usage. The independent baseline is generated concurrently using only the initial brief. No analytics, geocoder, social publishing or booking system is integrated.
