# In-app notification vendors for Cadence

Date checked: 2026-11-15. Pricing and package versions change often. Check again before you decide.

Sources: official docs, official pricing pages, official GitHub repos, and the npm registry (`npm view`, run on 2026-11-15). "Unverified" means that no primary source confirmed the claim.

## Summary

Cadence needs an in-app bell and feed with per-type preferences, a Node server SDK, and a later path to email. Three vendors fit all the hard criteria at low cost. Novu is the best fit. It has the lowest paid step, it accepts Tailwind classes on its prebuilt Inbox, and it ships a prebuilt preferences UI. Knock has the strongest feed model and headless hooks, but its paid tier costs $250/month and it has no prebuilt preferences component. Courier has pay-as-you-go pricing and a headless hook, but its docs do not say that Tailwind works on its prebuilt components.

SuprSend and MagicBell do not fit as well. SuprSend puts user preferences in its $250/month tier. MagicBell gives only 1,000 free deliveries and its next tier costs $249/month.

### Ranked shortlist

1. **Novu**: Tailwind classes go directly on the prebuilt `<Inbox />`, which also has a Preferences view. Pro costs $30/month. The free tier allows only 5 workflows.
2. **Knock**: Best headless hooks and the richest read-state model (seen, read, interacted, archived). Signed JWT auth. The paid tier is a large step, to $250/month.
3. **Courier**: `useCourier()` headless hook, a prebuilt `CourierPreferences` component, and pay-as-you-go at $0.005/send after the free tier.
4. SuprSend: Good SDKs, but preferences are on the $250/month Business plan.
5. MagicBell: Only 1,000 free deliveries, and the next tier costs $249/month.

## Comparison table

