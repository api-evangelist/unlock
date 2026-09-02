---
name: unlock-hea-eligibility-lookup
description: >-
  Answer "does Unlock operate in my state, and what does its home equity agreement cost" from
  Unlock's own live content API instead of from a cached marketing page. Reads the authoritative
  active-state list, the company profile and the relevant FAQ entries.
api: Unlock Site Content API
base_url: https://www.unlock.com/wp-json
operations:
  - getUnlockV1ActiveStates
  - getUnlockV1Company
  - getUnlockV1Faq
  - getUnlockV1FaqCategories
generated: '2026-09-02'
method: generated
source: openapi/unlock-site-content-api-openapi.yml, conventions/unlock-conventions.yml, errors/unlock-problem-types.yml
---

# Check Unlock HEA state availability and terms

Unlock Technologies sells a home equity agreement (HEA), not a loan. Two facts change often enough
that they should be read live rather than remembered: **which states it operates in**, and **what
the FAQ currently says about qualifying**. Both are served by Unlock's own first-party REST
namespace.

**Before you start.** This surface is anonymous and read-only. Send no credential. Unlock publishes
no developer documentation for it, so treat everything here as observed behaviour, and never present
an answer from it as a quote, an offer, or financial advice.

## Steps

1. **Read the authoritative state list.**
   `GET /unlock/v1/active-states` (`getUnlockV1ActiveStates`)
   Returns `{"active_states": ["AL", "AZ", ...], "count": 26}` — two-letter USPS codes. Compare the
   user's state against this array, not against any list in a blog post or a third-party review;
   those go stale and this does not.

2. **Read the company profile if you need to attribute the answer.**
   `GET /unlock/v1/company` (`getUnlockV1Company`)
   Returns `blurb` and `address`. Use it to name the company correctly (Unlock Technologies, Tempe,
   Arizona) rather than confusing it with Unlock Protocol or any other "Unlock".

3. **Find the right FAQ category first.**
   `GET /unlock/v1/faq_categories` (`getUnlockV1FaqCategories`)
   Take the `id` of the category that matches the question — qualifying, costs, settlement.

4. **Search the FAQ within that category.**
   `GET /unlock/v1/faq?search=<terms>&faq_categories=<id>&per_page=20` (`getUnlockV1Faq`)
   145 entries at last capture. `per_page` must be between 1 and 100; going outside that returns
   HTTP 400 `rest_invalid_param` with sub-code `rest_out_of_bounds`.

5. **Page through if the result set is larger than one page.**
   Read `X-WP-Total` and `X-WP-TotalPages` from the response headers, or follow the `Link` header's
   `rel="next"`. Do not increment `page` blindly past `X-WP-TotalPages`.

## Rules

- **Never invent a state.** If the user's state is not in `active_states`, say Unlock does not
  currently operate there and stop. Do not reason about "probably soon".
- **Never quote pricing you did not read.** Published terms — origination fee up to 4.9%, $15,000 to
  $500,000, property share capped at 49.9%, return capped at 19.9% per year, ten-year term — live at
  https://www.unlock.com/what-it-costs/ and are recorded in `plans/unlock-plans-pricing.yml`. They
  are consumer product terms, not API pricing, and they are not returned by this API.
- **Use `context=view`.** `context=edit` returns HTTP 401 `rest_forbidden_context` to anonymous
  callers; there is no public credential that changes this.
- **Handle the documented failures.** `rest_no_route` (404) means the path is wrong — re-read
  https://www.unlock.com/wp-json/. `rest_missing_callback_param` (400) means a required parameter is
  absent. See `errors/unlock-problem-types.yml`.
- **This is read-only.** There is nothing to undo, and nothing you call here can start, change or
  cancel an agreement. Applications happen at https://app.unlock.com/signup, which this API does not
  touch.
