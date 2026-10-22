# CCS HUB project context

Last source review: 2026-09-27.

This is the maintained replacement for the supplied "AI CODING AGENT PROJECT
CONTEXT" document. It describes repository behavior and development preferences,
not the deployment status of a running installation. Read alongside AGENTS.md.
Recheck source before editing: names, dependencies, UI behavior, and schema can
change. The user's current request defines the scope.

## 1. Project purpose and environment

CCS HUB is a College of Computer Studies community and academic platform for
students, professors, and administrators. Implemented areas include:

- Authentication, email verification, account recovery, Google OAuth, and profiles.
- Sections, subjects/teaching assignments, schedules, roles, and permissions.
- Posts, announcements, comments, reactions, friends, and notifications.
- Direct, section/subject group, and community-wide Global Chat.
- Floating chat, anonymous identities, replies, message actions, and reporting.
- Livestreaming, Meethub meetings, screen sharing, and attendance.
- Rewards, the $RAM wallet, cosmetic Shop, and Premium subscriptions.
- Private media uploads/storage and administrative moderation/restrictions.

Current workspace: `C:\Users\Administrator\Desktop\CCS_HUB`.
Frontend: `C:\Users\Administrator\Desktop\CCS_HUB\frontend`.
Backend: `C:\Users\Administrator\Desktop\CCS_HUB\backend`.
Shell: Windows PowerShell. Use the actual checkout root if this repo is moved.
All repository-relative paths below resolve beneath that root; final change
reports should link the exact absolute paths of the files changed.

## 2. Verified stack and sources of truth

| Area | Stack / source |
| --- | --- |
| Frontend | React 19, TypeScript, Vite, Tailwind CSS; `frontend/package.json` |
| Routing and state | React Router, Zustand, TanStack Query, Axios |
| UI libraries | Existing Heroicons, Lucide, react-icons, Framer Motion; reuse rather than add equivalents |
| Backend | FastAPI, async SQLAlchemy, Pydantic, python-socketio |
| Runtime dependencies | `backend/requirements.txt`; also inspect `backend/pyproject.toml` because these manifests differ |
| Database | PostgreSQL with asyncpg; database code in `backend/app/core/database.py` |
| Media | MinIO/S3-compatible storage; `backend/app/storage/minio.py` |
| Other infrastructure | Redis, email services; Docker Compose and Celery configuration exist but their presence does not prove runtime readiness |
| Application setup | `README.md`, actual environment/configuration, and verified commands |

Use `frontend/package.json` for frontend commands; the root package manifest is
not the frontend application's manifest. Do not guess installed versions from
README prose. For example, requirements.txt now includes asyncpg and fastapi-mail,
while an older README note says they are missing. Verify before changing setup.
Never copy credentials or secret-bearing configuration into project context.

## 3. Code map

### Project Showcase

`backend/app/api/v1/endpoints/projects.py` exposes authenticated discovery, owner
CRUD, owned raster image uploads and explicit metadata preview generation.
Models/schemas/services use the `project` / `project_*` names. Pending CCS accounts
remain denied; legacy eligibility is preserved. Only students/professors publish,
only owners edit/delete, and existing restrictions, blocks and content filtering
apply. Collaborators receive attribution without edit access. A private interaction
Post anchors existing moderation and canonical PostReaction/Comment/Share records.
Anchors stay excluded from automatic Feed/profile originals; explicit shares
render live project embeds through the existing Feed. Direct social reads/writes,
notification history/counts and socket delivery recheck the current project
audience, eligible owner/viewer, mutual blocks and deletion. Existing admin
deletion tombstones the project through SQL triggers. No extra socket connection
is created; project details reuse usePostRoom and PostDetailModal discussion.

`frontend/src/features/projects/` supplies protected `/projects`, `/projects/new`,
`/projects/:projectId` and `/projects/:projectId/edit` pages, using the shared
Topbar/role sidebars/theme components and `services/api/project.service.ts`.
Showcase navigation appears on desktop and in mobile More; the existing five-item
bottom navigation remains unchanged. Search/category/sort/page/My projects persist
in the URL. The owner form uses project-owned uploads, existing account discovery
for collaborator credits, categorized technology labels and revision-checked
edits. A background refresh never advances the draft's base revision; 409 keeps
the draft for confirmed reload. Unattached discarded images use the backend's
24-hour expiry/cleanup. Website previews are generated explicitly by owners after
saving; only escaped metadata, server-owned images and safe external links render.
ProjectScope cancels/removes private queries on account/eligibility/social revision
changes and rejects late results, without storing credentials in query keys.
Project details expose reactions, discussion, Feed share/unshare and the existing
report dialog. Creator/collaborator responses include existing profile avatar URLs,
signed through the normal storage guard and rendered with the shared Avatar fallback.
Cards show aggregate activity and relative upload times. Community uses icon
actions with accessible labels and the existing usePostView hook for visibility-
qualified, account-deduplicated view recording/counts. Project summaries expose
the canonical anchor views_count; no separate view storage is introduced.
project_activity notices use a
transactional project_deliveries dedupe ledger, suppress self/repeated toggles,
limit each actor/recipient to ten notices/hour and respect project_updates in
Notification Settings. Comments also reuse mention notices and route their focus
to /projects/:id?comment=:id. History/counts and push recheck current source
visibility; deleted source actions are hidden. No historical notices are seeded.
Validation/limits: `docs/PROJECT_SHOWCASE_PHASE_4.md`.

Apply `backend/migrations/20261022_project_showcase.sql`, then
`backend/migrations/20261022_project_community.sql` BEFORE backend rollout;
create_all cannot upgrade existing tables or install SQL integrity/cleanup triggers.
No application/production migration or deployment has been performed. Retain
anchor exclusions during rollback: old backend code would otherwise surface
anchors as ordinary Feed posts. Details: `docs/PROJECT_SHOWCASE_PHASE_2.md`.

Previews use bounded server-side metadata extraction with public-IP connection
pinning, hostname-verified TLS, infrastructure deny hosts and no redirects,
scripts, proxies or external iframe. Unsupported/failed fetches use uploaded image
fallbacks. `PROJECT_PREVIEW_DENY_HOSTS` adds deployment-specific infrastructure
hosts. Decoded static JPEG/PNG/WebP uploads become metadata-free WebP in the existing
private MinIO bucket. Generic serializers cannot renew project image signatures;
authorized project reads can. Existing issued links keep their normal expiry.
Durable preview/upload quotas and `python -m app.scripts.project_storage_cleanup`
(dry run by default, optional --apply/--reconcile) support maintenance without
assuming Celery readiness. Showcase validation results are recorded in the phase
reports; the Showcase test scripts and browser fixture were removed after
validation at the user's request. Existing repository regression tests remain.
Recorded network/storage/realtime checks used fixture boundaries, not live
infrastructure verification.

### Entry points and shared infrastructure

- `frontend/src/App.tsx`: application providers, routing, globally mounted widgets.
- `frontend/src/app/providers/SocketProvider.tsx`: authenticated socket lifecycle.
- `frontend/src/app/providers/QueryProvider.tsx`: shared query infrastructure.
- `frontend/src/lib/socket.ts`: shared socket service, events, and reconnect behavior.
- `frontend/src/lib/axios.ts`: shared HTTP/auth behavior.
- `frontend/src/features/auth/store/auth.store.ts`: authenticated client state.
- `frontend/src/styles/globals.css`: theme tokens and global styles.
- `frontend/tailwind.config.js`: Tailwind configuration.
- `frontend/src/features/dashboard/components/Topbar.tsx`: top navigation.
  Dashboard and standalone Profile, Meethub, Livestream, Chat and Announcements
  shells place it above the sidebar/content row. Its full-width desktop container
  uses an 8px edge inset. Below 640px, a compact 60px header exposes the existing
  search and schedule row through a search toggle; tablet wrapping is unchanged.
  Below 768px, the shared MobileAppShell adds Home, Chat, Create Post, Latest
  (the existing Announcements page) and Videos navigation. Profile remains in
  the account menu and initial mobile feature grid. Friends remains
  available in the hamburger menu. The shell reserves its safe-area height outside
  bounded page scroll containers.
  Chat's badge uses the existing chat-store unreadCount and an icon-relative shared
  badge renderer. Latest uses the backend announcement_reads total, scoped to
  the authenticated account. Opening a card title loads its detail and confirms
  that individual read, then cancels stale count requests, applies the confirmed
  total and invalidates it. Dashboard news and notification links open detail;
  full embedded announcements in post detail also confirm the individual read.
  Reads serialize per account. Listing/expanding cards does not mark them read. Focus, remount and
  reconnect refresh the total; existing socket notifications invalidate it.
  The existing 20261013_announcement_reads.sql migration is required; its
  presence in git does not establish deployment.
  Create Post opens the existing dashboard modal (Admin
  uses PostsPage); keyboard detection and fullscreen hide the bar, and live/meeting
  rooms omit it. The compact Topbar search toggle now extends below 768px;
  phone theme selection is also available in the account menu. Desktop is unchanged.
  Standard Student/Professor mobile Feed scrolling uses useMobileFeedNavigation
  on the existing center scroller: 24px downward travel hides both bars; upward
  travel or returning to the top restores them. Transform-only transitions keep
  feed geometry stable; nested scroll gestures, open overlays and editable focus
  suspend hiding. Navigation changes reset visibility and listeners are cleaned up.
  Browser-toolbar visual viewport resizing does not restore hidden bars; editable
  focus/keyboard resizing does. The binding tolerates late bottom-bar mounting
  and synchronizes its inert state without adding another scroll listener.
  Video-only views and other pages retain their existing behavior. On this Feed,
  keyboard use restores the bars and positions bottom navigation above the keyboard.
  The All feed on Student/Professor phones now starts with a profile welcome
  card, Resources/Sections/Schedule/Code Snippets shortcuts and the existing class
  reminder, independent of featured events. Profile metadata uses the already
  loaded own profile; section labels require a matching profile section ID.
  ResponsiveDashboardWidgets moves one stable portal host between a phone Feed
  slot and the existing desktop rail, keeping Livestream/Meethub consumers mounted
  across breakpoint changes. Existing preview data, requests and socket handlers
  remain shared. Professor's phone snippet shortcut reuses codeSnippetPosts.
  Mobile Home/Create Post reset the local feed filter to All as well as returning
  to the feed. Professor pagination rechecks remaining space on filter changes.
  The final mobile audit and outstanding browser/device checks are recorded in
  `docs/MOBILE_UI_PHASE_6.md`; fixture width coverage is not visual verification.
  Logo navigation uses the authenticated role home. Dashboard callbacks refresh
  existing feed/schedule/announcement loaders (plus admin stats), coalescing logo
  clicks and preserving the URL when already on the feed. Video filters return
  to All when the home logo is used.
