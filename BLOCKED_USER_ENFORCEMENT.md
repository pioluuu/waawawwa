# Blocked-user enforcement report

## 1. Root cause

The existing block relation was checked for friend-request sending and social
search/suggestions, but was not an authorization boundary for DM, profile, post,
comment, or private socket activity. Pending request acceptance could create a
friendship despite a block. The REST single-post endpoint also bypassed its
service's visibility authorization. A two-way saved block could break the old
friend-send check because it assumed at most one matching row.

## 2. Existing architecture

Reused `BlockedUser`/`blocked_users`, the existing friends block/unblock/list APIs,
FriendService, ChatService, PostService, CommentService, ProfileService and the
existing socket manager/SocketProvider lifecycle. Block ownership remains
directional for Settings and unblock; authorization is mutual.

## 3. Files changed

**Authorization and contracts**

- [backend/app/services/block_service.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/services/block_service.py)
- [backend/app/models/friend.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/models/friend.py)
- [backend/app/schemas/post.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/schemas/post.py)

**Social services**

- [backend/app/services/chat_service.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/services/chat_service.py)
- [backend/app/services/friend_service.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/services/friend_service.py)
- [backend/app/services/profile_service.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/services/profile_service.py)
- [backend/app/services/post_service.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/services/post_service.py)
- [backend/app/services/comment_service.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/services/comment_service.py)
- [backend/app/services/stream_comment_service.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/services/stream_comment_service.py)

**REST and realtime**

- [backend/app/api/v1/endpoints/posts.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/api/v1/endpoints/posts.py)
- [backend/app/api/v1/endpoints/users.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/api/v1/endpoints/users.py)
- [backend/app/websocket/manager.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/websocket/manager.py)
- [backend/app/websocket/chat_handler.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/websocket/chat_handler.py)

**Client state and UX**

- [frontend/src/lib/socialRelationships.ts](C:/Users/Administrator/Desktop/CCS_HUB/frontend/src/lib/socialRelationships.ts)
- [frontend/src/app/providers/SocketProvider.tsx](C:/Users/Administrator/Desktop/CCS_HUB/frontend/src/app/providers/SocketProvider.tsx)
- [frontend/src/features/friends/hooks/useFriends.ts](C:/Users/Administrator/Desktop/CCS_HUB/frontend/src/features/friends/hooks/useFriends.ts)
- [frontend/src/features/settings/PhaseTwoSettings.tsx](C:/Users/Administrator/Desktop/CCS_HUB/frontend/src/features/settings/PhaseTwoSettings.tsx)
- [frontend/src/features/posts/hooks/useFeed.ts](C:/Users/Administrator/Desktop/CCS_HUB/frontend/src/features/posts/hooks/useFeed.ts)
- [frontend/src/features/posts/components/PostCard.tsx](C:/Users/Administrator/Desktop/CCS_HUB/frontend/src/features/posts/components/PostCard.tsx)
- [frontend/src/features/posts/components/PostDetailModal.tsx](C:/Users/Administrator/Desktop/CCS_HUB/frontend/src/features/posts/components/PostDetailModal.tsx)
- [frontend/src/features/profile/pages/ProfilePage.tsx](C:/Users/Administrator/Desktop/CCS_HUB/frontend/src/features/profile/pages/ProfilePage.tsx)
- [frontend/src/features/profile/components/ProfileHoverCard.tsx](C:/Users/Administrator/Desktop/CCS_HUB/frontend/src/features/profile/components/ProfileHoverCard.tsx)
- [frontend/src/features/users/hooks/useUsers.ts](C:/Users/Administrator/Desktop/CCS_HUB/frontend/src/features/users/hooks/useUsers.ts)
- [frontend/src/services/api/post.service.ts](C:/Users/Administrator/Desktop/CCS_HUB/frontend/src/services/api/post.service.ts)
- [frontend/src/types/post.types.ts](C:/Users/Administrator/Desktop/CCS_HUB/frontend/src/types/post.types.ts)

**Tests and rollout**

- [backend/app/tests/test_social_blocks.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/tests/test_social_blocks.py)
- [backend/app/tests/test_global_chat.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/tests/test_global_chat.py)
- [backend/app/tests/test_content_filter.py](C:/Users/Administrator/Desktop/CCS_HUB/backend/app/tests/test_content_filter.py)
- [backend/migrations/20261005_social_blocks.sql](C:/Users/Administrator/Desktop/CCS_HUB/backend/migrations/20261005_social_blocks.sql)
- [PROJECT_CONTEXT.md](C:/Users/Administrator/Desktop/CCS_HUB/PROJECT_CONTEXT.md)
- [BLOCKED_USER_ENFORCEMENT.md](C:/Users/Administrator/Desktop/CCS_HUB/BLOCKED_USER_ENFORCEMENT.md)

The pre-existing Register, Topbar, StudentDashboard, ProfessorDashboard and
HeroSection edits were preserved and are not part of this fix.

## 4. Shared backend authorization

