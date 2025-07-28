Read the project file, that's what this week's project is going to be. Let me know if you have any questions.

Thought for 10 seconds
ChatGPT said:
Got it—I’ve read through the Enterprise Legacy Modernization Project brief. Before we dive in, a few quick questions so I can tailor my help precisely to what you need this week:
Which pathway and legacy codebase are you leaning toward?
e.g., SugarCRM CE vs. SuiteCRM; OpenOffice vs. LibreOffice; Plone vs. a Django 1.x app; etc.
Who’s your target user segment?
Knowing their industry and pain points will shape the modernization plan and the six features we prioritize.
Technology stack preferences or constraints?
For example, frontend framework (React/Vue), preferred cloud provider, database choices, or any internal tooling we must use.
Collaboration & deliverables:
Do you want me mainly for strategy and guidance, or should I also draft code snippets, architecture diagrams, and documentation along the way?
Schedule checkpoints:
Would you like mid-week progress reviews or feedback cycles, or should I just keep pace with the 7-day timeline and surface blockers as they appear?
Let me know, and I’ll jump right in!
You said:
I think that we'll be looking at OpenOffice. As for any other questions, that's going to have to wait until we do more investigation. Where can we find the open source code for OpenOffice, and what are the big open issues/where can we find the big open issues with OpenOffice?
ChatGPT said:

Thought for 1m 54s
Where to grab the code
What	URL / command	Notes
Primary Git repo	git clone https://gitbox.apache.org/repos/asf/openoffice.git	This is the official ASF Gitbox repository and contains the trunk branch used for releases. 
GitHub
Read-only mirror	https://github.com/apache/openoffice	Handy for browsing, pull-requests, and finding forks, but GitHub is not the canonical upstream. 
GitHub
Project “Source code” page	https://openoffice.apache.org/source.html	Brief build instructions plus links to the gitweb “tree” view. 
openoffice.apache.org
Where the open issues live
Tracker	URL	Why it matters
Bugzilla (all bugs & RFEs)	https://bz.apache.org/ooo/	Main defect/feature tracker that the PMC still uses. 
openoffice.apache.org
Bug-vote toplist	https://openoffice.org/qa/iz_votes.html	Community-voted “Top 20” unresolved items—good signal on long-standing pain points. 
OpenOffice
ASF Security Team dashboard	https://whimsy.apache.org/board/minutes/Security_Team.html → “openoffice”	Shows the project’s current security-health color and age of open CVEs.
whimsy.apache.org
Snapshot of the big-ticket open issues (mid-2025)
Theme	Example tickets / evidence	Why it’s important
Security backlog	ASF board lists OpenOffice as Health “amber” with “several issues > 365 days old.” 
whimsy.apache.org
Modern enterprises will not deploy software that lags on CVE fixes. Any modernization effort must prioritise closing or mitigating these.
Macro & link exploits	CVE-2023-47804, CVE-2023-1183 and others still affect installs < 4.1.15. 
Tenable®
Highlights need for a hardened macro engine or sandboxing.
Missing modern-format support	High-vote RFE: native SVG import (issue 2497) and PDF import filter (issue 10384). 
OpenOffice
UI/UX gaps	Tabbed document interface (issue 12686) and dual-monitor presentation support (issue 12719).
OpenOffice
Bi-directional & RTL defects	Directionality lost on export (issue 16295) and Hebrew text ordering bugs (issue 17514).
OpenOffice
Database / driver needs	SQLite SDBC driver request (issue 32117).
OpenOffice
Project sustainability	Wikipedia notes a persistent shortage of active developers and the security backlog leading to the “amber” rating. 
Wikipedia
How to keep tabs on the “big” issues
Create Bugzilla saved searches for:
keyword: security, status: NEW/ASSIGNED/REOPENED
High-vote RFEs (Votes > 50).
Watch the ASF Security minutes (link above) — updated after each monthly board meeting.
Subscribe to the dev@openoffice.apache.org and announce@apache.org lists for release and vulnerability announcements.
Set up CVE feeds filtered for cpe:/a:apache:openoffice to catch newly published vulnerabilities.
Let me know if you’d like help scripting a dashboard or need deeper dives into any of these areas!
You said:
I appreciate security is a big issue, but are there any requested features that are popular?
ChatGPT said:

Thought for 2m 2s
Below is a quick-scan of the most-voted feature requests in the OpenOffice Bugzilla. (The project still surfaces its “Top 20” via the public Issue votes overview page. Although the footer shows an old file-timestamp, the vote counters themselves continue to reflect current totals, so it remains a useful popularity signal.)
Rank ▲	Votes	Issue ID	Requested feature (abridged)	Component / theme
1	300	2497	Native import of SVG graphics	File-format / Draw
2	133	30631	Right-to-left (RTL) enabled form controls	UI / Localization
3	120	12686	Tabbed document windows (Firefox-style tabs for each open file)	UX
4	119	12719	Dual-monitor presentation support	Impress / UX
5	118	972	Align math-formula baselines with surrounding text	Writer / Typography
6	85	32117	SQLite SDBC driver (native Base connectivity)	Database
7	84	49991	Embed SVG graphics inside any document (not just link)	File-format / Draw
8	82	18024	Better handling of weak bidi characters	I18N
9	78	8949	Queries across multiple tables in DBase/Calc back-ends	Base
10	76	10384	PDF import filter (edit existing PDFs)	Filters / Interop
Other highly voted ideas worth noting
Anti-aliased drawing objects (Issue 28526 – 75 votes) – smoother vectors. 
OpenOffice
Sticky “Notes” UI overhaul (Issue 6193 – 64 votes). 
OpenOffice
“Reveal Formatting” pane (Issue 3395 – 63 votes). 
OpenOffice
Cross-references to headings without macros (Issue 25072 – 58 votes). 
OpenOffice
Where to watch or dig deeper
Bugzilla “votes” dashboard – the page we drew from:
https://openoffice.org/qa/iz_votes.html → shows live vote tallies.
Saved Bugzilla search: votes > 50, status = NEW|STARTED|REOPENED, type = ENHANC|FEATUR – keeps the focus on popular feature work.
dev@openoffice mailing list – proposals sometimes surface here before they get a formal RFE ticket.
Extensions site – occasionally a volunteer ships a workaround (e.g., an SVG import extension) that can inform a modernization roadmap.
Let me know if you’d like any of these RFEs unpacked in more depth, or help drafting a feasibility/effort matrix for the top items!
You said:
What is the current state of LaTeX integration with OpenOffice? What efforts have been made towards this, if any?
ChatGPT said:

Thought for 4m 39s
TL;DR
Apache OpenOffice still has no native LaTeX interpreter—everything relies on third-party add-ons. In 2025 the only options that are actively maintained are TexMaths (for inserting equations) and Writer2LaTeX (for exporting whole documents). Older projects such as OOoLatex are effectively abandoned, and the long-standing core RFE to accept TeX syntax in Math remains open.
1 What ships in the box?
OpenOffice Math (the built-in formula editor) understands its own “StarMath” markup, not TeX. You can type
sqrt { a^2 + b^2 }
but you cannot paste \sqrt{a^{2}+b^{2}} and have it recognised. 
OpenOffice
There is no LaTeX export filter in Writer, Calc or Impress out-of-the-box.
2 Community add-ons and their current health
Extension	Purpose	Last real update	Status with AOO 4.1.15
TexMaths	Insert TeX maths as PNG/SVG images; stores the TeX in the image’s attributes for re-edit	0.52.3 (26 Oct 2024)
roland65.free.fr
Works; PNG fine, SVG has minor rendering glitches on Windows; needs an external TeXLive/MiKTeX install. The author notes it “can also be used in Apache OpenOffice 4.x”. 
roland65.free.fr
Writer2LaTeX	Export Writer (and Calc) documents to .tex, .bib, EPUB, HTML+MathML	Extension catalogue refreshed 14 Jul 2025 – still offered for AOO 4.x
SourceForge
Functional. Upstream project continues on SourceForge/GitHub; the filter preserves most paragraph/character styles. 
extensions.openoffice.org
Calc2LaTeX	Convert Calc ranges → tabular environments	Occasional fixes, but essentially “mature”.
extensions.openoffice.org
OOoLatex	Historical macro set that TexMaths was forked from	last release 2007, project page now warns “no more maintained—use TexMaths”.
extensions.openoffice.org
SourceForge
3 Upstream work that hasn’t happened
Core RFE still open – Issue 2864 “TeX style formula input would be welcomed” was filed in 2002 and is still in the NEW state; no patches have landed. 
bz.apache.org
No signs in the dev@ mailing list or recent Git commits that anyone plans to embed a TeX engine or a MathML-to-LaTeX bridge in the trunk.
4 What this means in practice
Equation authoring
For routine writing: install TexMaths, point it at your LaTeX tool-chain, and you get WYSIWYG TeX formulas inside Writer, Calc or Impress.
For macro security–conscious environments: TexMaths shells out to latex/dvisvgm/dvipng; if that is prohibited, you’re stuck with StarMath.
Round-tripping documents
Writer → LaTeX: Writer2LaTeX is the only maintained path. Expect a fairly clean .tex, but heavily style-dependent.
LaTeX → Writer: there is no maintained importer. Most users paste finished equations via TexMaths and manage the narrative in Writer.
Long-term maintainability
Apache OpenOffice has a small developer pool; both TexMaths and Writer2LaTeX are hosted and driven outside the AOO PMC. If you need LaTeX workflows for an enterprise-modernisation project, you would likely bundle TexMaths and Writer2LaTeX in your installer and contribute fixes upstream rather than wait for core integration.
Quick recommendations for the Legacy-Modernisation week
Bundle TexMaths + Writer2LaTeX in your prototype and script their installation to give business users an “out-of-the-box” TeX path.
If equation fidelity in SVG is a blocker, patch TexMaths’ SVG post-processing for AOO’s older rendering library or fall back to PNG.
Keep an eye on LibreOffice’s native math/LaTeX work; back-porting pieces may be faster than pushing new code through Apache’s review pipeline.
Hope that clarifies where things stand—let me know if you need setup scripts, packaging pointers, or a deeper dive into TexMaths’ architecture.
You said:
I'm kind of interested in this lack of maintained LaTeX to Writer importer. So, are the un-maintained importers?
ChatGPT said:

Thought for 5m 55s
Short answer
Yes – a handful of projects once tried to bring LaTeX → Writer import to OpenOffice, but every single one is now abandoned or frozen, and none of them handles a full modern document without lots of manual clean-up.
What existed (and why it fizzled)
Project (year of last real release)	What it could do	Current state	Why it stalled
OOoLaTeX (2007)	Let you paste a LaTeX snippet and get a PNG/EMF formula in Writer/Impress. Not whole documents.	Extension page now shows a “not updated recently” warning; last beta is from 2007.
extensions.openoffice.org
One-person macro set; never handled sectioning, figures, BibTeX, etc.
Writer4LaTeX (part of the Writer2LaTeX family, c. 2010)	Ambitious plan to make Writer a frontend to LaTeX (compile & preview from inside Writer).	No releases since v1.0 in 2010; the current roadmap only mentions a future “TeX import filter (using pandoc).” 
Writer2LaTeX
Developer moved on to export-only work; import never left prototype stage.
latex2format / latex2odt (Ruby CLI, 2007)	Batch-convert .tex → ODT/Doc via TeX4ht + headless OpenOffice.	Repository archived May 2024; author calls it “outdated – use pandoc.” 
GitHub
Script broke each time TeX4ht or AOO changed; no maintainer.
mk4ht oolatex / make4ht -f odt (TeX4ht pipeline)	Produces an ODT that Writer can open. External converter – not an AOO extension.	TeX4ht itself is alive, and the ODT backend still works, but there’s zero OpenOffice-side integration or UX. 
kodymirus.cz
Needs command-line, tweaks for packages/images, and post-import cleanup.
Nothing newer than these has shown up on the dev@openoffice list or Bugzilla.
Practical options in 2025
Need	What actually works today
One-off “import” of a LaTeX paper	Use TeX4ht or pandoc to generate ODT, then tidy styles in Writer. Expect to re-layout equations and floats.
Round-trip editing with LaTeX users	Keep the LaTeX canonical; share PDFs for review, or move the collaboration to LibreOffice (better MathML, still no importer but more active devs).
Live LaTeX snippets inside Writer	Ship TexMaths (maintained through 2024) for equations; everything else stays native Writer.
Enterprise modernization project	Budget time either to (a) embed pandoc + TexMaths in an installer and write glue macros, or (b) sponsor development of the long-promised Writer2LaTeX import filter.
Why nobody maintains an importer
Huge feature gap – LaTeX macros, custom classes, TikZ, etc. are a moving target; a one-time parser breaks quickly.
Small AOO dev pool – only ~10 active committers cover the whole suite.
LibreOffice captured the academic audience – volunteers hacking on TeX bridges mostly work there now.
If your modernization week really needs LaTeX input, the realistic path is to wrap an external converter (pandoc or TeX4ht), then massage the resulting ODT with UNO macros or XSLT – faster than reviving a decade-old extension.
Happy to sketch that workflow or dig into any of the dormant codebases if you’d like!
You said:
It's not that I need LaTeX input, it's that the project is very open ended and getting such an importer together might be a good fit for the sort of feature I'm trying to add.
ChatGPT said:

Thought for 2m 56s
Below is a practical-engineering view of how you could resurrect (and finally maintain) a LaTeX → Writer importer that Apache OpenOffice has lacked for two decades.
1 What “import” really needs to do
Content type	Minimum viable behaviour	“Nice-to-have”
Text & structure (\section, lists, tables)	Map to paragraph / list / table styles in ODF.	Preserve custom class / package metadata as user-fields or styles.
Math	Render TeX to PNG/SVG and insert; keep the original TeX string for round-trip edits.	Convert MathML → StarMath so equations stay editable inside Writer.
Images / figures	Copy linked graphics into the package and keep captions.	Honour relative paths, TikZ output, sub-files.
Bibliography & refs	Flatten citations as plain text.	Drive BibTeX/Biber, map cite commands to Writer fields.
Aim for MVP-quality import covering 80 % of theses and articles; power users can tidy the rest manually.
2 Leverage what is still alive
Tool	How it helps	Proof it’s alive
Pandoc	Reads large LaTeX subset → ODT in one call & has Lua-filter hooks for tweaking styles.	Latest 3.7.0.2 shipped May 28 2025.
Pandoc
TeX4ht / make4ht	Converts full LaTeX (+ packages) → ODT; emits MathML.	make4ht v0.4d dated Apr 14 2025 lists ODT among “supported formats”. 
texdoc.org
Writer2LaTeX	Already an AOO/LO Java filter; roadmap explicitly plans “Add TeX import filter (using pandoc)” – ready partner to co-develop with.
writer2latex.sourceforge.net
TexMaths	Maintained OXT that turns TeX snippets → SVG/PNG and back-edits them; can be reused for equation post-processing. (No citation needed – you already know this.)	
UNO ImportFilter API	Official way to register “LaTeX” in the File ▸ Open dialog; gives you XFilter/XImporter hooks.	See SDK docs for the com.sun.star.document.ImportFilter service. 
openoffice.org
3 Architectural blueprint
┌────────────┐    ┌───────────┐    ┌───────────────┐
│ .tex file  │──▶│ Converter  │──▶│  .odt temp     │
└────────────┘    │ (pandoc   │    │  (pandoc or    │
                  │  or make4ht)│   │   make4ht)    │
                  └───────────┘    └──────┬────────┘
                                           │load via
                        UNO XImporter ▼    │UNO API
                               ┌───────────┴────────┐
                               │   Writer document  │
                               └────────────────────┘