- `frontend/src/features/dashboard/components/Sidebar.tsx`: sidebar/mobile drawer.
- `frontend/src/features/dashboard/components/SidebarFrame.tsx`: shared 256px
  desktop sidebar and native modal mobile drawer for standard/admin navigation.
  Shell pages pass mobile open/close state to their sidebar and Topbar; preserve
  focus restoration, breakpoint cleanup, and existing per-role navigation actions.
  Below 768px its existing hamburger dialog now displays MobileNavigationGrid:
  three columns of feature tiles, existing counts and active states, with More
  showing only role-filtered secondary features as compact three-column cards.
  More excludes IDs already in the primary mobile grid and never includes Create Post.
  Student/Professor tiles include the existing
  Code filter and Global Chat conversation; Admin retains administration actions.
  Tablet drawers and desktop sidebars keep their original lists. This grid shares
  sidebar data/actions and adds no API or socket subscriptions.
  The `app-content` class in globals.css owns the 1200px outer content limit and
  responsive padding, including mobile space for the persistent chat launcher.
- `/videos` is a ProtectedRoute for all authenticated, nonpending roles. It reuses
  `useFeed('public-video')`, the existing permission-filtered Feed API and VideoFeed
  player/interactions. Only public uploaded-video posts without announcement links
  enter this view; source pagination advances past nonvideo pages and stops on
  errors for explicit retry. No new backend endpoint, polling or socket connection.
  Feed tiles and VideoFeed now share VideoPlayer controls and videoPlayback's
  session mute preference/single playback owner. One app-level instance of the
  existing viewport autoplay policy observes Feed tiles; VideoFeed keeps its
  slide observer. Off-screen/teardown media pauses without resetting progress.
  `/videos?post=ID&media=INDEX` opens an explicitly selected, REST-authorized post
  through the existing post API (including authorized restricted posts), without
  widening the ordinary public collection. Playback time travels in router state.
  Student/Professor video navigation caches the source Feed snapshot/cursor by
  account, social revision and history entry (up to 30 minutes), and restores the
  source scroll anchor on Back. No duplicated hidden player or dashboard remains
  mounted. Controls support seeking, buffering, three-second idle hiding and
  standard/native fullscreen requests. Validation: `docs/VIDEO_PLAYER_CONTROLS.md`.
- Student/Professor feed views use `ClassReminderCard.tsx` for schedule display.
  It reuses `todayClasses.ts` with a local display tick; no periodic API request.
  Sections/teaching-assignment hooks expose their existing query error state.
  Loading, failed loads, unavailable schedules, empty days and completed days
  are separate states. The next-class helper sorts same-day candidates by time
  and includes next week's recurrence. Each dashboard mounts one widget rail,
  shown via the collapsed-by-default Show widgets toggle on phones (Campus updates
  on tablets). Phone-only spacing and widget transitions preserve desktop sizing. Student/Professor, Admin and standalone Chat, Announcements, Meethub and
  Livestream listing shells are viewport-bound; navigation stays outside their
  content scroll regions. Student/Professor chat uses the same bounded,
  edge-to-edge panel layout;
  the center feed and widget rail scroll independently while navigation stays put.
  Feed cards use a responsive 700px maximum, with a 256px navigation sidebar,
  320px widget rail and at least 24px desktop gaps. At 1400px and above, the
  sidebar and widget rail reach the viewport edges; extra width surrounds the
  centered feed cards. The rail collapses below 1400px.
  Feed pagination listens to the center scroll region and rebinds on section changes.
  Latest News previews the first announcement and links to its existing detail route. Upcoming Events now reads three real, eligible events through the existing API; Student/Professor narrow-screen feeds expose it in a collapsed panel, and Admin overview includes the same widget.
  Student phone widgets also contain the existing class reminder using loaded
  section assignments; desktop retains the schedule popover. Phone typography,
  wrapping and Next-label normalization leave desktop reminder styling intact.
  Home/reminder fixtures: `node --test frontend/tests/home-navigation.test.mjs`;
  these use mocked hook boundaries and server-rendered markup, not live data.
- Below 768px, mounted full-page and floating conversations expose
  `data-mobile-chat-conversation`. Mobile CSS uses that visible-panel marker to
  hide the shared Topbar/bottom navigation, remove shell padding and lock background
  scrolling. Full-page ChatPanel switches to a single conversation at this breakpoint,
  follows visualViewport height/offset with 100dvh fallback and applies all safe-area
  insets. Its list stays mounted across Back. The floating widget can open from
  profile/other existing actions on phones; closing/minimizing or mobile route
  navigation releases the marker. Feed scroll navigation treats the marker as an
  overlay and resets on entry/exit. Desktop floating positioning/controls and chat
  services/socket lifecycle are unchanged. CSS requires `:has()` support in the
  browser. Live phone keyboard/safe-area rendering remains a device validation step.
- `frontend/src/features/dashboard/components/Avatar.tsx`: shared avatar/frame renderer.
- Composer emoji/GIF and post reaction panels share `usePickerPosition.ts`:
  measured visual-viewport bounds, resize/scroll cleanup, and portals outside
  clipping containers (inside the open native dialog when applicable). Phone chat
  tools occupy a second composer row; full-page ChatPanel tracks keyboard viewport
  height below 1024px. Desktop layouts retain their existing sizing. Geometry
  fixtures run during Phase 4 covered requested phone widths and tablet/desktop
  bounds, then were removed per the test-cleanup preference. They do not establish live browser rendering
  or physical keyboard verification. See `docs/UI_UX_PHASE_4.md` for audit limits.
- `frontend/src/components/ui/ConfirmDialog.tsx`: existing confirmation dialog.
- `backend/app/main.py`: FastAPI/Socket.IO ASGI entry point; HTTP API prefix `/api/v1`.
- `backend/app/core/db_types.py`: UTCDateTime handling for existing timestamp columns.
- `backend/app/dependencies/auth.py`: backend authentication dependency.
- `backend/app/core/rate_limit.py`: rate-limiting infrastructure.
- `backend/app/api/v1/endpoints/`: authenticated feature endpoints.
- `backend/app/models/`, `schemas/`, `services/`: persistence, contracts, business logic.
- `backend/migrations/`: existing hand-written SQL migrations.

Login, Register and Google initial account setup share the `auth-background`
presentation in globals.css. The existing root `.dark` theme class selects the
provided light/dark background PNG, with centered cover sizing and a theme-aware
readability gradient. Auth page/grid heights use 100dvh; registration retains its
scrolling card with a viewport-minus-padding cap. Login/Register cards share
`auth-glass-card`: theme-aware translucent surfaces and 16px backdrop blur,
with opaque inputs and a near-opaque fallback without backdrop-filter support.
Validation colors use theme-specific contrast tints. Forms and auth flows are unchanged.

### Google initial account setup

Email verification uses `users.is_verified`, independently of CCS status. Public
and admin-created password accounts start false; admin creation sends confirmation
through the resend service. Own-account auth payloads also expose `email_verified`
from the same persisted flag, retaining `is_verified` compatibility. Explicitly
unverified accounts cannot receive sessions (including via Google), refresh tokens,
or authenticate HTTP/socket requests. Existing flags and legacy NULLs are preserved.
Duplicate unverified registration returns verification guidance without changing
credentials. Resend uses generic responses, existing email limits, an IP limit and
a PostgreSQL-serialized 60-second cooldown; confirmation always validates the current
stored hash and expiry, irrespective of Redis. Deploy the repeatable future-default
migration `backend/migrations/20261019_email_verification_default.sql` before backend
rollout; no application migration or existing-status reset was performed. See
`docs/EMAIL_VERIFICATION_PHASE_1.md` for audit findings and validation limits.
Admin user list/detail/mutation responses expose nullable `email_verified` from
the stored flag and separate CCS state; `email_verified=false` filters explicit
false rows and `counts.email_unverified` counts them globally. Legacy NULL is
shown as unknown, never assumed verified/unverified. User Management renders both
statuses on desktop/mobile/details and combines email filtering with existing
filters. Its existing queries refresh every 30 seconds while visible and on focus;
manual refresh remains available. Registration offers resend on the pending-email
duplicate response. Registration and expired-link screens share one resend button,
generic server messages, 60-second client cooldown and Retry-After handling. See
`docs/EMAIL_VERIFICATION_PHASE_2.md` for tests and browser verification limits.

Google OAuth exchanges the authorization code server-side and requires a verified
Google email and subject. Existing accounts keep the ordinary session path.
Unknown emails receive a signed 15-minute `google_setup` credential and display
`/initial-account-setup`; no account or application session exists before completion.
The setup credential is kept in tab session storage for reloads, never in the auth
store, and cannot authenticate HTTP/refresh/socket operations. Expiry requires
Google sign-in again. `/auth/google/setup` validates the credential, existing
username rules and optional registration-strength password plus confirmation.
Account, Google profile and session commit together under the registration email
lock. Explicit profile flush precedes the first session response because the shared
session factory disables autoflush. Google sign-in failures log generic messages
without Axios request objects/authorization codes. Retries preserve existing account fields. CCS status uses the current
registration rollout policy at completion; the existing pending access gates and
role home routing remain authoritative. No schema migration is required. Deploy
frontend and backend together because new Google sign-ins now return a setup
response instead of application tokens until completion.

### User Settings foundation

