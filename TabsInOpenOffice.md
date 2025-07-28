I was looking at OpenOffice's most voted for features, and one of the biggest ones is browser/Firefox style tabs in-app for documents.

What would go in to making this change/update?
ChatGPT said:
Adding browser-style tabs to Apache OpenOffice is a significant architectural change, but it’s also a highly visible and popular UX enhancement that aligns well with modernization goals. Here's what would go into making it happen:
🔧 1. Understanding the Existing Architecture
OpenOffice has a monolithic, event-driven C++ architecture with a custom windowing system built on the VCL (Visual Class Library). Each document (Writer, Calc, Impress, etc.) typically opens in its own top-level window.
To implement tabs:
You'll need to refactor how documents are instantiated and displayed within the main application window.
VCL must be extended or modified to support a container for tabbed child windows inside a single parent frame.
🧱 2. UI Layer Modifications
Implement a tab bar widget (either custom or adapted from an existing toolkit).
Modify the main application shell to:
Replace the MDI/SDI window management with a tabbed document interface.
Ensure tabs are dynamically created, labeled, focused, and destroyed based on document state.
🔄 3. Document Lifecycle Management
Create an abstraction that allows documents to share a single top-level frame but maintain independent state.
Modify the logic for:
New/open/close/save operations to route to the appropriate tab.
Tab switching to handle document activation and redraw.
Refactor inter-document communication and event handling.
🪛 4. State & Session Handling
Ensure tab state (active tab, document paths, unsaved changes) is persisted between sessions if needed.
Optional: support drag-to-reorder tabs, detachable tabs (open in new window), or grouped tab behavior by app (Writer tabs, Calc tabs).
🚨 5. Challenges & Risks
High coupling in the OpenOffice codebase makes UI refactors tricky.
Thread safety and resource cleanup need rigorous testing—e.g., what happens if a tab crashes?
Support for multiple application types (Writer, Calc, Draw, etc.) in a shared tab bar may require complex logic or separate containers per app.
✅ 6. Why It’s a Good Feature for This Project
This fits perfectly within the “Six New Features” criteria in the modernization project:
High user impact (requested by many users).
Modern UX expectation (standard in most apps today).
Touches core UI/UX, showcasing understanding of legacy architecture.
🔩 Tools That Will Help
Use AI tools like Cursor or Claude to explore the VCL and application shell.
Target a single application (e.g., Writer) first for tab integration before generalizing.
Would you like a step-by-step technical roadmap for adding tabs to Apache OpenOffice Writer specifically?
You said:
Sure, let's explore this in more detail
ChatGPT said:
Great! Here's a step-by-step technical roadmap for implementing browser-style tabs in Apache OpenOffice Writer. This plan isolates changes to one application (Writer) to keep scope manageable while laying groundwork for broader adoption across the suite.
🚧 Phase 1: Deep Dive into the Existing Architecture
🔍 1. Analyze VCL and Framework Code
Goal: Understand how OpenOffice manages windows and UI.
Key locations:
vcl/source/window/* – VCL widget and windowing system
sw/source/ui/* – Writer's UI code
framework/source/* – Application shell, controller logic
sfx2/source/* – Document and view management
📦 2. Map Writer Document Lifecycle
Identify how Writer documents are:
Loaded (SwDocShell, SwView)
Displayed (frame creation, window hierarchy)
Closed and saved
🧱 Phase 2: Build Tab Container UI in the Shell
🧩 3. Implement a Tab Bar Widget
Either:
Reuse any existing tab implementation in VCL (TabControl?), or
Build a custom tab bar widget within vcl.
Embed it in the main application frame (framework/source/uifactory/uielement.cxx or similar).
🖼 4. Modify the Application Shell
Replace or extend the Writer Frame logic to:
Render a tab bar at the top
Host multiple Writer views within a single frame
Tab switching must:
Hide/show document windows
Trigger focus events, redraw, and update menus/toolbars
🔄 Phase 3: Wire Up Tab Management
🧠 5. Intercept Document Open/Create Commands
Modify SfxApplication::OpenDoc() or SfxObjectShell::DoInitNew() to:
Route new documents to open in a tab, not a new window
Generate a new TabEntry with document metadata (title, path, modified state)
🗃 6. Implement Tab Lifecycle
On tab close: trigger normal document close logic (CanClose, SaveModified, etc.)
On tab rename: update label on file save/as
On drag-to-reorder (optional): manage tab index list
⚙️ Phase 4: Enhance Usability and Stability
🔄 7. Preserve Session State (Optional)
Store tab order, open documents, and window size/location.
Reload session on restart (config stored in user/registry or via .xcu files).
📋 8. Address Edge Cases
Unsaved changes across multiple tabs
Switching between different document types (limit to Writer at first)
Accessibility, theming, localization
🧪 Phase 5: Testing and Iteration
✅ 9. Manual and Automated Testing
Use the QA test framework to:
Open/close dozens of tabs
Check stability, memory usage, and redraw correctness
Ensure no regressions in existing workflows
🎨 10. Visual Polish and User Options
Theme tab appearance to match the OpenOffice UI
Optional: add settings for “Open in tab” vs “Open in new window”
🧰 Bonus: Suggested AI-Driven Tasks
Task	Tool	Prompt Example
VCL exploration	Cursor	"Where is TabControl defined in VCL?"
Writer shell refactor	Claude Code	"Refactor SwView creation to target tab container, not window"
Tab bar layout prototype	Copilot / ChatGPT	"Create a basic VCL tab bar layout with close buttons and tab switching"
📦 Deliverable Suggestions for Legacy Week
To show off this feature in your project:
Record before/after demo of document management
Include screenshot of 3+ documents open in tabs
Write a brief devlog explaining the shell changes and document routing
Mention this feature as a “modern UX enhancement aligned with browser-native patterns”
Would you like example code snippets or AI prompt templates for some of the more complex parts like document switching or tab creation?
You said:
Actually, I would like for you to write up a document outlining the architecture of this project (unless you are saying Phase 1 is when we will figure that out)
ChatGPT said:
Yes — Phase 1 is where you’ll uncover most of the architecture through exploration, since OpenOffice doesn't have clean, modern documentation. But to help accelerate that, here’s a working architecture document draft outlining the structure of your tabbed interface modernization project for Apache OpenOffice Writer:
🧭 Project Architecture: Tabbed Interface for Apache OpenOffice Writer
📌 Overview
This document outlines the proposed architectural design for integrating browser-style tabs into Apache OpenOffice Writer. The goal is to enable multiple documents to be opened in a single application window via a tabbed user interface, improving user experience and aligning with modern UI standards.
🏗️ High-Level Architecture
🖥 Existing System Overview
Language: C++
UI Framework: VCL (Visual Class Library)
Document Shell: SwDocShell
View Controller: SwView
Window Management: SDI/MDI-style frames managed via SfxFrame, SfxViewFrame, and SfxObjectShell
🧩 Proposed Enhancement
Replace or extend the default SDI behavior to:
Embed multiple SwView instances in a tabbed container.
Manage a new component called TabManager that handles UI logic, routing, focus, and lifecycle.
🧱 Key Components
1. TabManager (New Component)
Responsibility: Controls creation, switching, and destruction of tabs.
Location: Lives within the framework or sfx2 module.
Interfaces:
AddTab(SwDocShell*)
CloseTab(int tabIndex)
SwitchTab(int tabIndex)
UpdateTabTitle(SwDocShell*)
2. TabBar UI (New Widget)
Based on: TabControl in VCL or custom VCL component.
Integrates with:
Menu/Toolbar updates via focus context change
Window title bar and status bar via events
3. Modified Application Shell
Host: Main frame window becomes a tab container.
Changes to: SfxFrame, SfxViewFrame, and/or SwApp::Main()
Impact: Prevents new top-level windows unless explicitly requested (e.g., popouts)
4. Modified Document View Layer
SwView: Renders document content per tab
SwDocShell: Stores document state and metadata
Behavior:
Each tab corresponds to a SwView instance embedded within the same parent window
Switching tabs triggers redraw and context event propagation
🔁 Document Lifecycle
Action	Old Behavior	New Behavior (Tabbed)
New Document	Opens in new window	Creates a new tab with new SwDocShell
Open File	Opens in new window	Opens in new tab
Close Document	Closes window	Closes tab, prompts save
Save/As	Updates file + window title	Updates file + tab label
Switch Focus	Window-level focus switch	Tab switch triggers SwView::Activate()
🔄 Event and Focus Handling
SwView::Activate() and SwView::Deactivate() are triggered by tab switching.
Menu and toolbar state must respond to active SwView instance.
Global accelerator keys must be updated to reference focused tab context.
🗃 Persistence Layer
Optional feature to persist open tabs on close:
Store in user config (registrymodifications.xcu)
Restore tab layout on next launch
🧪 Testing Plan
Open/close 10+ tabs, test memory stability
Ensure Ctrl+Tab and Ctrl+W shortcuts function correctly
Check file operations per tab (save, autosave, close with unsaved changes)
Switch rapidly between tabs with large documents to test rendering behavior
📦 Deployment Strategy
Target a build with tabs enabled only for Writer initially
Add UI preference toggle for “Enable Tabbed Documents”
If stable, generalize to other modules (Calc, Impress) in later phases
📚 Dependencies and Risks
Area	Risk	Mitigation
VCL Complexity	VCL has limited documentation and is highly custom	Use AI-assisted code exploration (Cursor, Claude)
Event Routing	Focus and UI updates may break	Wrap SwView logic with TabManager hooks
Regression Bugs	Legacy code might expect 1:1 window-doc model	Start with isolated Writer changes and thorough testing
✅ Goals of This Architecture
Preserve core business logic of Writer
Improve user experience without breaking existing workflows
Build a foundation for cross-document workflows and cloud UX patterns
Would you like me to create this as a downloadable document (Markdown, PDF, or Word)? Or should I load it into the canvas for collaborative edits?
You said:
Yes, please put it as a downloadable Markdown file
ChatGPT said:
Your architecture document is ready for download:
📄 OpenOffice_Tabbed_UI_Architecture.md
Let me know if you'd like a PDF version or if you want to iterate further on any section. 
You said:
No, that looks good. Please spin up a PRD.md file as well
ChatGPT said:
Your Product Requirements Document (PRD) is ready for download:
📄 PRD_Tabbed_UI_OpenOffice.md
Let me know if you want it converted to PDF or expanded for broader OpenOffice suite integration. 