Post-filter: run TexMaths over embedded TeX → SVG, convert MathML → StarMath, apply style XSLT.
Phase 1 (1-week proof-of-concept)
Java UNO component that implements ImportFilter and shells out to pandoc (pandoc -f latex -t odt --lua-filter fix-styles.lua).
Register MIME text/x-tex in manifest.xml so Writer’s File ▸ Open lists “LaTeX Document”.
Bundle everything in an OXT with a minimal UI (“Import LaTeX…”).
Phase 2 (hardening)
Add fallback to make4ht when pandoc bombs on exotic macros.
Pipe output ODT through an XSLT/Lua-filter to align styles with the enterprise template.
Call TexMaths in headless mode to replace raster formulae with SVG (or StarMath if you build a MathML→StarMath bridge).
Phase 3 (nice-to-have)
BibTeX/Biber driver: run bibliography, map \cite to Writer fields.
Progress dialog & error reporting inside Writer.
Contribute code upstream to Writer2LaTeX so both suites share one importer.
4 Implementation guideposts
Task	Key pointers
Create the filter component	Follow SDK DevGuide “OfficeDev › Filtering Process” and FilterFactory examples. Your class must expose XFilter, XImporter, optionally XInitialization.
Sandbox external calls	Use ProcessBuilder with a user-config path to TeX Live / pandoc. Fail fast if binaries missing.
Equation pipeline	Pandoc outputs MathML inside ODT → read the XML DOM → convert to StarMath (starmath.dtd) or delegate to TexMaths for SVG.
Styles mapping	Pandoc can emit custom styles via a Lua filter (e.g., map \textbf to Strong_Emphasis). Easier than post-XSLT.
Packaging	ant + SDK’s OOoExtensionTarget.jar or gradle-oo-plugin to produce the .oxt. Sign it for corporate deployments.
5 Resourcing & timeline (indicative)
Week	Deliverable
0.5	Feasibility spike: convert sample thesis with pandoc & make4ht; capture gaps.
1	Prototype .oxt with ImportFilter → pandoc backend, simple dialog.
2–3	Style/XSLT cleanup, TexMaths hook, automated tests (LaTeX → Writer → PDF diff).
4	Beta with dual back-ends (pandoc / make4ht), signed installer, docs.
5+	Optional StarMath conversion & bibliography pipeline; contribute patches to Writer2LaTeX branch.
A single developer who already knows Java/UNO can usually get the MVP loading documents inside ~40 person-hours; equation polishing and bibliography handling roughly doubles that.
6 Why this is a good Legacy-Week feature
High visible impact – instantly opens decades of scientific content in OpenOffice.
Aligns with upstream interest (Writer2LaTeX roadmap) so your work has a home.
Mostly glue work – leverages mature converters, letting you focus on integration and UX, not reinventing TeX parsing.
Clear extension scope – fits the “add six strategic features” brief without touching core C++ code.
Next steps
Pick a sample corpus (simple article, Beamer slides, thesis) and run both converters to decide which backend is your MVP default.
Spin up the UNO SDK example “HelloFilter” and stub in the external call.
Ping the Writer2LaTeX maintainer on SourceForge about collaboration—they’re already planning a pandoc-based import filter.
Let me know if you’d like starter code, build scripts, or a deeper dive into the equation-to-StarMath path!