- Below 768px, Profile uses a reserved responsive cover, overlapping avatar,
  wrapping identity metadata and a direct owner-only Edit Profile button. Mobile
  Posts/Photos/About navigation retains Shares and owner-only Saved in the More
  activity selector. Photos derives image URLs from already-loaded visible posts
  and opens the existing post detail; it adds no fetch or URL transformation.
  Counts remain the existing loaded posts and own friends, without inferred
  follower/following totals. Uploads, cosmetic rendering, friendship actions and
  profile fields retain their existing handlers. Desktop navigation is retained;
  resizing from mobile Photos to desktop returns to Posts.
- Profile navigation no longer duplicates Security. Password management remains
  in `/settings?section=security`; Active Sessions and Login Activity retain their
  existing Settings destinations and shared security components.

- `/settings` in `frontend/src/App.tsx` uses the shared `ProtectedRoute` for all
  authenticated roles. `frontend/src/features/settings/SettingsPage.tsx` uses
  `?section=` for category navigation, with a mobile list/detail/back pattern.
- Topbar owns one account popover for Profile, Settings and Logout. Logout
  reuses useAuth logout, revokes the current server session and retains existing
  store clearing/navigation and the shared confirmation dialog.
- Account information is read-only, using existing `/auth/me`, `/profiles/me`
  and optional section lookup through the shared Axios client and TanStack Query.
  Security renders the existing ChangePasswordSection only when fresh account
  data explicitly reports `has_password=true`. Google-only accounts show their
  state and link to the existing /setup-password flow. Light/Dark reuse the active ThemeContext.
  `/confirm-password-change` allows both authenticated and unauthenticated
  visitors; the existing backend confirmation token authorizes the change.
- Settings Terms render the exported registration Terms content/effective date
  from TermsPage. Help, guidelines and About are static. Privacy Policy is an
  explicit pending-review destination.
- Phase 2 adds local-password email changes, existing blocked-user list/unblock,
  backend optional announcement/friend notification preferences and private
  support tickets. Google-only email changes remain unavailable because Google
  login identifies accounts by email without provider-subject linking. Profile
  visibility and hidden presence/last-active controls remain unavailable.
- Private state lives in notification_preferences, email_changes and
  support_tickets. Apply `backend/migrations/20261005_user_settings.sql` BEFORE
  backend deployment; it also enforces case-insensitive unique account email.
  Migration verification does not establish application database deployment.
  See `backend/USER_SETTINGS_DEPLOYMENT.md` for contracts and rollout details.
- New authenticated `/settings` API routes reuse the existing auth dependencies.
  Email changes require the local password, current-inbox proof and new-inbox
  proof within 30 minutes; tokens are hashed, scoped, single-use and replaceable.
  Current account data refreshes only after final confirmation. Password token
  email snapshots reject old-mailbox reset/change links afterward; existing
  hashing/session invalidation remain shared. Critical notices bypass optional
  preferences and remain in history if realtime delivery fails.
- Optional policy is applied in existing FriendService/AnnouncementService
  producers before notification rows and pushes. Existing content/actions and
  old notification history stay available; security/moderation bypass the policy.
- Support is independent of content moderation. Own history and admin-only
  inbox/status APIs paginate 25 rows. Admin dashboard Settings mounts the inbox.
  `SUPPORT_EMAIL` is optional official configuration; no private/developer address
  is inferred. No uploads or diagnostics are collected by the support form.
- Phase 3 adds private auth_sessions and sid claims to password/Google sign-ins.
  Apply backend/migrations/20261005_auth_sessions.sql before coordinated backend
  rollout; old tokens without sid require a fresh sign-in. Active Sessions and
  Login Activity read actual own sign-in rows (25/page); earlier and failed
  logins are not backfilled. Browser/device labels are approximate; IPs and raw
  agents are not stored. Individual/all-other revocation is durable for access,
  refresh and sockets; password changes/setup revoke rows in the same transaction.
  The existing Redis cutoff remains an extra gate. OAuth now rejects inactive
  accounts. Socket application events validate before dispatch; passive sockets
  are checked on every worker every five seconds and locally after revocation.
  No additional frontend socket connections or subscriptions are introduced.
- Automatic deactivation/deletion remain pending a reviewed lifecycle policy.
  Settings offers explicitly labeled review requests using existing private
  support tickets/admin inbox. Typed REQUEST plus a confirmation dialog submits
  only a request; it never disables, anonymizes or deletes the account. Existing
  shared content, academic records, moderation evidence, purchases and media
  remain intact. See the deployment document for the relationship audit.
- `frontend/tests/settings.browser.mjs` uses local Playwright/Chromium fixtures,
  not real authentication, email, database or realtime services. It requires an
  available Playwright installation/browser. `SETTINGS_TEST_URL` selects the
  running frontend; `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH` can select a local
  executable. No application credentials are used.

### Feed and posts

- Feed filters, composer, cards, and social actions use the shared surface,
  typography, focus and control tokens. Student top-bar/mobile filters include
  All, Friends, Videos and Code Snippets; Professor feeds still exclude Friends.
  Snippets reuse the content renderer's fenced-code parser and sort by creation
  time. Student pagination rechecks raw page changes and filter switches, so
  nonmatching pages do not stall the code feed; existing cards stay mounted
  while older pages load.
- `frontend/src/features/posts/components/CreatePostModal.tsx` uses a native
  modal dialog with focus containment/restoration and scroll locking. Escape,
  backdrop and close actions remain blocked while uploading/posting. Upload,
  visibility, formatting and submission handlers retain their existing contracts.
- Media retains signed URLs, video-first gallery order and uncropped single-file
  presentation. Code previews preserve whitespace, copy the original snippet,
  and retain their expansion/full-view controls and dark syntax surface.
- Below 768px, cards use compact author/body spacing and persistent reaction,
  comment, share and view totals from existing data. Author roles are shown;
  the post contract supplies no author section, so none is inferred. Existing
  filters are available in the mobile search panel. Single images reserve a 4:3
  frame and videos 16:9 with contain sizing; gallery order, video mounts and
  playback handlers are unchanged. Mobile post details use the existing visual
  viewport sizing with an avatar/input/send row and scrollable attachment tools.
  Fenced code in mobile comments reuses the syntax/copy/scroll renderer; plain
  comments and desktop presentation are retained. Code previews use eight lines
  below 768px and twelve above it.
- Feed fixtures: `node --test frontend/tests/code-snippet-feed.test.mjs` exercise
  filtering, pagination and home refresh with mocked network/layout boundaries.
  These are not browser or live API/socket tests.
- Videos has a Back to Feed control; post/media modals close to their underlying
  page. Native video/iframe fullscreen retains the platform's exit controls.
  Live entry links carry a validated originating application path, falling back
  to role home for direct links. The live stage's return control minimizes the
  existing session, including exiting supported DOM fullscreen, without ending
  or reconnecting it.

- Post engagement counts are persistent on feed cards and Profile postcards.
  Postcards show their top three emoji and reaction total in a compact row with
  comments/views, replacing the Likes count. Detail Likes totals still combine
  legacy binary likes and current emoji responses. The Like/emoji control writes
  PostReaction rows. Cards
  and profile tiles subscribe through the existing reference-counted socket rooms.
  `usePostRoom` filters every event by post ID and reads current callbacks, so
  another open post cannot overwrite modal comments/reactions. New committed
  views broadcast aggregate-only `post:views_updated`; legacy likes broadcast
  `post:like_updated`. Neither event creates a socket connection. Live view
  updates do not modify the 24-hour view-recording cache timestamp.
  Modal comment totals use the server count rather than the first loaded page.
  Code blocks retain independent two-axis scrolling; post modal height follows
  visualViewport and its mobile composer separates tools from input/send.
- Post views use POST `/posts/{post_id}/view`, existing authentication/CCS guards
  and PostService visibility checks. PostgreSQL serializes a rolling 24-hour
  user/post deduplication record and cumulative counter in one transaction.
  Only aggregate counts are public to authorized readers; viewer records stay
  private. GET/feed requests never record views. The shared account/post query
  records after one second of foreground visibility and avoids duplicate requests
  from reposts/modal reopening. Views do not update the post edit timestamp.
  Apply `backend/migrations/20261010_post_views.sql` before backend deployment,
  then deploy frontend. No historical backfill; no application DB migration was
  performed. Policy, rollback and validation: `docs/UI_UX_PHASE_2.md`.

### Sections

- Section dashboard, section cards/member management, and ProfessorTeachingHub
  use shared theme surfaces and control styles. Teaching cards and student grids
  adapt to available content width, including the sidebar and directory columns.
- Create/join/edit section, add-student, and add/edit subject forms use labeled
  inputs, selected-day semantics, larger controls and viewport-bounded panels.
  Existing nested modal/confirmation behavior, role gates, membership operations,
  teaching-assignment payloads, attendance calculations and chat hooks are retained.

### Weekly schedule

- `frontend/src/features/dashboard/pages/ClassesPage.tsx` presents the existing
  Monday?Sunday occurrences in a grid that adapts to the available content width.
  Status, ordering, counts and Student/Professor actions use their existing data.
- Class details use a native modal dialog with focus restoration, scroll locking
  and viewport-bounded content. The page retains its internal scrolling contract.

### Announcements

- `/announcements/:announcementId` mounts the existing AnnouncementDetailPage
  behind ProtectedRoute. Section cards, shared links and search retain the ID;
  the authenticated detail API remains authoritative for visibility. Detail Back
  returns through router history, with `/announcements` for a direct-link entry.

- Announcement create/update normalizes owned storage URLs to object keys; response
  serialization signs `image_url` with the same helper as embedded announcements.
  MinIO upload/bucket operations use `MINIO_ENDPOINT`/`MINIO_SECURE`; signing uses
  `MINIO_PUBLIC_URL` when configured (production HTTPS origin, without a path).
  Announcement lists refresh every five minutes and on focus; detail state refreshes
  its image on the same interval and when the tab becomes visible. The backend
  caches 24-hour signatures with a 30-minute rotation margin. See
  `backend/ANNOUNCEMENT_MEDIA_DEPLOYMENT.md` for deployment and verification.

- Shared announcement feed, filters, cards, detail view and sharing controls use
  the established surface and control styles. Card keyboard activation ignores
  events from nested controls so those controls retain their own actions.
