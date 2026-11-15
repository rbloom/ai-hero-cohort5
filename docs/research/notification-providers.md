# Notification providers for Cadence v1

Research for [#2](https://github.com/rbloom/ai-hero-cohort5/issues/2) (map: [#1](https://github.com/rbloom/ai-hero-cohort5/issues/1)).
All sources accessed **2026-11-15**. This document surveys and compares. It does **not** pick a provider; that is a separate ticket.

**What Cadence needs in v1:** an in-app bell and feed, email, event-driven notifications, Broadcasts (Platform Admin or Instructor to many Recipients), and batched email digests, for all four kinds of Recipient (Student, Instructor, Platform Admin, Team Admin). The stack is React Router 8, React 19, SQLite/Drizzle, Tailwind and Radix, plus lucide icons.

**How to read the scores:**

- **Drop-in (0 to 5):** a prebuilt bell and feed, how far its styling can be bent toward a Tailwind + Radix look, and React 19 support.
- **Headless (0 to 5):** hooks or a client for building our own bell and feed with our own Radix components.

The scores are my own judgement, based on the cited facts.

---

## 1. Knock (cloud only)

| Aspect | Finding |
| --- | --- |
| Free tier | **10,000 messages/month**. Workflows, **broadcasts**, delivery channels, stored profiles and team members are unlimited, and **feeds (in-app) and batching/digest are included**. Paywalled: removing Knock branding and email client previews (Starter, $250/mo). Per-tenant branding and i18n are Enterprise only. [pricing][knock-pricing] |
| Batching / digest | A native **batch step** collects per recipient, with a fixed, dynamic, relative or sliding window and an optional batch key. A batch can stay open for up to 90 days. Templates can render only the first or last 10 activities unless you are on Enterprise. [batch][knock-batch], [plans][knock-plans] |
| Broadcasts / audiences | Broadcasts are listed as unlimited on Free [pricing][knock-pricing]. Audiences are a CLI-managed resource [CLI][knock-cli]. |
| Email | Orchestrates delivery through a connected email provider (e.g. SES, SparkPost, Resend) [SparkPost integration][knock-sparkpost]. Plan on a separate ESP account. |
| React drop-in | `@knocklabs/react` 0.15.1 (MIT, published 2026-11-06) ships `NotificationIconButton`, `NotificationFeedPopover` and `NotificationFeed` [overview][knock-react], [npm][npm-knock]. Styling uses `--rnf-*` CSS variables, `rnf-` class names you can replace, and a `colorMode` light/dark prop [feed docs][knock-feed]. Peer deps: **React ^17, ^18 or ^19** [package.json][knock-pkg]. **Score 4/5**: it supports React 19, but restyling to Tailwind means overriding CSS variables rather than passing classes. |
| React headless | `useKnockFeed`, `useNotifications`, `useNotificationStore`, `useFeedSettings` and `useKnockClient` are exported from `@knocklabs/react-core` [source][knock-core-src]. **Score 5/5** |
| Feed auth | By default the client uses the public API key and a user id, so anyone could impersonate a user. **Enhanced security mode** requires an RS256 JWT that the backend signs with a Knock signing key (`sub` = user id), passed as `userToken`. It can carry grants and be refreshed with `onUserTokenExpiring` [auth][knock-auth]. |
| Self-host / OSS | No self-host option. The SDKs are MIT [repo][knock-repo]. |
| Lock-in / export | The CLI `knock pull`/`push` covers workflows, layouts, partials, translations, audiences and more, so templates can live in git [CLI][knock-cli]. There is also an OpenAPI-described REST API [API ref][knock-api]. |
| EU residency | **Not available yet.** Knock's docs say "We expect to support EU data residency by the end of 2026." [plans][knock-plans]. Its privacy policy relies on SCCs and the EU-US DPF [privacy][knock-privacy]. |

## 2. Novu Cloud

| Aspect | Finding |
| --- | --- |
| Free tier | **10,000 workflow runs/month**, all channels (email, in-app and others), **US and EU data residency**, up to **5 workflows**, 2 environments, 3 team members, and 24h activity-feed retention. Pro ($30/mo) removes Novu branding [pricing][novu-pricing]. |
| Limits on Free | **Digest duration max 1 day** (Pro 7d, Team 30d), delay max 1 day, 5 workflows, in-app (Inbox) message retention 30 days [limits][novu-limits]. |
| Batching / digest | Native **digest step**. The digest window is plan-capped as above [digest][novu-digest]. A 1-day cap is enough for a daily digest but not a weekly one. |
| Broadcasts / audiences | **Topics**: trigger once with `to: {type: "Topic", topicKey}`. There are up to 100,000 subscribers per topic, and billing counts one run per subscriber [topics][novu-topics]. No plan restriction is documented. |
| Email | Delivered through a connected email provider. Novu bills on workflow runs, not per channel [pricing][novu-pricing]. |
| React drop-in | `@novu/react` 3.19.2 (published 2026-09-07) has `<Inbox />` with Bell, Notifications, InboxContent and Preferences sub-components [inbox][novu-inbox], [npm][npm-novu]. React 18+ and **React 19 supported**, peer `^18 \|\| ^19` [inbox][novu-inbox]. Styling uses `appearance.variables` plus `appearance.elements`, and **elements accept Tailwind class strings**, including class-returning functions. Render props `renderNotification`, `renderSubject` and others are available [styling][novu-styling], [items][novu-items]. **Score 5/5**: it is the best fit for Tailwind among the drop-ins. |
| React headless | `useNotifications`, `useCounts`, `usePreferences` and `useNovu` in `@novu/react`, plus the framework-free `@novu/js` [source][novu-hooks-src]. **Score 5/5** |
| Feed auth | Application identifier plus `subscriberId`. In production, enable **HMAC**: the server computes `HMAC-SHA256(secretKey, subscriberId)` and the client passes it as `subscriberHash`. No JWT option is documented [production][novu-prod]. |
| Self-host / OSS | Yes, see section 3. |
| Lock-in / export | The core is MIT-licensed (`LICENSE-MIT`, with an `enterprise/` folder under `LICENSE-ENTERPRISE`) [repo][novu-repo]. Because the same API runs in cloud and self-hosted, cloud to self-host is a possible exit path [self-host][novu-selfhost]. |
| EU residency | **On Free.** Point `apiUrl` at `https://eu.api.novu.co` and `socketUrl` at `wss://eu.socket.novu.co` [pricing][novu-pricing], [production][novu-prod]. |

## 3. Novu self-hosted (Community edition)

| Aspect | Finding |
| --- | --- |
| Cost | No license fee. You pay for the infrastructure. |
| Features | Bell, Inbox, Notifications, Preferences and **Digest are available**. Cloud-only features: Snooze, removing Novu branding (Inbox and email), email activity tracking, MFA and RBAC. Self-hosted allows one team member [comparison][novu-compare], [self-host][novu-selfhost]. |
| Infra | MongoDB, **Redis with AOF**, S3-compatible storage (10 GB minimum), and API, Worker, WS and Dashboard services. The single-VM minimum is about 4 vCPU and 8 GB RAM [self-host][novu-selfhost]. That is heavy next to Cadence's SQLite-only stack. |
| React / auth | Same SDK as the cloud version, pointed at your own API host [self-host][novu-selfhost]. |
| EU residency | Wherever you host it. |

## 4. Courier

| Aspect | Finding |
| --- | --- |
| Free tier | **Developer: 10,000 sends/month**, including **Journeys and broadcasts** [pricing][courier-pricing]. A "send" is one template to one recipient, so a multi-channel template counts as one send. Test environments count too. **Above 10k, every send is billed, including the first 10k**, at $0.005 per send [pricing][courier-pricing]. |
| Batching / digest | Journeys **batch node**: per recipient, it releases on max items (100 by default, up to 1,000), an inactivity window or a max-wait ceiling, and can be split by category key. There are also **topic digest schedules** and an add-to-digest node [journeys][courier-journeys], [daily digest guide][courier-digest]. No Free-tier cap is documented. |
| Broadcasts / audiences | Broadcasts are included on Developer [pricing][courier-pricing]. The Lists API fans out a single send to subscribed profiles [lists][courier-lists]. |
| Email | Sent through connected providers. |
| React drop-in | `@trycourier/courier-react` 9.3.5 (MIT, published 2026-11-09) ships `CourierInbox`, `CourierInboxPopupMenu`, `CourierToast` and `CourierPreferences`. It needs **React 18+** (a separate `-17` package exists), and its peer `>=18` admits 19 [React SDK][courier-react], [npm][npm-courier]. Theming uses a JSON theme object (e.g. `inbox.header.backgroundColor`) and light/dark themes. The UI is built as **web components with shadow DOM** [web components][courier-wc], so Tailwind classes won't reach inside. **Score 3/5** |
| React headless | `useCourier()` returns `auth`, `inbox`, `toast` and `preferences`, and there are custom list-item renderers [React SDK][courier-react]. **Score 4/5** |
| Feed auth | The backend mints a **JWT** with the Courier API key via `/auth/issue-token`, with scopes such as `inbox:read:messages`. The client calls `auth.signIn({ userId, jwt })` [React SDK][courier-react], [auth][courier-auth]. |
| Self-host / OSS | No self-host option. The SDKs are MIT [repo][courier-repo]. |
| Lock-in / export | Templates live in Courier's designer and Journeys. I did not verify a templates-as-code export path. |
| EU residency | EU datacenter (eu-west-1) "available on every paid plan: Business and Enterprise", **so not on Free** [datacenters][courier-dc], [security][courier-sec]. |

## 5. MagicBell

| Aspect | Finding |
| --- | --- |
| Free tier | **Builder: 1,000 deliveries/month**, counted **per channel**: one notification to 100 users with in-app plus email for 50 of them is 150 deliveries. The next tier is Startup at $249/mo for 50k [pricing][magicbell-pricing]. |
| Batching / digest | **No digest feature is documented.** The API "batching" refers to bulk API operations [API][magicbell-api]. |
| Broadcasts / audiences | The **Broadcast** primitive takes recipients as users or topics, from 1 to 1,000 per broadcast. The dashboard also has broadcasts [broadcast][magicbell-broadcast]. |
| React drop-in | `@magicbell/react` 1.14.0 (published 2026-07-29) has `ContextProvider`, `FloatingInbox`, `Inbox` and `InlineNotification`. Styling uses CSS variables (`--magicbell-*`) and `.magicbell--*` classes [React][magicbell-react]. No React peer range is declared [npm][npm-magicbell]. **Score 3/5** |
| React headless | `@magicbell/react-headless` 5.5.1 (peer React >= 18.3.1, last published 2026-03-20) [npm][npm-magicbell-headless]. Hooks are not documented on the current React page. **Score 3/5** |
| Feed auth | The backend creates a **User JWT, HS256-signed with the project Secret Key**, with claims `user_email` and/or `user_external_id` plus `api_key`. It is passed as the `token` prop [user auth][magicbell-auth]. |
| Self-host / OSS | No. The license is "SEE LICENSE IN LICENSE" [npm][npm-magicbell]. |
| EU residency | **Not documented.** The only "eu" settings are for email-provider regions [Mailgun][magicbell-mailgun]. |

## 6. SuprSend

| Aspect | Finding |
| --- | --- |
| Free tier | **10,000 notifications/month**, unlimited channels, **in-app feed (with SuprSend branding)**, **Lists and broadcasts**, and unlimited team members [pricing][suprsend-pricing]. |
| Paywalled | **Batching and digest are Business only ($250/mo)**. Branding removal is on Essentials ($100/mo). Self-host/BYOC and multi-region data centers are Enterprise only [pricing][suprsend-pricing]. |
| Email | **Bring your own provider**, e.g. SendGrid, paid directly [pricing][suprsend-pricing]. |
| React drop-in | `@suprsend/react` 1.5.0 (MIT, published 2026-10-22). The older `@suprsend/react-inbox` is **deprecated** [inbox][suprsend-inbox], [npm][npm-suprsend]. `Inbox` popover plus full-screen and side-sheet feeds. Styling uses a `theme` object, `themeType` light/dark, and replaceable bell, badge and notification-card components [feed][suprsend-feed]. Peer `react >= 16.8` [npm][npm-suprsend]. **Score 3/5** |
| React headless | `@suprsend/react-core` 2.4.0 [npm][npm-suprsend-core], with headless docs [inbox][suprsend-inbox]. **Score 4/5** |
| Feed auth | Public key plus `distinct_id`. In production, use **Enhanced Security Mode**: an **ES256 JWT** that the backend signs with a dashboard signing key (`entity_type: "subscriber"`, `entity_id` = distinct_id) [client auth][suprsend-auth]. |
| Self-host / OSS | Enterprise only (self-hosted or BYOC) [pricing][suprsend-pricing]. |
| EU residency | Docs mention per-workspace data regions, but "Custom Data Center Region" is blank on Free and Business. Multi-region appears under Enterprise only [pricing][suprsend-pricing], [auth][suprsend-auth]. |

## 7. Engagespot (risk: appears dormant)

| Aspect | Finding |
| --- | --- |
| Status | On 2026-11-15, `engagespot.co/`, `engagespot.co/pricing` and `engagespot.com/` all returned **HTTP 404**. `docs.engagespot.co` returned a page containing only its title. The npm packages `@engagespot/react-component`, `react-hooks` and `core` were all last published **2025-05-09** (v3.0.1) [npm][npm-engagespot]. |
| Free tier | **Could not be verified from a primary source.** Aggregators report a free "Launch" plan with 10k events/month and US/EU regions, but these are secondary sources and unconfirmed [toolradar][es-toolradar], [freetier][es-freetier]. |
| React | Drop-in `@engagespot/react-component` and hooks `@engagespot/react-hooks`, peer `react >=16.8.3` [npm][npm-engagespot]. Not scored: 18 months without a release and the vendor site is down. |
| Recommendation | Treat it as **not viable** until the vendor site is reachable again. |

## 8. Plain email API plus a hand-built in-app feed

The in-app bell and feed would live in Cadence's own SQLite (Drizzle) tables, with a React Router loader and action, using the existing session. Email would go through Resend or Postmark.

| Aspect | Resend | Postmark |
| --- | --- | --- |
| Free tier | **3,000 emails/mo, capped at 100/day**. 3 domains, 1,000 marketing contacts, broadcasts to contacts, 30-day retention [pricing][resend-pricing] | **100 emails/mo**, no overage. Meant for testing [pricing][postmark-pricing] |
| Broadcasts | Broadcasts API to **segments** of contacts, with topics and unsubscribe-list management [broadcasts][resend-broadcasts] | Separate broadcast message streams [pricing][postmark-pricing] |
| EU | Can send from eu-west-1, but "all account data … is stored in the United States" [regions][resend-regions] | No EU hosting is described [pricing][postmark-pricing] |
| SDK | `resend` 6.38.1, MIT [npm][npm-resend] | `postmark` 5.1.1, MIT [npm][npm-postmark] |

- **In-app feed:** we would build it ourselves. That means a `notification` table, unread counts, a Radix Popover bell with a lucide `Bell` icon, and polling or SSE for live updates. **Drop-in 0/5 and headless 5/5**, because we own everything.
- **Auth:** the existing Cadence session. No tokens or HMAC are needed.
- **Digest and batching:** we would build them ourselves (a scheduled job that groups unsent rows per Recipient).
- **Lock-in:** none. All data is in our own DB.
- **Cost:** the feed costs only build and maintenance time. Email is capped by the Resend free tier (100/day).

---

## Comparison table

| Provider | Free quota / mo | In-app feed on Free | Digest on Free | Broadcasts on Free | EU on Free | Drop-in | Headless | React 19 | Self-host / OSS | Feed auth |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **Knock** | 10k messages | Yes (branded) | Yes (batch step, 10-item render cap) | Yes | **No** (planned end of 2026) | 4 | 5 | Yes (peer ^19) | No (SDK MIT) | RS256 JWT (enhanced security) |
| **Novu Cloud** | 10k workflow runs, 5 workflows | Yes (branded) | Yes, **window up to 1 day** | Yes (Topics) | **Yes** | 5 | 5 | Yes (peer ^19) | Yes (MIT core) | HMAC `subscriberHash` |
| **Novu self-hosted** | Unlimited (your infra) | Yes (branded) | Yes | Yes | Your choice | 5 | 5 | Yes | **Yes** | HMAC |
| **Courier** | 10k sends (multi-channel = 1) | Yes | Yes (batch node, digests) | Yes | No (paid plans) | 3 (shadow DOM) | 4 | Yes (peer >=18) | No (SDK MIT) | Server-minted JWT |
| **MagicBell** | **1k deliveries, per channel** | Yes | **No digest documented** | Yes (max 1k recipients each) | Not documented | 3 | 3 | Not declared | No | HS256 JWT |
| **SuprSend** | 10k notifications | Yes (branded) | **No (Business $250)** | Yes | No (Enterprise) | 3 | 4 | Not declared (>=16.8) | Enterprise only | ES256 JWT |
| **Engagespot** | Unverified | Unverified | Unverified | Unverified | Unverified | n/a | n/a | Not declared, stale | No | n/a |
| **Resend + own feed** | 3k email (100/day) | Build it | Build it | Resend Broadcasts or own fan-out | No (US data storage) | 0 | 5 | n/a | Fully ours | Cadence session |

Notes:

- Each vendor counts its free quota differently: messages, runs, sends, per-channel deliveries or notifications. Courier's multi-channel-counts-once rule and Novu's per-run billing are the most favourable for "bell + email per event". The exact counting unit for Knock and SuprSend was not confirmed beyond the pricing-page labels.
- The orchestrators (Knock, Novu, Courier, SuprSend) still need a downstream email provider, so Resend's free cap (100/day) can become the real email ceiling either way.

## Shortlist (input to the decision ticket, not the decision)

1. **Novu Cloud.** It is the only vendor whose free tier includes EU residency, a digest step and broadcasts (Topics). Its drop-in Inbox takes Tailwind classes directly and supports React 19, and the MIT self-hosted edition is an exit path. Watch: the 1-day digest window cap (fine for daily digests, not weekly), the 5-workflow cap, and HMAC-only feed auth.
2. **Knock.** It has the most generous free feature set (unlimited broadcasts, feeds and batching), strong headless hooks, React 19 support, and templates-as-code via its CLI. Watch: no EU residency yet (promised for end of 2026), branding on Free, and a 10-item digest render limit.
3. **Courier.** It has a generous send-counting model (multi-channel counts once) plus batch and digest support and broadcasts on Free. Watch: shadow-DOM components fit Tailwind poorly, EU is paid-only, and there is a billing cliff above 10k where all sends become billable.
4. **Hand-built feed plus Resend** as a baseline. It has no lock-in and is native to the Radix and Tailwind stack, but we would build digests and broadcasts ourselves, and Resend stores account data in the US.

**Excluded:**

- **MagicBell:** its 1k per-channel deliveries are too small, and it documents no digest.
- **SuprSend:** digest is paywalled at $250/mo.
- **Engagespot:** the vendor site is down and its SDK has not been updated since May 2025.
- **Postmark:** a 100/mo free tier is far too small to be the email leg.

## Open questions for the decision ticket

- What is the expected v1 volume (Recipients × events/month), so it can be compared against 10k runs, messages or sends?
- Is EU data residency a hard requirement (it would rule out Knock until it ships) or a tiebreaker?
- Is a weekly digest needed in v1? Novu Free caps the digest window at 1 day.

## Sources

[knock-pricing]: https://knock.app/pricing
[knock-plans]: https://docs.knock.app/manage-your-account/knock-plans
[knock-batch]: https://docs.knock.app/designing-workflows/batch-function
[knock-cli]: https://docs.knock.app/cli
[knock-sparkpost]: https://docs.knock.app/integrations/email/sparkpost
[knock-react]: https://docs.knock.app/in-app-ui/react/overview
[knock-feed]: https://docs.knock.app/in-app-ui/react/feed
[knock-pkg]: https://github.com/knocklabs/javascript/blob/main/packages/react/package.json
[knock-core-src]: https://github.com/knocklabs/javascript/blob/main/packages/react-core/src/index.ts
[knock-auth]: https://docs.knock.app/in-app-ui/security-and-authentication
[knock-repo]: https://github.com/knocklabs/javascript
[knock-api]: https://docs.knock.app/api-reference/overview
[knock-privacy]: https://knock.app/legal/privacy-policy
[npm-knock]: https://www.npmjs.com/package/@knocklabs/react
[novu-pricing]: https://novu.co/pricing
[novu-limits]: https://docs.novu.co/platform/developer/limits
[novu-digest]: https://docs.novu.co/platform/workflow/add-and-configure-steps/configure-action-steps/digest
[novu-topics]: https://docs.novu.co/platform/concepts/topics
[novu-inbox]: https://docs.novu.co/platform/inbox/react/components/inbox
[novu-styling]: https://docs.novu.co/platform/inbox/configuration/styling
[novu-items]: https://docs.novu.co/platform/inbox/advanced-customization/customize-notification-items
[novu-hooks-src]: https://github.com/novuhq/novu/blob/next/packages/react/src/hooks/index.ts
[novu-prod]: https://docs.novu.co/platform/inbox/prepare-for-production
[novu-repo]: https://github.com/novuhq/novu
[novu-selfhost]: https://docs.novu.co/community/self-hosting-novu/overview
[novu-compare]: https://docs.novu.co/community/self-hosted-and-novu-cloud
[npm-novu]: https://www.npmjs.com/package/@novu/react
[courier-pricing]: https://www.courier.com/pricing
[courier-journeys]: https://www.courier.com/docs/platform/automations/
[courier-digest]: https://www.courier.com/docs/guides/send-a-daily-digest
[courier-lists]: https://www.courier.com/blog/introducing-the-lists-api
[courier-react]: https://www.courier.com/docs/sdk-libraries/courier-react-web
[courier-wc]: https://www.courier.com/docs/sdk-libraries/courier-ui-inbox-web
[courier-auth]: https://www.courier.com/docs/in-app/authenticate-users
[courier-dc]: https://www.courier.com/docs/workspaces/datacenters
[courier-sec]: https://www.courier.com/docs/workspaces/security
[courier-repo]: https://github.com/trycourier/courier-web
[npm-courier]: https://www.npmjs.com/package/@trycourier/courier-react
[magicbell-pricing]: https://www.magicbell.com/pricing
[magicbell-api]: https://www.magicbell.com/docs/api
[magicbell-broadcast]: https://www.magicbell.com/docs/primitive/broadcast
[magicbell-react]: https://www.magicbell.com/docs/libraries/magicbell-react
[magicbell-auth]: https://www.magicbell.com/docs/api/authentication/user
[magicbell-mailgun]: https://www.magicbell.com/docs/email/mailgun
[npm-magicbell]: https://www.npmjs.com/package/@magicbell/react
[npm-magicbell-headless]: https://www.npmjs.com/package/@magicbell/react-headless
[suprsend-pricing]: https://www.suprsend.com/pricing
[suprsend-inbox]: https://docs.suprsend.com/docs/inbox-react
[suprsend-feed]: https://docs.suprsend.com/docs/react-in-app-feed
[suprsend-auth]: https://docs.suprsend.com/docs/client-authentication
[npm-suprsend]: https://www.npmjs.com/package/@suprsend/react
[npm-suprsend-core]: https://www.npmjs.com/package/@suprsend/react-core
[npm-engagespot]: https://www.npmjs.com/package/@engagespot/react-component
[es-toolradar]: https://toolradar.com/tools/engagespot/pricing
[es-freetier]: https://freetier.co/directory/products/engagespot
[resend-pricing]: https://resend.com/pricing
[resend-broadcasts]: https://resend.com/docs/dashboard/broadcasts/introduction
[resend-regions]: https://resend.com/docs/dashboard/domains/regions
[npm-resend]: https://www.npmjs.com/package/resend
[postmark-pricing]: https://postmarkapp.com/pricing
[npm-postmark]: https://www.npmjs.com/package/postmark

npm versions, publish dates and peer dependencies were read from `https://registry.npmjs.org/<package>` on 2026-11-15. GitHub license and activity data came from the GitHub API on the same date.