| | Novu | Knock | Courier | SuprSend | MagicBell |
|---|---|---|---|---|---|
| Free tier | 10k workflow runs/mo, 5 workflows, 24h activity retention ([pricing](https://novu.co/pricing)) | 10k messages/mo, Knock branding ([pricing](https://knock.app/pricing)) | 10k sends/mo ([billing](https://www.courier.com/docs/workspaces/billing)) | 10k notifications/mo, feed has SuprSend branding ([pricing](https://www.suprsend.com/pricing)) | 1,000 deliveries/mo ([pricing](https://www.magicbell.com/pricing)) |
| First paid tier | Pro from $30/mo ([pricing](https://novu.co/pricing)) | Starter $250/mo + usage ([pricing](https://knock.app/pricing)) | Business $0.005/send from the first send ([pricing](https://www.courier.com/pricing)) | Essentials $100/mo, no preferences ([pricing](https://www.suprsend.com/pricing)) | Startup $249/mo ([pricing](https://www.magicbell.com/pricing)) |
| React UI | `@novu/react` Inbox, Bell, Notifications, Preferences | `@knocklabs/react` bell, popover, feed | `@trycourier/courier-react` Inbox, popup, toast, preferences | `@suprsend/react` Inbox popover | `@magicbell/react` FloatingInbox, Inbox, UserPreferences |
| Headless | Yes, hooks | Yes, hooks | Yes, `useCourier()` | Yes, `@suprsend/react-core` hooks | `@magicbell/react-headless` (details unverified) |
| Tailwind | Yes, documented for `appearance.elements` | CSS vars or replace CSS. Tailwind not mentioned | Theme object. Tailwind not mentioned | Theme object. Tailwind not mentioned | CSS classes and vars. Tailwind not mentioned |
| Node SDK | `@novu/api` | `@knocklabs/node` | `@trycourier/courier` | `@suprsend/node-sdk` | `magicbell-js` |
| Per-type prefs | Workflow x channel, prebuilt UI | Workflow / category / channel, no prebuilt React UI | Topics, prebuilt component | Categories, Business plan only | Categories, prebuilt component |
| Real-time | WebSocket | Socket | WebSocket | Socket (likely Socket.IO) | WebSocket |
| Read states | read, seen, archived, snoozed | seen, read, interacted, archived, link clicked | read, opened, archived, clicked | seen, read, interacted, archived | read, seen, archived |
| Feed auth | HMAC-SHA256 hash | RS256 JWT | Scoped JWT from API | ES256 JWT | HS256 JWT |
| Email later | Yes, same workflow | Yes, same workflow | Yes, same template/routing | Yes | Yes |

The per-vendor sections give a source link for each cell.

## Novu

- **Pricing.** Free: 10,000 workflow runs/month, up to 5 workflows, 2 environments, 3 team members, all channels including in-app, and 24h activity feed retention. Pro starts at $30/month, includes 30,000+ runs, and charges $1.20 per 1,000 extra runs. ([pricing](https://novu.co/pricing)) Risk: each notification type is a workflow, so the free tier allows 5 types only.
- **React.** `@novu/react` has `<Inbox />`, `Bell`, `Notifications`, `InboxContent`, and `Preferences`, plus headless hooks ([inbox overview](https://docs.novu.co/platform/inbox/overview)). Peer dependency: `react ^18 || ^19` (npm, v3.19.2). `useNotifications` returns the list, pagination, and actions ([useNotifications](https://docs.novu.co/platform/sdks/react/hooks/use-notifications)).
- **Tailwind.** The `appearance` prop takes `variables` and `elements`. You "can pass Tailwind CSS classes to specific elements using the elements object" ([styling](https://docs.novu.co/platform/inbox/configuration/styling)).
- **Node SDK.** `@novu/api` (npm, v3.19.1). `@novu/node` is deprecated in favor of `@novu/api` (npm deprecation notice, linked to [server SDK docs](https://docs.novu.co/platform/sdks/server/typescript)).
- **Preferences.** There are three layers: workflow defaults, subscriber global preferences, and subscriber per-workflow channel preferences. The most specific layer wins. A "critical" workflow cannot be turned off. The prebuilt Inbox has a Preferences view ([preferences](https://docs.novu.co/platform/concepts/preferences)).
- **Real-time.** WebSocket subscription, on by default (`realtime` prop) ([useNotifications](https://docs.novu.co/platform/sdks/react/hooks/use-notifications)). There is a separate `socketUrl` for EU ([production](https://docs.novu.co/platform/inbox/prepare-for-production)).
- **Read state.** `read()`/`unread()`, `seen()` (no unseen), `archive()`/`unarchive()`, `snooze()`, `delete()`. Bulk actions: `readAll`, `seenAll`, `archiveAll`, `archiveAllRead` ([useNotifications](https://docs.novu.co/platform/sdks/react/hooks/use-notifications)).
- **Identity and auth.** The client sends `applicationIdentifier` + `subscriberId`. With HMAC on, the server computes `subscriberHash = HMAC-SHA256(NOVU_SECRET_KEY, subscriberId)` as hex. If the hash is missing, the Inbox does not load ([production](https://docs.novu.co/platform/inbox/prepare-for-production)). The hash does not expire (it is deterministic), so you cannot revoke it without rotating the key (inference from the docs).
- **Email path.** Add an email step to the same workflow. Preferences resolve per channel ([preferences](https://docs.novu.co/platform/concepts/preferences)).
- **Other.** The core repo is MIT. Enterprise code has a commercial license ([GitHub](https://github.com/novuhq/novu)). Production self-hosting: unverified.

## Knock

- **Pricing.** Free: 10,000 messages/month, unlimited profiles, workflows, and channels. Knock branding stays on. Starter: $250/month with 50,000 messages, then $0.005 each. Per-tenant preferences are on Enterprise only. ([pricing](https://knock.app/pricing))
- **React.** `@knocklabs/react` (npm, v0.15.1, peer `react ^17 || ^18 || ^19`) has `KnockProvider`, `KnockFeedProvider`, `NotificationIconButton`, `NotificationFeedPopover`, and `NotificationFeed` with `renderItem` ([feed docs](https://docs.knock.app/in-app-ui/react/feed)). Headless: `useKnockClient`, `useNotifications(client, channelId)`, and `useNotificationStore(feed)`, which returns `items` and `metadata.unread_count`/`unseen_count`. With headless hooks, you must mark items as seen yourself ([headless feed](https://docs.knock.app/in-app-ui/react/headless/feed)).
- **Tailwind.** Prebuilt components use `--rnf-` CSS variables, or you replace the `rnf-` CSS. Tailwind is not mentioned ([feed docs](https://docs.knock.app/in-app-ui/react/feed)). With Tailwind, use the headless hooks.
- **Node SDK.** `@knocklabs/node` (npm, v1.41.1).
- **Preferences.** A `PreferenceSet` has `workflows`, `categories`, `channel_types`, and `channels`. The default is opt-in. An environment default set is available. There is a hosted preference page, but no prebuilt React preference component ([preferences](https://docs.knock.app/preferences/overview)). You build the UI with `knockClient.user.getPreferences()`/`setPreferences()` ([React preferences](https://docs.knock.app/in-app-ui/react/preferences)).
- **Real-time.** Socket connection. Inactive tabs disconnect automatically. Cross-tab sync with BroadcastChannel is experimental ([feed docs](https://docs.knock.app/in-app-ui/react/feed)).
- **Read state.** `seen_at`, `read_at`, `interacted_at`, `link_clicked_at`, `archived_at`. Seen, interacted, and archived are managed for in-app feeds ([message statuses](https://docs.knock.app/send-notifications/message-statuses)).
- **Identity and auth.** By default, anyone with the feed ID can read the feed. In "enhanced security mode", the client must send a `userToken`: an RS256 JWT with `sub` = user ID, signed with `signUserToken` from `@knocklabs/node`. The default expiry is 1 hour. Refresh it with `onUserTokenExpiring` ([security](https://docs.knock.app/in-app-ui/security-and-authentication)).
- **Email path.** Add an email channel step to the workflow. Use `channel_types.email` preferences ([preferences](https://docs.knock.app/preferences/overview)).

## Courier

- **Pricing.** Developer: free, 10,000 sends/month. Business: $0.005/send, billed from the first send. ([pricing](https://www.courier.com/pricing)) One send = one template to one recipient. A multi-channel send counts once, and Inbox has no extra fee. Sends from test environments count. ([billing](https://www.courier.com/docs/workspaces/billing)) Inbox and preferences on the free plan: unverified (the pricing and billing pages do not say).
- **React.** `@trycourier/courier-react` (npm, v9.3.5, peer `react >=18`) has `CourierInbox`, `CourierInboxPopupMenu`, `CourierToast`, and `CourierPreferences`. Headless: `useCourier()` ([React SDK](https://www.courier.com/docs/sdk-libraries/courier-react-web)). `@trycourier/react-inbox` is deprecated (npm). In a custom inbox, call `registerFeeds`, `listenForUpdates`, and `load` in an effect. Actions: `readMessage`, `archiveMessage`, `readAllMessages`, and more ([custom inbox](https://www.courier.com/docs/in-app/build-a-custom-inbox)).
- **Tailwind.** Prebuilt components take `CourierInboxTheme` objects (`lightTheme`/`darkTheme`) and `renderListItem` and other render props ([v8 migration](https://www.courier.com/docs/sdk-libraries/courier-react-v8-migration-guide)). Tailwind on prebuilt parts is unverified. The headless hook returns plain data, so Tailwind works on your own JSX.
- **Node SDK.** `@trycourier/courier` (npm, v10.0.0).
- **Preferences.** Subscription topics have the states `OPTED_IN`, `OPTED_OUT`, and `REQUIRED`. A user can pick channels per topic when custom routing is on. There is a hosted page or an in-app preference center ([preferences](https://www.courier.com/docs/platform/preferences/preferences-overview)). The prebuilt component is `CourierPreferences` ([React SDK](https://www.courier.com/docs/sdk-libraries/courier-react-web)).
- **Real-time.** WebSocket. You must call `inbox.listenForUpdates()`, or the feed loads once only ([React SDK](https://www.courier.com/docs/sdk-libraries/courier-react-web)).
- **Read state.** `read`, `opened`, and `archived` timestamps, plus `clickMessage` ([custom inbox](https://www.courier.com/docs/in-app/build-a-custom-inbox)).
- **Identity and auth.** The server calls `client.auth.issueToken({ scope: "user_id:<id> inbox:read:messages inbox:write:events", expires_in })`. The client calls `signIn({ userId, jwt })`. Without `expires_in`, the token never expires. The SDK does not refresh tokens ([auth](https://www.courier.com/docs/platform/inbox/authentication)). This is one API call per token mint, not a local signature.
- **Email path.** One template routes to many channels, and Inbox and email count as one send ([billing](https://www.courier.com/docs/workspaces/billing)).

## SuprSend

- **Pricing.** Free: 10,000 notifications/month, feed has SuprSend branding. Essentials: $100/month. Business: $250/month, which adds "user preferences" and digests ([pricing](https://www.suprsend.com/pricing)). This fails the low-cost preferences criterion.
- **React.** `@suprsend/react` (npm, v1.5.0) has the `Inbox` popover with a bell, custom card components, and a `theme`/`themeType` prop ([popover feed](https://docs.suprsend.com/docs/react-in-app-feed)). Headless: `useFeedClient` and `useFeedData` from `@suprsend/react-core` ([headless](https://docs.suprsend.com/docs/react-sdk-headless-feed)). `@suprsend/react-inbox` is deprecated (npm). Tailwind is not mentioned.
- **Node SDK.** `@suprsend/node-sdk` (npm, v1.15.1).
- **Preferences.** Categories with On, Off, or Can't Unsubscribe defaults, global channel opt-outs, and per-category channel opt-outs. Hosted or embedded preference page ([preferences](https://docs.suprsend.com/docs/user-preferences)).
- **Real-time.** Socket (`initializeSocketConnection`). The docs say "socketio" ([JS feed](https://docs.suprsend.com/docs/js-inapp-feed)).
- **Read state.** `markAsSeen`, `markAsRead`/`markAsUnread`, `markAsInteracted`, `markAsArchived`, `markAllAsRead`, and `markBulkAsSeen` ([JS feed](https://docs.suprsend.com/docs/js-inapp-feed)).
- **Identity and auth.** Public key + an ES256 JWT with `entity_type: "subscriber"`, `entity_id`, `iat`, `exp`, signed on the server with `jsonwebtoken`. Refresh with the `refreshUserToken` callback ([client auth](https://docs.suprsend.com/docs/client-authentication)).

## MagicBell

- **Pricing.** Builder: free, 1,000 deliveries/month. Startup: $249/month with 50,000 deliveries. Deliveries count per channel, so in-app + email = 2 ([pricing](https://www.magicbell.com/pricing)).
- **React.** `@magicbell/react` has `FloatingInbox`, `Inbox`, and `UserPreferences` (per-category channel toggles). Styling uses CSS classes and `--magicbell-*` variables. Tailwind is not mentioned ([React docs](https://www.magicbell.com/docs/libraries/magicbell-react)). `@magicbell/react-headless` ("Hooks to build a notification inbox", npm v5.5.1) exists. Its current API and support status are unverified.
- **Node SDK.** `magicbell-js` (`magicbell-js/project-client`, `client.broadcasts.createBroadcast`) ([JS docs](https://www.magicbell.com/docs/libraries/magicbell-js)). `@magicbell/core` is deprecated (npm).
- **Real-time.** WebSocket ([JS docs](https://www.magicbell.com/docs/libraries/magicbell-js)).
- **Read state.** read, seen, and archived, with `read_at` and `seen_at` ([notification](https://www.magicbell.com/docs/primitive/notification)).
- **Identity and auth.** HS256 JWT with `user_email` and/or `user_external_id` + `api_key`, signed with the project secret ([user auth](https://www.magicbell.com/docs/api/authentication/user)).

## Others considered

- **Engagespot.** `@engagespot/node` was last published on 2024-11-06 and `@engagespot/react-component` on 2025-05-09 (npm). Release activity looks low, so we did not research it further.

## Open questions and risks

- **Identity mapping with DevUI user switching.** Cadence users have integer IDs (`DevUser.id: number` in `app/components/dev-ui.tsx`). A reseed can give an ID to a different person. Use a namespaced, stable recipient ID (for example `cadence-<env>-<userId>`), and use a separate vendor environment for each developer or for CI. Otherwise, feeds from old seeds can show for new users. When DevUI switches user, the client must get a new signed token or hash for the new user and reconnect the provider. For Knock, a new `authenticate` call restarts the socket ([security](https://docs.knock.app/in-app-ui/security-and-authentication)). For Courier, call `signIn` again ([auth](https://www.courier.com/docs/platform/inbox/authentication)). Key the provider on the user ID so that React remounts it.
- **Profile sync.** The display name must get to the vendor. Novu updates profile fields from the client only with a valid `subscriberHash` ([production](https://docs.novu.co/platform/inbox/prepare-for-production)). It is safer to identify or upsert users from the server.
- **Read-state semantics.** The vendors use different models. Novu and Knock separate "seen" (the badge count clears when the feed opens) from "read". Courier uses read/opened. Decide whether the bell badge counts unseen or unread. If Cadence also stores state in SQLite, decide which side is the source of truth. If you use only the vendor state, you cannot query it from Drizzle.
- **Testing without calling the vendor.** Put the vendor behind a small server module (for example `notify(userId, type, payload)`) and a client hook adapter. Tests then use an in-memory fake. Sends from test environments count toward billing on Courier ([billing](https://www.courier.com/docs/workspaces/billing)), and this is probably true for the others (unverified). Novu's core is MIT, so it could run locally for integration tests, but production self-hosting is unverified ([GitHub](https://github.com/novuhq/novu)).
- **Free-tier fit.** Novu's 5-workflow limit caps the number of notification types. Knock and SuprSend show their branding on the free tier. MagicBell's 1,000 deliveries is low.
- **Token lifetime.** Novu's HMAC hash does not expire. Courier tokens without `expires_in` never expire. Knock tokens expire after 1 hour by default. Match the token lifetime to the Cadence session.
- **Prebuilt UI and Tailwind.** Only Novu documents Tailwind classes on prebuilt parts. For Knock and Courier, plan to use the headless hooks with your own Tailwind components.
