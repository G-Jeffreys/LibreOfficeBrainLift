Alright, the refactoring of the plan is well under way. LibreOffice has been built locally according to your instructions, and, pleasantly, I have found out that Document level tabbing is not actually in LibreOffice. I have added this markdown document which my team mates mocked up describing the current plan. Take a look and give me any feedback you have. Keep in mind that this must fit the G2P6 Enterprise Week reqs.

Thought for 57 seconds
ChatGPT said:
Here’s a quick “peer-review” of the proposal with an eye on the G2P6 Enterprise Week rubric and a couple of lessons learned from hacking on LibreOffice’s codebase this week.
1 — Rubric Alignment
G2P6 requirement	How the current draft stacks up	Action you should take
Exactly six meaningful new features【turn1file9†L51-L55】	Draft lists three big features—Drive/Dropbox, Slack notifications, document-level tabs【turn1file0†L9-L11】.	Either expand to six features or cleanly split the existing mega-features into six self-contained deliverables (see Section 2).
Feature functionality + integration (50 pts)	All three ideas are valuable but broad. Risk: “wide but shallow” implementations.	Define an MVP cut per feature that can truly ship by Day 7 and demonstrate integration (e.g., tab bar only on Writer first).
Technical quality (20 pts)	Good emphasis on OAuth2, VCL, CMIS refactor. Need explicit mention of testing, security, and rollback paths.	Add short “Done = ✅” acceptance criteria for each feature (unit/integration tests, cross-platform parity).
AI-assisted dev documentation (10 pts)	Not addressed yet.	Start a running log of ChatGPT/Copilot prompts and decisions—include it in the repo wiki.
2 — Getting to “Six Features”
Below is a workable split that still feels coherent:
Google Drive Connector (reuse & harden CMIS layer).
Dropbox Connector (new REST implementation).
Slack Document-Lock Notifications (webhook MVP).
Single-Sign-On / Enterprise OAuth (share tokens across connectors & Slack).
Document-Level Tabbing (Writer-first) – experimental flag.
Session-Restore & Quick-Switch Palette (opens last tab set, Cmd/Ctrl + P to fuzzy-search open docs).
Why this helps:
Each delivers obvious user value and can be demoed independently.
You’ll satisfy the “exactly six” rule without inventing unrelated fluff.
Features 4 & 6 are small but high-impact, perfect for the tight 7-day timeline.
3 — Feature-Specific Notes
A. Drive / Dropbox connectors
Reuse before rewrite: LibreOffice already mounts remote files via ucb and CMIS; stabilise that path for Drive first, then layer the Dropbox backend to the same interface.【turn1file0†L44-L47】
Offline flow: clarify whether background sync is queue-based (good) or immediate; avoids blocking UI threads.
B. Slack notifications
Hook into the SfxMedium::LockFile() lifecycle—this fires for every “open-for-edit”.
Keep first cut simple: one workspace, one channel, JSON payload like {"user":"John","doc":"Q3_Plan.ods","status":"locked"}.
C. Document-level tabbing
Scope risk: cross-platform VCL plumbing is hard. Prototype only on GTK first (GtkNotebook) and hide behind --enable-experimental-features flag【turn1file3†L60-L66】.
MVP checklist: create tab, close tab, dirty-state asterisk, Ctrl+Tab switch.
4 — Timeline Fit (7 days)
Day	Deliverable
1–2	Harden Drive CMIS, spike Dropbox auth flow.
3	MVP Slack webhook + token-sharing (features 1–4 share OAuth code).
4	GTK tab bar skeleton + session-restore JSON cache.
5	Polish Dropbox; quick-switch palette via UNO dispatch.
6	Cross-platform smoke tests, security scan, AI-prompt doc write-up.
7	Demo script + rollback plan, final bug bash.
5 — Quick Wins for Graders
Include unit tests that assert CMIS URL parsing, Dropbox REST error handling, and VCL tab focus events.
Add a docs/ai-prompts.md capturing the exact ChatGPT/Copilot interactions—this directly scores rubric section 4.
Record a 2-minute screen-capture showing all six features in action; drop the link in README.
TL;DR
Solid direction—just break the three umbrella items into six shippable slices, trim scope to an MVP per slice, and start logging your AI usage. Hit those, and you’ll check every G2P6 box while still delivering high-impact modernization for LibreOffice. Good luck!
You said:
Okay, architecturally, at what points will we be interacting with/modifying LibreOffice?
ChatGPT said:

Thought for 1m 4s
Below is a “map of attack” that shows where in the LibreOffice stack each of the six features will plug in or require code changes. I’ve grouped the touch-points by the main architectural layers LibreOffice exposes so you and your teammates can divvy up ownership quickly.
1 — Remote-Storage Connectors
(Google Drive & Dropbox)
Layer	Concrete code-paths you’ll touch	Why
UCB (Universal Content Broker)	ucb/source/ucp/cmis/ refactor for Drive; brand-new ucb/source/ucp/dropbox/ provider	Drive can largely piggy-back on the CMIS provider once you add the OAuth header injection; Dropbox needs a fresh UCP that wraps its REST endpoints
OAuth Token Service (new)	svl/source/misc/tokenmanager.cxx (new), plus platform keychain helpers in vcl/*	A central UNO service so the Drive, Dropbox and Slack modules can share refreshed tokens
File-picker UX	svtools/source/control/filectrl.* and cui/source/dialogs/	Add cloud panes & sync status decals to the open/save dialog
Background sync / conflict UI	ucbhelper/transfer.cxx, autosave timer hooks in sfx2/source/doc/	Queue uploads so the UI thread never blocks; pop a non-modal conflict dialog when we detect version drift
2 — Slack Lock-State Notifications
Layer	Code-paths	Why
Document lock lifecycle	sfx2/source/doc/filelock.cxx / SfxMedium::LockFile()	Emit a UNO-broadcast LockChanged signal right where LibreOffice actually creates or removes the .~lock file
Slack webhook client (new)	connectivity/slack/SlackClient.cxx (small libcurl wrapper)	Posts JSON payloads and retries w/ exponential back-off
Per-user settings	officecfg/registry/schema/org/openoffice/Office/Slack.xcs + a tab in Tools → Options	Toggle notifications and map LibreOffice user ↔ Slack user
3 — Enterprise-wide OAuth SSO
Mostly glue: The token service above surfaces a com.company.auth.OAuthTokenProvider UNO interface that every connector and the Slack module consume. You’ll register this service in services.rdb and package it in scp2/ so it’s available early during application startup.
4 — Document-Level Tabbing UI
Layer	Code-paths	Why
VCL widgets	vcl/source/control/ → new tabbar.cxx/hxx • abstracts GtkNotebook, NSTabView, Win32 tab	Cross-platform tab strip core
Frame overhaul	framework/source/fwe/frame/Frame.cxx & sfx2/source/window/frame2.cxx	Allow multiple SfxViewShells per top-level window; route Dispatch to the active tab
Keyboard & context-menu handlers	sfx2/source/appl/appcmd.cxx	Ctrl + Tab cycling, “Move to new window”, etc.
5 — Session-Restore + Quick-Switch Palette
Layer	Code-paths	Why
Session manager	desktop/source/app/app.cxx writes session.json on quit; reads it on start	Persists the current tab set so users land back where they left off
Palette UI (Fuzzy switcher)	framework/source/uielement/ new QuickSwitcher.cxx • invoked via Ctrl + P	Lists open documents (tabs) and recently closed ones; uses UNO PopUp component
6 — Shared Test & Build Infrastructure
Add cppunit tests under each module to assert:
CMIS URL mapping for Drive/Dropbox
Slack client retry logic
Tab-bar focus & dirty-state handling
Update Repository.mk and configure.ac to toggle each feature behind --enable-g2p6-experimental, matching the Enterprise-Week rubric for incremental delivery.
How this meets G2P6 Enterprise Week expectations
You now have six clearly bounded features, each mapped to explicit integration points the graders can inspect.
The plan keeps risky work (tabbing, Dropbox REST client) behind experimental flags, so the MVPs stay “shippable in 7 days.”
Centralizing OAuth ticks the “modern authentication” rubric line, while the Slack hook nails “real-time notifications.”
Use this matrix when you open T-shirts in Phabricator: it tells reviewers exactly which LibreOffice subsystem you’re touching and why.
You said:
Develop an architectural plan and a PRD for this project, and have them prepared as .md files that I can download.
ChatGPT said:

Thought for 20 seconds
The documents are ready!
Architectural Plan: Download
Product Requirements Document (PRD): Download
Feel free to review and let me know if you’d like any revisions or deeper dives into specific sections. 
You said:
For each feature, write out a task-and-subtask based blueprint for implementing the feature. Output these blueprints each as their own .md file for me to download.
ChatGPT said:

Thought for 24 seconds
Your implementation blueprints are ready! Download each Markdown file below:
Google Drive Connector – Download
Dropbox Connector – Download
Slack Lock Notifications – Download
Enterprise OAuth SSO – Download
Document-Level Tabs – Download
Session Restore & Quick Switcher – Download
Let me know if you’d like adjustments, additional milestones, or resource assignments! 
