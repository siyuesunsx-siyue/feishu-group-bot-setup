---
name: feishu-group-bot-setup
description: Create or configure a Feishu bot for a specified group, define its behavior, grant the minimum permissions, configure message delivery, publish it, add it to the group, and verify it. Use when an agent is asked to build, customize, install, or troubleshoot a Feishu group bot.
---

# Feishu Group Bot Setup

Use this skill to take a user from the bot requirement to a verified bot in a Feishu group. Work with the user's existing Feishu account and app where possible. Keep the bot's purpose, audience, permissions, and delivery behavior specific to the request.

## 1. Establish the requested bot

Before making cloud changes, identify:

- Bot name, purpose, language, tone, and example requests/replies.
- Target group and the user who can approve app changes or add bots there.
- Whether the bot must only post messages, or also receive group messages and reply.
- Which people or groups it may respond to, and whether it must be explicitly @mentioned.
- Where its code will run and which event transport that runtime supports.

Do not infer a group, recipient, message-reading scope, or permission from a display name. Ask for the exact target when it cannot be unambiguously identified. Obtain explicit user approval before creating or changing an app, granting scopes, publishing a version, subscribing to events, adding the bot to a group, or sending a live test message. Explain the exact change and scope when requesting approval. Use the smallest scope that can satisfy the approved behavior.

## 2. Choose the bot type

Use a **custom group webhook bot** only when the requirement is one-way posting to a group. It does not provide a normal application identity that listens to arbitrary group messages and replies. Keep the webhook URL secret; anyone with it may be able to post to that group.

Use a **custom/self-built app with bot capability** when the bot must be @mentioned, receive events, respond, use Feishu APIs, or have a stable app identity. This is the normal choice for conversational group bots. A user-created app may require an administrator to grant permissions, approve scopes, and publish an app version before it is usable.

Official references:

- [Bot overview](https://open.feishu.cn/document/client-docs/bot-v3/bot-overview)
- [Create a custom bot](https://open.feishu.cn/document/client-docs/bot-v3/add-custom-bot)
- [Event subscription configuration](https://open.feishu.cn/document/ukTMukTMukTM/uYDNxYjL2QTM24iN0EjN/event-subscription-configure-)
- [IM API reference](https://open.feishu.cn/document/uAjLw4CM/ukTMukTMukTM/reference/im-v1)

Check the current official console and documentation for exact permission names and approval requirements; they can change. Do not request broad group-history access if @-only events meet the requirement. If the request truly requires all group messages, explain that this is a more sensitive scope, name the required permission and affected group scope, and wait for explicit approval before enabling it.

## 3. Create and configure the app

1. In the Feishu Open Platform, create a **custom/self-built enterprise app** in the correct organization. Confirm the organization and app owner with the user if ambiguous.
2. Set the requested app name, icon, description, and bot behavior. Keep the description user-facing and limited to what the bot actually does.
3. Enable the bot capability.
4. Add only the API permissions required for the approved behavior. Typical interactive group bots need permission to receive the selected message events and send/reply to messages. Add contact, file, or other scopes only when the bot behavior requires them.
5. Subscribe to the required message event (commonly `im.message.receive_v1`). Select a delivery method supported by the runtime:
   - **Long connection/WebSocket**: use when the bot process can maintain an outbound connection and the selected SDK supports it.
   - **Webhook callback**: use when the service has a reachable HTTPS callback. Verify the endpoint, request signature/token checks, and event challenge handling before enabling production traffic.
6. Make event handling idempotent. Ignore the bot's own messages; for group responses, normally require an actual @ mention and restrict handling to the approved group(s). Prevent loops with event de-duplication and sender checks.
7. Configure app availability to the intended users only. Avoid expanding tenant or member availability as a shortcut to test.
8. Save and publish/submit the app version. Complete administrator review or permission approval as required. Do not describe the app as live until the console shows it is available and the runtime is connected.

## 4. Protect credentials and runtime

- Keep App Secret, webhook URL, verification token, and encryption key in a secret manager, environment variables, or a local ignored config file. Never place them in source, skill files, screenshots, chat messages, logs, or Git history.
- Use placeholders in examples. Never print a secret to confirm it was saved; report only whether it is present and whether authentication succeeded.
- Prefer an existing approved runtime. Document how to start, stop, inspect health, and restart it. Run only one production listener unless the implementation has shared durable de-duplication.
- If credentials are exposed, advise rotating them in the Feishu console and updating the runtime securely.

## 5. Add the bot to the group

1. Confirm the exact target group with the user; use its title plus a second identifying detail if multiple groups have similar names.
2. Ensure the app is available to the relevant group members and the user has permission to add bots.
3. Add the app bot using the group's add-bot/app flow. If group policy blocks it, report the policy or administrator action needed; do not work around it by broadening access.
4. Verify the bot appears in the intended group and nowhere else unless separately requested.
5. For a live test, first get approval for the exact test text and target group. Prefer a short harmless @ message. Verify both the inbound event and the outbound message, and distinguish API acceptance from a notification displayed on a person's device.

## 6. Troubleshoot in order

When the bot does not respond, inspect one layer at a time:

1. App is enabled, published, and available to the tester.
2. Bot capability is on and the bot is a member of the exact group.
3. Required event subscription is enabled and matches the chosen transport.
4. The listener/callback is running, authenticated, and receiving events.
5. The event belongs to the approved group, includes the expected @ mention, and is not a duplicate or the bot's own message.
6. The handler accepted the event and routed it to the correct behavior.
7. The send API returned success; then separately check the message in the group and, if relevant, device notification settings.

Use redacted logs that show stage, timestamp, and error category without message bodies, user IDs, app secrets, or webhook URLs. Do not repeatedly resend when the previous send result is unknown; first query the group or delivery record to avoid duplicates.

## 7. Handoff

Summarize the bot's purpose and approved group scope, app/version status, permissions/events, runtime and start/health instructions, verification evidence, and remaining blockers. Do not include credential values, unrelated project details, private conversations, or family-specific data. State clearly whether the bot was only configured, published, added to the group, connected to its runtime, or actually verified end to end.