`ensure_users_can_interact` returns generic HTTP 403 without disclosing the
block owner. `is_blocked_between` uses EXISTS, while SQL blocked-ID unions serve
feed, discovery and event delivery. Stable canonical UUID pair advisory locks
coordinate social writes with block/unblock across PostgreSQL transactions and
workers. The per-request post author lookup avoids additional per-post block
queries.

## 5. Direct messages

Both directions reject new/existing DM sends, replies and media messages. History
remains readable, but loading history does not mark blocked senders read. Direct
reactions/read receipts are rejected. Explicit group invitations also reject
blocked invitees. Delivery rechecks authorization after save to suppress a DM
broadcast/notification if a block committed before delivery.

## 6. Friends and unblock

Request send and acceptance check the same mutual rule. Block, friendship removal
and pending-request cancellation commit together. Acceptance and friendship
creation now commit together under the pair lock. Repeating a block repairs
legacy hidden friendships. Unblock removes only that owner's block; it restores
neither friendships nor cancelled requests. A reverse block still restricts both.
Legacy blocked relations are excluded from friendship/pending-list presence data.

## 7. Profiles

Normal profile and profile post/share APIs reject both directions. The profile UI
shows ?Profile unavailable? with Back, clears old content and prevents pre-change
responses from repopulating it. Shared academic member/attendance directories and
separate admin moderation authorization remain available.

## 8. Posts, comments and reactions

Stored post authors authorize like, reaction and share actions. Comment creation,
editing, replies, likes and reactions check stored post/comment/parent authors.
Single-post REST access now uses the existing visibility guard; comment history
also validates post access. The additive `can_interact` response hint disables
social post controls while allowing required announcement content to be read.
Removal of one's own historic content/reporting remains possible.

## 9. Feed and discovery

SQL filters exclude blocked social authors and sharers before feed retrieval.
Original posts reached via another user's share are also filtered. Linked
announcement posts retain their existing audience rules. Search and suggestions
reuse the shared SQL exclusion, and local search results clear on relationship
changes.

## 10. Socket.IO and presence

Authenticated connection identity remains authoritative. Existing send/reaction
handlers use ChatService checks; DM typing/read and room-entry handlers reuse
conversation authorization. Post-room admission and event delivery both filter
blocks, including subscriptions made before blocking. The unused unsafe decorator
stubs now export the real manager handlers instead of registering alternatives.
No second socket connection was introduced.

## 11. Client caches and notifications

The existing provider consumes a neutral account-only relationship-change signal.
The shared client refresh updates blocked lists, friendship/request/suggestion and
conversation queries; resets local feed/profile/search/hover state; clears typing;
and closes affected DM/post views. Stale feed/profile responses cannot reinsert
pre-change content. Inappropriate post controls are disabled/hidden, and restricted
profile/post reads render safe unavailable states. Backend rejection happens
before content writes and notifications; friend notifications also recheck the
relation. No blocked-attempt notification or whole-site reload was added.

## 12. Migration and rollout

Apply `backend/migrations/20261005_social_blocks.sql` before deploying the backend.
It ensures the existing owner/target unique index, adds the reverse lookup index,
removes legacy blocked friendships and cancels their pending requests. It does
not change table columns or remove messages, reports, moderation or academic
records. Duplicate legacy block rows cause the transaction to fail safely;
reconcile duplicates and retry rather than deleting relationships automatically.

Coordinate restart of all backend workers so no older handler remains active,
then deploy the frontend. Testing the migration does not apply it to the
application database. The application database was not migrated by this task.

## 13. Verification and limits

- Full backend run: 76 tests passed, no skips, with social-block, chat, anonymous
  identity, socket-session and content-filter suites. PostgreSQL cases used a
  disposable database and isolated schemas.
- The 10-case social-block matrix was rerun after final authorization changes;
  selected moderation/socket and REST cases were also rerun after their changes.
- Tests cover mutual DM/media/reply rejection, direct REST access, authenticated
  socket spoof attempts, typing/read/reactions, stale post-room delivery, preserved
  history, cleanup, unblock/reverse-block behavior, feed/search, normal allowed
  interactions, migration repeatability, uniqueness and concurrent pair locks.
- Frontend TypeScript/Vite production build, Python compilation and diff whitespace
  checks passed. The existing Vite large-bundle warning remains.
- API tests use an in-process FastAPI fixture with real PostgreSQL. Socket tests
  invoke actual handlers with fixture authentication and mocked network delivery.
  These are not live browser, network WebSocket or WebRTC end-to-end verification.
- Existing Python datetime deprecations and Redis fixture resource/fallback warnings
  appeared; they did not fail the tests. No live Redis delivery claim is made.
- The historical frontend test script paths listed elsewhere in project context
  are absent from this checkout; those Node fixture commands could not run.

## 14. Intentional academic/shared visibility

Global, section and existing shared group history/ordinary room messages remain
visible. Section/subject membership, schedules, attendance, required announcements,
public/academic livestream access, Meethub classroom participation and organizer
controls retain their academic rules. Directed nonanonymous chat/stream replies
and reactions check blocks; shared anonymous interactions preserve anonymity so
block behavior cannot be used to identify the real sender. Official announcement
APIs and private bookmarking remain available under existing academic policies.