- `frontend/src/features/announcements/components/CreateAnnouncement.tsx` uses a
  viewport-bounded native dialog with focus restoration and scroll locking. It is
  shared by dashboard, section and admin consumers. Role/officer checks, target
  sections, upload handling, expiry conversion and publishing payloads are unchanged.

### Friends

- Friends screens share theme-aware cards, filter controls and 44px action targets
  across the Student/Professor dashboard sections and standalone `/friends` route.
- `frontend/src/features/friends/components/ReportUserDialog.tsx` uses a native
  modal dialog with focus restoration, scroll locking and viewport-bounded content.
  Dismissal remains disabled during submission; report reasons and payloads are
  unchanged. Friendship, search and chat hooks retain their existing behavior.

### Chat

- `frontend/src/features/chat/components/ChatWidget.tsx`: floating launcher/panel.
- `frontend/src/features/chat/hooks/useFloatingChatPosition.ts`: panel position/bounds.
- `frontend/src/features/chat/components/ChatWindow.tsx`: message layout, composer,
  anonymous selector, reply previews, and existing action handlers.
- `frontend/src/features/chat/components/AnonymousIdentityControl.tsx`: alias UI.
- `frontend/src/features/chat/components/MessageReactions.tsx`: existing reactions.
- `frontend/src/features/chat/components/ChatPage.tsx` and `ChatPanel.tsx`: full chat.
- `frontend/src/features/chat/hooks/useChat.ts` and `store/chat.store.ts`: shared chat access/state.
- `frontend/src/features/chat/globalChat.ts`: Global Chat ID/preferences/display helpers.
- `frontend/src/services/api/chat.service.ts` and `types/chat.types.ts`: API/types.
- `backend/app/models/conversation.py`: Conversation, Message, membership/reaction models.
- `backend/app/models/user.py`: authenticated users and private anonymous preferences.
- `backend/app/schemas/chat.py`: chat contracts and anonymous serialization safeguards.
- `backend/app/services/chat_service.py`: sends, history, aliases, permissions, replies/actions.
- `backend/app/api/v1/endpoints/chat.py`: REST endpoints.
- `backend/app/websocket/manager.py`: active Socket.IO chat handlers/broadcasts.
- `backend/app/services/moderation_service.py`: reporting/restrictions/moderation.

Do not assume `backend/app/websocket/chat_handler.py` owns the active send path
merely because of its filename. Verify imports and handler registration; the
working path reviewed here is in manager.py.

### Shop and adjacent features

- `frontend/src/features/shop/components/ShopSection.tsx` and `ShopSection.css`.
- `frontend/src/features/shop/hooks/useEquippedCosmetics.ts`.
- `frontend/src/services/api/reward.service.ts`.
- `frontend/src/features/dashboard/pages/StudentDashboard.tsx`,
  `ProfessorDashboard.tsx`, and `AdminDashboard.tsx`: pass the existing avatar to Shop.
- Student/Professor sidebar navigation exposes Rewards and Shop through the existing
  dashboard sections. Admin retains Shop but has no Rewards navigation or view.
- `backend/app/services/shop_service.py`, `backend/app/models/reward.py`, and
  `backend/app/api/v1/endpoints/rewards.py`: shop/reward persistence and operations.
- `frontend/src/features/livestream/`: livestream AND Meethub frontend components/hooks.
  Do not invent a separate `frontend/src/features/meethub/` directory.
  Listings use unscaled, full-width responsive card grids. Meeting participants
  use a 384px desktop rail; expanded livestream chat uses a 320?400px rail.
  Below desktop, the existing meeting drawer and stacked stream/chat layout remain.
- `backend/app/api/v1/endpoints/meethub.py` and `livestream.py`: corresponding endpoints.
- Active Livestream/Meethub discovery shares account-scoped TanStack queries across
  listing pages, widgets and Sidebar counts. One SocketProvider bridge handles
  existing started/ended events and the content-free `stream:activity_changed`
  signal, batches refreshes and cancels stale list requests. Sidebar API polling
  was removed. Focus and socket reconnection reconcile missed events.
  List endpoints accept opt-in `active_only=true`, retaining existing status and
  permission checks while requiring an online broadcasting host or a joined
  MeetHub organizer. Default list/schedule/history behavior is unchanged.
  MeetHub entry windows and kicked-user exclusions filter only active discovery,
  with a local boundary timer;
  attendance, scheduled records and history are retained. Live dashboard previews
  show only active streams; upcoming cards remain on `/livestreams` separately.
  Presence uses the existing process-local socket manager; HTTP and sockets must
  share that runtime, as with existing WebRTC signaling. No schema migration.
  Deploy backend and frontend together (backend first). Validation and limitations:
  `docs/ACTIVE_FEEDS_PHASE_2.md`.
- Livestream and Meethub reuse `frontend/src/features/livestream/rtcConfig.ts`
  for the existing STUN/TURN routes. Livestream queues early ICE until remote SDP
  is applied and accumulates remote tracks in one stream. Both retain peers during
  transient disconnection and register signaling listeners before joining.
  Meethub falls back to muted video with an Enable audio button when browsers
  block audible autoplay.
  Public relay availability and live cross-network delivery require separate verification.
  `node --test frontend/tests/live-media.test.mjs` checks signaling ordering, track
  attachment/remount, transient disconnection and autoplay using mocked boundaries.

Chat role badges use additive display hints from ChatService: sender_role,
sender_is_mayor and sender_can_verify_ccs on named messages, and role/is_mayor/
can_verify_ccs on existing conversation participants. Stored account roles and
current SectionMember rows joined to existing Sections supply the labels; inactive
or pending accounts do not get Mayor designation. Issuer hints reuse ccs_access
and exclude current moderation restrictions; they never authorize an operation.
Role metadata is cached only within one request/socket event, with Mayor lookups
batched for history/participants. Anonymous service serialization skips these
lookups and MessageResponse forcibly clears every role/issuer hint; reply previews
stay unchanged. Global Chat still exposes no participant roster or new section data.
ChatRoleBadges reuses RoleBadge (Professor/Admin/Mayor) across sender labels, DM
headers/list items and group members. Group member Mayor labels remain scoped to
the group's actual section. Pending students' Request CCS Verification action
opens the selected eligible Professor/Mayor/Admin through the existing direct
conversation endpoint and shared useVerificationChat hook. It sends no message
or code automatically. The existing exact-section contact lookup uses the same
hook and retains its authorized section labels. Generic Request Verification
links focus that lookup on /verify-student. Labels are snapshots at fetch/send
time, with no new role socket event; actual operations recheck current authority.
The pending-only direct-pair gate rechecks mutual blocks and both participants'
moderation restrictions on reads/writes/room access/delivery; existing pair locks
remain owned by social writes. Malformed or non-pair direct memberships are denied
to pending accounts. Verification participant payloads retain the email field as
an empty string for both parties, rather than disclosing private contact data.
Existing 3-new-contact/day and 10-text-send/hour database quotas remain authoritative,
with the existing 2,000-character/no-media verification limit. Pending users retain
the floating chat route, with Global Chat/help actions and no unrestricted friend
picker. Their chat components disable unrelated useFriends reads/subscription;
other consumers keep the default behavior. No posting, section, meeting or other
protected permission is granted by opening verification chat. Verification guidance
lives inside the message scroller so it does not take fixed composer space.
Local validation and outstanding browser/production-runtime checks are recorded in
`docs/AUTH_CHAT_VERIFICATION_PHASE_5.md`.

## 4. Current chat behavior to preserve

Global Chat already works; extend its implementation rather than rebuilding it.
The frontend's GLOBAL_CHAT_ID is shared with the backend's community conversation.
Normal, anonymous, direct, and group behavior must remain correctly separated.

- SocketProvider owns connection lifetime based on authentication. Reuse the
  singleton socketService and chat store. Opening, dragging, minimizing, editing
  aliases, or changing presentation must not create another connection.
  HTTP and socket recovery share `refreshAccessToken` in `lib/axios.ts`; refresh
  endpoint failures must never recursively refresh. Only invalid refresh
  credentials end the session; network/server failures retain the launcher.
  Socket authentication uses fresh credentials for every handshake, and backend
  handshake rejection uses Socket.IO `ConnectionRefusedError` (`connect_error`).
  Reconnect keeps the same socket/listeners, rejoins remembered rooms, and uses
  `features/chat/lib/restoreChatHistory.ts` to recover missed pages before
  refreshing conversation/unread counts. Logout cancels recovery and clears chat.
  Transient recovery has bounded backoff; online/pageshow/visible-tab events can
  resume exhausted network retries. Repeated authentication rejection is capped.
- Verify matching listener cleanup with the original callback reference. Existing
  useChat consumers can each register message:read listeners; do not add new ones
  as a side effect of a UI task or claim this architecture is globally deduplicated.
- On the pending verification page, Open Global Chat selects the loaded canonical
  conversation and opens the floating panel directly without navigating away.
  The button waits for that conversation to load. Minimizing restores its globe
  head in ChatHeadStack, which reopens Global Chat. The stack reuses
  chat.store state and the existing ChatWidget/socket lifecycle. Saved head
  positions are clamped on mount; query devtools use bottom-left to keep the
  bottom-right chat heads accessible. Pending users cannot maximize
  or navigate to the full /chat page; its account-aware guard redirects to
  /verify-student. Global Chat and approved verification DMs use the floating
  panel. Legacy and verified accounts retain full chat access.
- The floating launcher is a globe icon. Its drag/position behavior has been
  updated separately; inspect current ChatWidget.tsx before changing it.
- The panel is header-draggable on desktop (1024px+), uses measured viewport
  bounds and session position, and uses a full visible-viewport layout below that
  breakpoint. Preserve keyboard viewport tracking and safe-area spacing.
- Keep header/composer accessible and the message list independently scrollable.
  Preserve backreading and existing auto-scroll behavior. Latest-message and
  reply jumps scroll only the message container; follow-latest tracks the
  reader's position before new messages change the list height.
- Own Global Chat message options sit LEFT of the bubble. Received-message
  options retain their existing right-side placement. Do not apply this global
  own-message adjustment to direct/group conversations unintentionally.
- Preserve Reply, Unsend, Remove for Me, reporting, reactions, attachments, and
  other existing action behavior when changing layout.
- The top-bar Messages button has been removed. NotificationBell remains;
  actual Chat routes/features and the floating launcher remain available.
- Respect existing layering and immersive/modal handling. Do not solve layering
  issues with arbitrary enormous z-index values.

## 5. Anonymous identity: privacy and persistence

Anonymous means hidden from the community, not untraceable by CCS HUB moderation.

- The server generates neutral word combinations with a random six-digit suffix
  using Python secrets. Generation takes no name, email, username, student ID,
  section, profile data, account hash, or sequential identifier as input.
- `users.anonymous_username` persists the current alias; the nullable
  `anonymous_username_changed_at` records custom-name changes.
- Current aliases are unique case-insensitively through a PostgreSQL unique index.
  Generated collisions retry safely. Past aliases are not permanently reserved.
- Frontend AND backend validation accept 3-24 ASCII letters, digits, or underscores.
  Reject whitespace, control characters, HTML, and unsupported characters.
  No existing profanity filtering was found in the inspected path; do not claim
  it exists. Recheck before adding a content-filtering dependency/system.
- The first custom edit is immediate. Actual subsequent changes have a server-side
  24-hour cooldown; saving the identical name does not extend the cooldown.
- User-row locks serialize identity creation, edits, and anonymous message sends.
- `messages.anonymous_alias_snapshot` records the alias at send time. Renaming
  affects future messages only. History and reply previews use the saved alias.
- Legacy NULL snapshots fall back to `Anonymous Student`; never use the real
  profile name as an anonymous fallback.
- Public anonymous messages omit the real avatar and use an empty sender_id.
  The real Message.sender_id stays in storage for ownership/reports/moderation.
- Anonymous account preferences must not be added to public account/profile
  schemas where they could link a public profile to an anonymous identity.
- The backend resolves send-time identity; do not trust a client-supplied alias.
- Authenticated `POST /api/v1/chat/anonymous-identity` creates/returns the caller's
  identity; `PATCH` updates only that caller's alias. There is no public lookup
  mapping aliases to real accounts and no additional socket/event system.

Required additive migration:
`backend/migrations/20260927_anonymous_chat_usernames.sql`.
Apply it before deploying code that selects the new columns. The earlier work
verified it in an isolated PostgreSQL database, not the application database.
Current deployment status is unknown and must be checked, never inferred.

## 6. Shop UI and transaction boundaries

Shop already has working purchase, equip/unequip, ownership, expiration, Premium,
and wallet logic. Preserve those paths when making visual changes.

- Header balance and all products come from real shop response data.
- Featured cosmetics are selected from the existing catalog; no fake products.
- Previews reuse the logged-in user's avatar and actual item config: frame ring,
  name color/gradient, or profile-background gradient.
- Cards use existing rarity data and restrained accents. Keep the CCS HUB dark
  premium design, accessible contrast, and existing theme compatibility.
- The grid adapts to available container width, including sidebar space:
  one column by default, two at 580px, three at 900px.
- Preserve existing category filters, mobile horizontal scrolling, touch targets,
  disabled insufficient-balance states, and owned/equipped/Premium distinctions.
- Reuse rewardService methods and the existing confirmation dialog. Purchasing
  moves $RAM and must retain the existing user confirmation step.
- Backend remains authoritative for balance, entitlements, duration, and expiry.
  Do not calculate wallet transactions or grant inventory in presentation code.

## 7. Coding workflow and communication

1. Read the user's request, AGENTS.md, and relevant context. Honor explicit scope.
2. Inspect git status/diff, actual files, callers, and dependencies. Preserve
   unrelated user work, including concurrent changes.
3. Diagnose the actual cause. Search with rg before inventing paths or symbols.
4. Implement the smallest complete change through existing components/services.
5. Verify the relevant behavior and inspect the final diff for unintended edits.
6. Report exact changed-file paths, the outcome, relevant test results, and any
   material remaining limitations or migration/deployment steps.

Be direct. Avoid unnecessary introductions, speculative warnings, or unrelated
suggestions. Report meaningful additional affected files and why they are needed.

When workspace access is available, read and modify source directly; do not ask
the user to run commands or paste files that the agent can already inspect.
Ask for missing errors/source only when necessary after using available evidence.
Do not hide a failure with a workaround instead of diagnosing its cause.

For manual-only assistance, provide either a complete file or exact find/replace
blocks verified against the current source. Do not omit sections in a purported
full replacement. Multiple edits do not justify overwriting unrelated code when
a precise patch is available. Never require whole-file output just because a
file is long.

## 8. Security, database, and API safety

- Preserve authentication, backend authorization, private media, restrictions,
  rate limits, and moderation. Frontend visibility is not authorization.
- Never log, paste, hardcode, or commit credentials, tokens, keys, or secret .env
  contents. Existing configuration defaults are not approved deployment secrets.
- Inspect models, schemas, services, endpoints, and frontend callers together
  before changing a contract or persistence behavior.
- Do not invent existing fields/endpoints. New fields/endpoints are allowed when
  required by an authorized feature, with explicit compatible implementation.
- Prefer additive, default-safe/nullable migrations that preserve existing data.
  Do not recreate tables or rely on SQLAlchemy create_all() for schema upgrades.
- Do not assume Alembic migrations exist merely because Alembic is a dependency;
  inspect the repository's actual SQL migration workflow.
- Keep migration creation, migration testing, and application to a target database
  distinct in reports. Confirm the target/environment for database operations.
- Tests must use disposable fixtures/databases. Do not point destructive test
  setup or cleanup at application/production data.
- Avoid unrelated dependency upgrades, component duplication, architecture
  rewrites, or styling churn. Reuse shared helpers and existing icon libraries.

Email transport uses `resend[async]==2.50.0` and the existing EmailService templates.
`RESEND_API_KEY` and `RESEND_FROM_EMAIL` are loaded through Settings from the
backend `.env` (or environment variables). Run backend commands from `backend`
with `python -m uvicorn app.main:app` using the environment containing FastAPI;
use the deployment virtual environment on the VPS. Provider acceptance requires
an email ID; it does not establish inbox delivery. Recovery/resend responses
remain generic to prevent account enumeration; delivery failures are logged
without provider diagnostics or message content. Legacy SMTP settings remain
for configuration compatibility.

## 9. PowerShell commands and verification

Examples below assume the current checkout path. Inspect relevant scripts first
if they have changed. Use `-LiteralPath` for literal filesystem operations.

```powershell
Set-Location 'C:\Users\Administrator\Desktop\CCS_HUB'
git status --short
rg --files frontend/src/features/chat
rg -n 'anonymous_username' backend/app frontend/src
Get-Content -LiteralPath 'C:\Users\Administrator\Desktop\CCS_HUB\frontend\src\features\chat\components\ChatWindow.tsx'
git diff --check
```

Frontend build (TypeScript checking plus Vite production build):

```powershell
Set-Location 'C:\Users\Administrator\Desktop\CCS_HUB'
npm --prefix frontend run build
```

Frontend dev server, when needed and not already running:

```powershell
npm --prefix frontend run dev
```

Backend chat regressions, using the configured Python environment/dependencies:

```powershell
Set-Location 'C:\Users\Administrator\Desktop\CCS_HUB\backend'
python -m unittest app.tests.test_global_chat app.tests.test_anonymous_usernames -v
```

Socket/auth fixture regressions (no application database writes):
`node --test frontend/tests/socket-session.test.mjs` from the repository root,
and `python -m unittest app.tests.test_socket_session -v` from `backend`.

The PostgreSQL integration cases in test_anonymous_usernames.py require the
`CHAT_ALIAS_TEST_DATABASE_URL` environment variable pointing at a disposable
PostgreSQL database whose name ends in `_test`; without it those cases SKIP.
Each test uses a separate schema. Never count skipped cases as passed database
coverage or substitute the application's DATABASE_URL. No actual database URL or
credentials belong in this document.

Backend development entry point, only with required services/configuration ready:

```powershell
Set-Location 'C:\Users\Administrator\Desktop\CCS_HUB\backend'
python -m uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

Verification expectations:

- Run appropriate checks rather than assuming every change needs the full suite.
- Frontend npm test currently only prints `No tests specified`; it is not coverage.
- A production build proves compilation/bundling, not live API/realtime behavior.
- Backend pytest configuration lives in pyproject.toml; inspect fixtures before
  broad runs. The commands above target the existing chat unittest suites.
- Browser-check layout changes at relevant widths, especially 320, 360, 375, 390,
  412, 430px and desktop sizes. Check long content, overflow, touch targets, and
  keyboard-sized viewports where relevant.
- For chat: verify own/received messages, anonymous rendering, replies/actions,
  scrolling, and no new subscriptions/refetches caused by presentation changes.
- For Shop: verify real-data binding, previews/filters, confirmation, disabled
  states, ownership, Premium, and temporary/permanent duration presentation.
- Fixture-backed browser checks are not live backend/network tests. State which
  kind ran. Report actual failures, limitations, and existing warnings honestly.
- A previous run passed 34 chat/alias tests with isolated PostgreSQL and browser
  checks at 320-1440px. This is historical evidence, not proof for future edits.
- Vite has reported a large-bundle warning. ESLint previously encountered root
  config dependency-resolution issues; recheck rather than calling lint passed.

### Campus Events (Phases 1-5)

- Events are independent of teaching assignments and announcements. UUID models
  `Event`, `EventSection`, `EventResponse` live in `backend/app/models/event.py`.
  Timestamps use the existing UTCDateTime/UTC-storage convention. Composite
  primary keys prevent duplicate section targets and duplicate user responses.
- Apply `backend/migrations/20261007_events_foundation.sql` before backend rollout.
  It only adds Events tables/indexes. Its isolated PostgreSQL test application
  does not establish application database deployment. See `docs/EVENTS_PHASE_1.md`.
- Authenticated GET `/api/v1/events/` returns `{total, items}` with skip/limit,
  search, category and all/upcoming/week/past filters. GET `/api/v1/events/{uuid}`
  uses the same SQL audience predicate and returns 404 for hidden/missing events.
  Public events are authenticated campus-wide. Section events are scoped to
  members, professor advisors/active teaching assignments, creator and admin.
  Academic visibility does not depend on social blocks.
- Upcoming includes ongoing activities until their end; past uses end <= now.
  This Week ends at next Monday midnight in Philippine time (UTC+8). The UI
  explicitly labels Philippine time and reuses parseServerDate. Covers store
  object keys and reuse presign_stored_url only after authorization. Responses
  expose aggregate counts, safe host name/avatar, and no attendee identities.
- Events UI redesign adds a compact featured dashboard, cover cards, cached
  upcoming and true own-response widgets, and Month/Week/List calendar with mobile
  agendas. Calendar uses the existing <=31-day overlap API, adds category only to
  its calendar projection, and explicitly paginates beyond 100 items. Shared
  EventForm has grouped fields, cover drop/preview and inline validation. Detail
  retains real response controls and adds Copy Event Link. Global shell stays intact.
- Protected `/events/manage` uses the same professor/admin roles and existing
  writes. Professors default to My Events; admins to All Events. Authenticated
  GET `/events/manage` returns server-filtered/paginated management rows with
  current edit/delete hints and target names only for managers. Discovery accepts
  optional `response_status=going|interested` and returns actual viewer_response,
  using current visibility plus the authenticated user's own response. These are
  additive reads with no new model/migration. See `docs/EVENTS_UI_REDESIGN.md` for
  implementation, deployment order and verification limits.
- Protected `/events` and `/events/:eventId` reuse Topbar, Sidebar/mobile drawer,
  shared Axios, TanStack Query, Avatar and theme tokens. The disabled Events
  entry is activated; admin navigation also links to Events. Discovery has
  filters, pagination, featured presentation, fallback images and request-driven
  loading/error/empty states. Phase 2 adds native create/edit EventForm and shared
  confirmed deletion. The form sends Philippine UTC+8 times explicitly, even when
  the browser uses another timezone. Phase 3 adds Going/Interested controls with
  explicit clearing, current state, pending/error feedback and aggregate counts.
  Phase 4 adds publish notices and the Upcoming Events widget. Automatic reminders
  are deferred for missing scheduler infrastructure. Phase 5 links announcements
  to Events and adds Events/All modes to the existing weekly Schedule.
- POST `/events/` and PUT `/events/{uuid}` accept multipart `data` (JSON EventWrite)
  plus optional `cover`. DELETE `/events/{uuid}` returns 204. Role checks restrict
  writes to professor/admin; professors edit their own events and must retain
  permission for old/new section targets. They may delete their own event even
  after losing section assignments. Admins manage all. GET `/events/target-sections`
  returns only advisor/active teaching sections for professors, all for admins.
  Actual target validation reuses SectionService's strict professor/admin helper.
- Announcements optionally accept `event_id` on create/update. An omitted update
  preserves the association; null removes it. Responses expose only a viewer-scoped
  `event_reference` preview, never the raw FK. Current title/time/location are read
  from Event with the same audience predicate, including shared announcement cards
  and admin listings. Hidden/deleted references return an unavailable marker.
  Deletion sets the FK null while retaining `event_linked`, preserving announcement
  text and allowing a safe unavailable card. No Event body is copied into text.
- Apply `backend/migrations/20261007_event_integration.sql` after the foundation
  migration and before the Phase 5 backend. It adds nullable announcement event_id,
  a deletion marker, SET NULL foreign key and index. Test migration runs do not
  establish deployment to the application database. See `docs/EVENTS_PHASE_5.md`.
- Schedule defaults to existing Classes. Events/All query authenticated
  GET `/events/calendar?start=...&end=...` with a timezone-aware positive range of
  at most 31 days, skip/limit <=100, SQL audience filtering and interval overlap.
  The UI displays the current Monday-Sunday Philippine week, 50 events per page,
  including completed and multi-day events. Event links open Event Details; class
  occurrence logic, details/actions and teaching assignment models stay separate.
- EventReferenceCard uses the existing account-scoped detail cache and focus
  refresh; errors hide the preview. EventPicker supports visible upcoming/past
  search and pagination in existing announcement create/admin edit dialogs.
- Write schemas reject blank/oversized text, invalid enums, naive timestamps,
  end <= start, inconsistent/duplicate targets and unrequested fields. Covers are
  JPEG/PNG/GIF/WebP up to 10MB, verified with existing Pillow before the existing
  MinIO uploader stores them under `events/{event_uuid}`. Caller-supplied keys/URLs
  are not accepted. Remove-cover clears only the association; replaced/deleted
  event media is retained. Failed database writes clean only the new upload.
- Events reuse account restrictions and ContentFilterService with source EVENT.
  Title/description/location are sanitized in one submission; event, violation,
  restriction, private warning and new-event publish notices commit together.
  Existing moderation warning delivery and account-only event notification delivery
  run after commit; realtime failure leaves saved notification history intact.
  Row locks serialize writes; generated timestamp defaults are refreshed before
  responses to keep explicit UTC. EventDetail returns can_edit/can_delete and
  target section IDs only to managers. See `docs/EVENTS_PHASE_2.md` for rollout.
- PUT `/events/{uuid}/response` accepts JSON `{status: "going" | "interested" | null}`
  for any authenticated, unrestricted viewer. Null deletes only that viewer's
  response. EventDetail adds viewer_response; no attendee identities are exposed.
  Account then event row locks serialize updates with event management; visibility
  is rechecked after locking and PostgreSQL ON CONFLICT preserves uniqueness.
  Counts/current state are returned in the committed detail response. The UI
  updates its account-scoped detail cache and invalidates event lists. No new
  migration is needed for response actions. See `docs/EVENTS_PHASE_3.md`.
- New-event notices reuse Notification rows, the existing friends notification API,
  and manager.send_to_user/new_notification. Public events notify active accounts
  except the creator; section events notify distinct active targeted members except
  the creator. Edits and response writes do not send publish notices. Apply
  `backend/migrations/20261007_event_notifications.sql` before Phase 4 backend
  rollout: its partial unique index prevents duplicate publish notices per user/event.
  No existing event is retrospectively announced. Event notice history/unread counts
  recheck current Events visibility, hiding notices after audience loss/deletion.
  The bell uses a calendar icon and navigates to protected Event Details.
- Dashboard EventCardList replaces sample data with cached GET events limit=3,
  reusing existing backend filters/pagination. Student/Professor mobile and desktop
  observers share an account-scoped query; admin overview uses the same component.
  Date/time is Philippine time, with real loading/empty/error/retry states and
  detail/View All navigation. Existing new_notification handling invalidates event
  lists for publish notices; no new connection or listener was added.
- Automatic event reminders are intentionally deferred. The Celery stub names
  missing email/notification task modules and undeclared broker/result settings;
  no Beat schedule/service or durable reminder/retry mechanism exists. Required
  worker/scheduler, idempotency, current-visibility checks and operational readiness
  are documented in `docs/EVENTS_PHASE_4.md`. No startup loop/frontend reminder
  timer was introduced.
- `python -m unittest app.tests.test_events -v` uses fixture auth/storage and real
  PostgreSQL only when EVENTS_TEST_DATABASE_URL targets a disposable *_test DB.
  `frontend/tests/events.browser.mjs` uses fixture APIs/auth/sockets with Playwright;
  EVENTS_TEST_URL and optional Playwright module/executable paths select tooling.
  `frontend/tests/events-management.browser.mjs` covers Phase 2 management flows
  at the same ten widths with mocked network/auth/media/socket boundaries.
  Discovery browser tests also cover response switching, clearing, counts, refresh,
  pending controls and error recovery. `frontend/tests/events-notifications.browser.mjs`
  covers all three role dashboards, widget caching/navigation/states and event
  notification display/read/navigation at ten widths with mocked service boundaries.

## 10. Maintaining this context

### CCS student verification foundation

`users.ccs_verification_status` is nullable: NULL preserves legacy access under
existing rules; `pending`/`verified` are independent of email `is_verified`.
No user backfill/default or existing-account reclassification occurs. Phase 2
enrolls new students through password/Google/admin creation only when the shared
`ccs_verification_rollout` singleton is enabled and worker readiness flag
`CCS_STUDENT_VERIFICATION_ENABLED` is true. Both default off. FOR SHARE locks
serialize enrollment with switch changes; same-email creation uses advisory
transaction locks. Existing unverified accounts cannot be deleted/replaced by
re-registration; they use the existing resend-email flow. Account/profile and
professor/admin code consumption commit together. Own-account auth payloads
expose CCS state separately from public search schemas. Google password setup
replacement tokens include a persisted sid. Pending sign-ins/student home route
to `/verify-student`, with settings, logout, Global Chat and account-bound code entry.
Phase 3 centralizes pending authorization in `services/ccs_access.py`: authenticated
REST access defaults to deny except exact account/settings and permitted chat
routes, with `CCS_VERIFICATION_REQUIRED` 403s and private/no-store responses.
Chat lists filter protected previews. Pending users access the canonical Global
Chat and two-member verification DMs with active admins, assigned/advising
professors or current section Mayors; officer status alone is insufficient.
Verification messages are text-only, <=2000 characters, 10/hour/sender across
verification DMs and three new conversations/day/pending account, serialized by
PostgreSQL advisory transaction locks. Existing blocks/moderation remain required.
Socket dispatch, manager room entry, session sweeps and central outbound delivery
check stored state; old rooms/global broadcasts cannot grant protected access.
Pending media upload/signing, profile media/section assignment and notification
history/push are denied; settings notification preferences remain available.
Previously issued/shared MinIO bearer URLs remain usable until expiry. Outbound
authorization adds database reads and fails closed on outages; load/multi-worker
and real browser/WebRTC verification remain outstanding.
Phase 4 adds separate `ccs_student_codes`/`ccs_code_audit` tables, leaving staff
signup invitations unchanged. Codes use 256 bits of random entropy, SHA-256-only
storage, explicit eligibility confirmation, an immutable recipient account,
section scope, and 1/24/72-hour expiry. Active admins, section advising/teaching
professors and current section Mayors may issue; Officers may not. Issuer scope,
activation and moderation are rechecked at redemption. Authenticated attempts
are counted durably (20 issuance/hour, 10 redemption/15 minutes/account), including
invalid codes. A locked pending, active, email-confirmed student and a locked
unused code commit regular membership, profile section, verified state, consumption
and audit together. Conflicting sections/privileged or duplicate memberships need
review; no section transfer or role promotion occurs. Existing membership writes
share a commit-free, per-user-serialized helper. Academic chat sync runs after
commit and can be retried through existing conversation opening flows.
SectionDetailModal embeds issuer controls; plaintext exists only in the issuance
response/component and optional explicit clipboard copy. No history/recovery API
or code storage in local/session storage exists. Pending code entry refreshes
own-account state, including after a lost response, then follows existing verified
navigation without replacing tokens or creating another socket.
Phase 5 adds a pending-only exact section-name contact lookup at
GET /ccs-verification/contacts, returning at most 20 available authorized Mayors
and advising/actively teaching Professors with minimal section labels. Mutual
blocks, inactive/pending issuers and current moderation restrictions exclude
contacts. Lookup uses the existing Redis atomic counter (20 per 15 minutes;
existing fail-open behavior on Redis outage). Open verification chat reuses the
existing direct-conversation endpoint, rechecking current authority, blocking
and restrictions; it neither sends a message nor issues a code automatically.
The existing PostgreSQL-backed DM quotas remain authoritative. Verification
composers show text-only limits to both parties; normal legacy/verified chats
keep attachments and their existing character limit. Global Chat shows a small
Pending Verification badge for named pending senders, from the shared REST/socket
serializer. Anonymous messages suppress that flag in backend and frontend.
Status badges reflect the last message/history fetch, not a new socket event.
See docs/CCS_VERIFICATION_PHASE_5.md for implementation and historical validation.
Phase 6 wraps the existing providers with CCSAccountProvider, sharing a cancellable
own-account query keyed by account ID and a nonsecret session version. Replacement
credentials for the same account trigger a fresh check. ProtectedRoute waits for
a fresh initial /auth/me response,
syncs server role/CCS state and defaults to redirecting pending users to onboarding.
Only verification, settings and password setup routes explicitly allow pending
accounts. Pending chat access uses the floating panel, not the full /chat route. Missing/NULL server CCS fields preserve legacy behavior; no section
membership is inferred. Initial account-load failure shows retry/logout instead
of mounting protected content. Subsequent stale account checks refresh on window
focus/reconnect (30-second freshness), with an explicit onboarding status refresh.
Live stage/tour widgets wait for confirmed nonpending state; navigation design
and server permissions remain unchanged. Themed onboarding explains current and
post-verification access without promising elevated role or section permissions.
Pending-to-verified transitions cancel old requests, update the same account's
auth/query state, clear stale section state and reset other active/inactive query
caches, including old pending errors. Existing loaders reload profile, membership
and conversations; section/conversation queries pass cancellation signals through
Axios to prevent late pending results overwriting refreshed stores. Own chat
participant hints update while active chat history is preserved. The existing
socket and tokens remain in use: backend dispatch/delivery read stored CCS state,
and newly opened academic chats join through existing hooks. Lost redemption
responses and external verification use the same transition on account refresh.
See docs/CCS_VERIFICATION_PHASE_6.md for historical route/state and cache validation.
Phase 7 deployment/rollback instructions and the infrastructure validation gates
are in `docs/CCS_VERIFICATION_PHASE_7.md`. The complete-sequence migration test
uses a synthetic legacy relational snapshot, not a production clone. Checked-in
deployment compose is development-only; no systemd unit or distributed Socket.IO
manager is supplied. Real production-clone, proxy/storage/OAuth/WebSocket/WebRTC
and load validation remain separate activation prerequisites.
Admin-controlled activation backend and the existing Admin Settings panel are
implemented. The panel shows saved/effective status, modification metadata and
paginated audit history. Only backend-confirmed Super Admin capability exposes
the switch; both directions require ConfirmDialog and current server revision.
No optimistic toggle or automatic write retry occurs; conflicts/permission loss/
uncertain responses refresh status, and account/credential replacement discards
private panel state and late responses. Read requests cancel on replacement.
See `docs/CCS_ADMIN_ACTIVATION_PHASE_3.md` for UI behavior and historical validation.
GET/PUT `/admin/security/ccs-verification` and GET its `/audit` child read/write
the same rollout singleton. Ordinary authorized admins read; writes require an
explicit PostgreSQL `ccs_verification_super_admins` UUID grant plus current active,
unrestricted, nonpending admin status. No grant is seeded and no grant API exists.
Granted users cannot be edited/reset/suspended/deleted through admin user-management
mutations; use operator/self-service security flows. Setting revision, last editor,
database UTC time and a dedicated audit event commit together under row lock;
stale revisions return 409. New `CCS_STUDENT_VERIFICATION_FORCE_OFF` defaults false
and vetoes enrollment only, preserving stored preference and all existing CCS
states. Readiness false with saved ON still returns registration 503 unless the
explicit emergency veto is true. DB toggle changes require no restart; environment
changes require coordinated worker restarts. Apply the additional additive
`20261009_ccs_admin_activation.sql` after the three CCS migrations, before backend
deployment. Use the audited API for subsequent activation/rollback rather than
the older manual SQL toggle. See `docs/CCS_ADMIN_ACTIVATION_PHASE_2.md` for
provisioning and migration instructions.
Final local validation/runbook: `docs/CCS_ADMIN_ACTIVATION_PHASE_4.md`. Historical tests covered
an unchanged all-table fixture snapshot during ON/OFF, fresh worker-process reads,
missing singleton failure, all four additive migrations twice, and code issuance/
redemption after audited disable. Production-clone/live infrastructure validation
and explicit operator Super Admin provisioning remain deployment prerequisites;
local completion does not activate enrollment or establish production migration.
Keep activation OFF pending those checks and explicit activation approval. Apply
`20261009_ccs_verification_foundation.sql`, then
`20261009_ccs_verification_rollout.sql`, then `20261009_ccs_student_codes.sql`
in backend/migrations before deployment;
create_all does not upgrade users or provision the rollout singleton. No production
migration/activation has been established. Audit and operational details:
`docs/CCS_VERIFICATION_PHASE_1.md`, `docs/CCS_VERIFICATION_PHASE_2.md`,
`docs/CCS_VERIFICATION_PHASE_3.md`, `docs/CCS_VERIFICATION_PHASE_4.md`.
At the user's request, six CCS-specific backend test modules, five CCS browser
suites and their two HTML fixtures were removed on 2026-10-10. Historical test
results remain in the audit documents; those suite commands are no longer runnable.
Runtime verification code, migrations, local activation and the explicit
CCS HUB ADMIN grant were preserved. Unrelated regression tests remain.


Keep durable architecture, constraints, verified paths, and commands here. Update
only relevant facts after authorized architectural/contract changes. Avoid secret
values, temporary database paths, machine-specific test fixtures, stale deployment
claims, and chronological conversation logs.

Priority: accuracy and privacy/security, preservation of working functionality,
small safe changes, maintainability, complete delivery, then speed.

Verify -> understand -> modify -> verify.

## 11. Blocked-content moderation

### Mention backend foundation

Posts (including announcement-share captions), comments/replies, messages and
announcements accept optional `mentions` arrays of `{user_id, start, end}`.
Offsets count Unicode code points, not JavaScript UTF-16 units. Plain `@text`
without metadata stays ordinary text; identities are never inferred on reads.
`MentionService` validates spans, current username (or an existing saved label
after a rename), mutual blocks, active/CCS state and destination visibility.
Stored JSONB spans also contain a server-validated label snapshot. Existing
clients may omit the field; edits retain unchanged spans at their existing
offsets and remove changed spans. Mention-aware editors must submit updated
offsets when inserting/removing text before a mention. Clearing sends `[]`.
Announcement content, targets and mentions now commit in one transaction.

`GET /api/v1/mentions/users` supports literal username/first-name/last-name prefix
search, bounded pagination (`limit`, `offset`, `next_offset`), and optional
`post_id`, `conversation_id` or `announcement_id` context. It exposes only public
display fields, filters mutual blocks and applies destination access rules.
Redis limits searches to 120/minute/account, retaining the existing fail-open
Redis-outage convention. The shared editors debounce searches by 250ms.

Apply `backend/migrations/20261015_mentions.sql` manually before backend rollout;
it adds empty JSONB arrays and prefix indexes without converting old text.
No application or production database migration is established. Posts/Comments
UI now uses `features/mentions/MentionTextarea.tsx` around the existing textarea
inputs, with 250ms debouncing, cancellable scoped search, pagination, keyboard
selection and dialog/visual-viewport picker positioning. Editors keep ID spans
through ordinary edits and convert UTF-16 caret positions to Unicode code points.
PostCard/PostContentBody/MarkdownText and comment/reply bodies render saved IDs
as profile links; plain `@text` stays plain. Code and existing markdown links
retain their rendering. Saved post snapshots retain mention metadata. Mention
edit failures keep drafts open; successful edits use server-sanitized text and
metadata without replacing share presentation fields.

ChatWindow now uses the shared MentionInput variant around its native single-line
input in full-page and floating Global, direct, group and academic section chats.
Conversation-scoped search retains membership/block authorization. Pending
accounts keep ordinary text input with autocomplete disabled because the existing
CCS HTTP policy does not authorize their mention-search requests. This phase does
not widen that policy or profile access. Message bodies and attachment captions
render saved target IDs, with contrasting links on outgoing bubbles; quoted reply
previews remain plain to preserve their narrow anonymous-safe contract. Existing
useChat/socketService sends carry mention inputs with replies/media/anonymous
mode. The existing incoming listener retains metadata and unsend clears it.
Draft mentions reset on conversation/account changes; late send acknowledgements
preserve newer drafts.

Mention notifications now stage atomically with content in MentionNotificationService
and push after commit through the existing account-only new_notification event.
Private mention_deliveries rows deduplicate each source/target permanently, including
muted, self and quota-suppressed targets. Real anonymous actors remain private;
public notices use only alias snapshots. Current source audience, mutual blocks,
CCS eligibility and mention preferences filter both delivery and notification
history/unread counts. Limits are 100 notices per actor/hour and 10 per actor/target/hour.
Edits may notify newly selected targets once; remove/re-add does not replay.
Announcement drafts retain valid spans and notify only on permitted publication.
Announcements now use the shared editor and ID rendering too.

Apply backend/migrations/20261015_mention_notifications.sql AFTER the mention
foundation migration and BEFORE deploying notification producers. It adds the
mentions preference and delivery ledger, seeds historical content without alerts,
and is repeatable. No application/production migration or deployment was performed.
Preferences default on; legacy saves preserve the current mention preference.
Validated notification destinations open existing post/comment or conversation/
message views. Optional focus IDs fetch bounded older context under existing read
permissions. The existing socket listener owns mention toast/query updates; no
additional connection or listener exists. Account/request guards discard stale
notification and focused-message responses. Future-dated operator-created
announcements have no new scheduling worker; delivery requires a publish/write path.
See `docs/MENTIONS_PHASE_1.md` for the contract and validation limits.
See `docs/MENTIONS_PHASE_2.md` for UI scope and fixture/browser validation limits.
See `docs/MENTIONS_PHASE_3.md` for chat integration and validation limits.
See `docs/MENTIONS_PHASE_4.md` for delivery, migrations and final validation limits.

`ContentFilterService.check` returns sanitized text using the existing normalized
whole-word/phrase rules. All connected post/comment create/edit and chat write
paths must persist that return value. The content, one automatic UserReport per
submission, private notification and any restriction commit together. Call
`push_warning` only after commit; it uses the existing account-only
`new_notification` event and shared socket toast. Failed pushes retain the notice
in notification history. Raw message bodies are not added to the moderation audit.

The existing admin rule defaults are three violations in a rolling 24 hours and
a 24-hour restriction, with admin configuration preserved. The third submission
is sanitized and published; subsequent submissions retain existing restriction
checks. Anonymous violations use the real account internally. Existing submission
IDs/fingerprints deduplicate retry violations, not message delivery. No frontend
blocked-language submission veto exists.

Tests: `python -m unittest app.tests.test_content_filter -v` from backend. Database
cases require `CONTENT_FILTER_TEST_DATABASE_URL` pointing to a disposable
PostgreSQL database ending in `_test`. The pre-existing content-filter migration
is still required. Apply `backend/migrations/20260928_content_filter_variations.sql`
after it and before deploying variation support; `create_all()` is insufficient.
The additive migration keeps existing rules exact (`detect_variations=false`).

Admin-only `POST /admin/content-filter/words/bulk` accepts comma/newline-separated
`entries` and `detect_variations`, validates each entry, infers Word/Phrase,
skips blanks/duplicates/existing entries and returns counts plus invalid line
reasons. Imports use one insert/commit and the existing rule lock/revision cache.
Spaces remain part of phrases; use Single Add for literal-comma phrases.
Existing entries (including disabled entries) remain unchanged by imports.
The 1,000-entry capacity limit remains; over-capacity imports add nothing.

Explicit grouped variations are stored in `blocked_words.variations` (up to 100
normalized strings per primary rule). Apply the additive
`backend/migrations/20260929_content_filter_groups.sql` before deploying the new
backend. It adds an empty array to every existing rule without merging, deleting,
renaming, or changing activation/matching settings. Existing code continues to
filter existing rules during this migration. Deploy/restart all backend workers
before using the new UI to configure variations; older workers do not load them.
Keep the column if rolling back code. No automatic conversion of existing rules
is performed. Independent existing rules remain independent even if their text
is also explicitly added to another group.

Admin forms edit explicit variations as part of the primary rule; the group's
active status controls all its members. Bulk UI defaults to `grouped=true`:
each line is `primary, variation, variation`. Existing primary rules are skipped,
not merged. Legacy API calls default to separate entries (`grouped=false`).
Omitting variations from legacy updates preserves the saved array. Rule edits
bump the existing revision so every worker reloads. Matching uses the existing
normalization, boundaries, masking, optional obfuscation and single-violation flow.

Variation detection is optional per entry. New UI forms default it on; old API
clients and migrated entries default off. The shared matcher preserves NFKC,
case-folding, phrase whitespace and original-text span masking. Opted-in ASCII
alphabetic tokens of 4-32 letters support selected 0/4/3/1/5/7/8 substitutions,
up to three extra copies per letter run, and single dot/dash/underscore/zero-width
separators between runs. A curated `tangha` alias supports `tanghaa` for `tanga`.
Short words, numeric/mixed tokens and non-ASCII rules keep exact matching.
No substring or fuzzy matching is introduced. All existing write surfaces and
violation/restriction/notification paths remain shared.

## 12. Admin report cases

`GET /admin/reports?grouped=true` presents the existing human UserReport records
as content cases; the default API still returns individual reports. PostgreSQL
groups and paginates cases before batched loading of users, profiles, content,
restrictions and individual allegations. Reporter identities are never returned,
even to this admin view. Automatic blocked-content records remain separate.

Priority counts distinct authenticated reporters on non-dismissed allegations:
0-2 normal, 3-4 medium, 5-9 high, 10+ urgent. Category totals include historical
allegations, including dismissed ones. A case remains pending while any report
is pending; otherwise it is valid if any report is valid, or dismissed. Summary
cards are global case counts (urgent/high count pending cases). Filters combine
at case level without truncating a case's evidence or reporter count. Sorting
always places pending cases first and uses the content key as a stable tie-break.
The existing modal lets admins select an individual report and calls the existing
action endpoints. Multiple validated allegations remain one presented incident;
they never enter the independent automatic content-filter violation count.

Submission uses the existing Redis rate-limit helpers (10 attempts/hour/account),
plus a serialized PostgreSQL fallback (10 saved reports/hour). Transaction-scoped
advisory locks serialize reporter submissions and content escalation, and existing
post/message unique indexes are retained. They continue to prohibit repeat reports
even after dismissal; account reports permit another after dismissal. New active
account reports are also protected by a partial unique content-key index.
Milestones at 3/5/10 notify active admins through existing Notification rows and
the account-only `new_notification` event, once per milestone. Push failure leaves
the notification in history. The admin listing refreshes every 15 seconds.

Signals for review only: five distinct reporters within any ten-minute window;
at least three reporters and half the cohort whose accounts were under seven days
old at reporting; and at least three overlapping reporters across three targets
in the latest 24-hour evidence window. None triggers restriction or hiding.

Deployment: apply `backend/migrations/20260929_report_cases.sql` before deploying
the backend, then update the frontend. This additive migration adds durable
content keys, escalation markers, indexes and an insert trigger; it preserves all
reports and IDs. Known post/message keys survive FK deletion. Legacy records with
already-lost target references stay separate instead of guessing their incident.
Historical milestones are seeded without retrospective notifications. No new
case table or automatic visibility changes are introduced. Keep additive columns
when rolling back code. Migration testing does not mean the app DB was migrated.

Tests: `python -m unittest app.tests.test_report_cases -v`; PostgreSQL cases use
the same disposable `CONTENT_FILTER_TEST_DATABASE_URL` *_test guard and isolated
schemas as the content-filter suite. Redis limits and realtime delivery are mocked
in those fixtures; real database constraints, concurrency and transactions run.

## 13. Mutual social blocking

`blocked_users` retains a directional owner for Settings/unblock, but the shared
`backend/app/services/block_service.py` authorizes social interaction mutually.
It provides EXISTS checks, a SQL blocked-ID union, and canonical pair advisory
transaction locks shared by block/unblock and social writes. No sender or author
identity is accepted from clients for authorization.

DM creation/sends (including replies/media), direct reactions, typing, room entry
and read receipts enforce the relation through the existing ChatService/socket
manager. Old DM history remains readable without marking blocked senders read.
Explicit social group invitations check invitees; automatic academic memberships
and existing shared conversations stay intact. Ordinary Global/section/group
messages remain visible. Nonanonymous directed replies/reactions and stream chat
replies/reactions check the real target; anonymous shared-room interactions retain
public anonymity and are not used to disclose a sender's blocked relationship.

Friend send/accept rejects blocked pairs. Blocking transactionally removes both
friend rows and cancels pending requests; a repeated block also repairs legacy
relationships. Unblocking removes only the owner's row and never restores friends
or requests; an opposite-direction block continues to restrict both users.
Normal profiles and profile post/share listings reject blocked access. Separate
admin moderation and academic records keep their existing authorization.

Social feed/share discovery excludes blocked authors and sharers in SQL; linked
announcement posts remain available under their existing audience checks. Posts
return a `can_interact` UX hint from a batched per-request blocked-author lookup.
Like/react/share and comment create/edit/reply/react/like enforce actual stored
post/comment authors, including parent comments. Post-room entry and delivery
also check blocks so an old subscription cannot leak new social events.

Existing SocketProvider handles a neutral account-only
`social_relationship_changed` cache signal. Shared frontend state refreshes friend,
blocked-list and conversation queries, resets local feed/profile/search/hover
state, closes affected DM/post views and clears typing. Feed/profile requests
started before a relationship change cannot reinsert their old responses. No
additional socket instance, page reload, or block-attempt notification exists.

Apply `backend/migrations/20261005_social_blocks.sql` before backend rollout.
It ensures the existing unique owner/target index, adds the reverse lookup index,
and repairs legacy friendships/pending requests involving saved blocks. A unique
index failure on legacy duplicates fails the transaction safely; reconcile duplicate
block rows before retrying. No columns/tables or historical/academic records are
removed. Restart all backend workers together, then deploy frontend. Migration
verification in an isolated test database does not establish app DB deployment.

Tests: `python -m unittest app.tests.test_social_blocks -v` from backend.
`BLOCK_TEST_DATABASE_URL` must point to disposable PostgreSQL ending in `_test`;
each case owns an isolated schema. The suite covers real service/REST database
behavior, socket-handler invocation with mocked delivery/auth, repeatable migration,
cleanup, duplicate protection and pair-lock concurrency. Live browser/WebSocket
and WebRTC classroom end-to-end verification are separate checks.
