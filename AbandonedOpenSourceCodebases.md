What open source abandoned codebases have more than one million lines of code?
ChatGPT said:

Thought for 2m 56s
Below are five prominent open-source codebases that exceeded one million source-lines-of-code (SLOC) and are now considered abandoned or formally retired. LOC figures come from project self-reports or well-documented third-party counts; “abandoned” means no active development, releases, or governing community today.
Project	Peak SLOC (approx.)	Primary language(s)	Last meaningful activity	Why it’s abandoned
XFree86 (classic X-server)	≈ 17 million 
LWN.net
C	Project effectively ceased 2004; remaining devs migrated to X.org	License dispute and governance conflicts drove contributors away; X.org became de-facto successor, leaving XFree86 “abandoned by nearly all.” 
dwheeler.com
OpenOffice.org “legacy” trunk	≈ 7 – 11 million (7 M per Apache docs; 11 M per Open Hub)
openoffice.apache.org
Datamation
C++, Java	Upstream stopped 2011 when Oracle closed Sun assets; Apache fork sees only sporadic bug-fixes	Oracle donation split community; most developers moved to LibreOffice, leaving original code largely untouched for ~ 9 years.
Reddit
Wikipedia
OpenSolaris core	≈ 10 million 
osnews.com
C, C++, shell	Development halted Aug 2010 after Oracle acquisition	Oracle closed the repository; illumos/OpenIndiana forked, but the original OpenSolaris tree is frozen. 
Wikipedia
Netscape Communicator 4.x (pre-Mozilla)	≈ 30 million 
Amazon
AbeBooks
C, C++	Code discarded 1999–2000 when Mozilla rewrote from scratch	Communicator code proved unmaintainable; Mozilla replaced it, and AOL ended Communicator updates in 2002. 
Wikipedia
Apache Harmony (clean-room Java SE)	≈ 1.3 – 1.5 million
SlideServe
openhub.net
Java, C	Retired to Apache Attic 16 Nov 2011	IBM (key backer) switched to OpenJDK; community voted 20-2 to end the project.
codecentric AG
Wikipedia
Why large abandoned codebases still matter
Security & supply-chain audits – dormant but widely mirrored code can hide unpatched flaws.
Code archaeology & research – they provide real-world mega-scale examples for program-analysis studies (e.g., refactoring tools, bug-density research).
License provenance – portions sometimes resurface in forks or vendor products; knowing original licensing terms avoids legal headaches.
Rule of thumb: once an open-source project clears the million-line mark, its social architecture (governance, contributor on-boarding, funding) must scale as aggressively as its technical architecture—otherwise the maintenance burden eventually outgrows volunteer energy, leading to stagnation and abandonment.
You said:
What about LaTeX or some of the things it depends on?
ChatGPT said:

Thought for 3m 58s
Short answer
None of the core LaTeX tool-chain components hits the one-million-source-lines-of-code (SLOC) mark and is abandoned. They are either (a) far smaller or (b) still actively maintained.
component	current status	SLOC (≈)	comment
Knuth’s TeX engine	feature-frozen but still bug-fixed by Knuth	24 985 lines in tex.web
TeX - LaTeX Stack Exchange
classic literate-Pascal source; intentionally tiny
LaTeX2ε / LaTeX-kernel	active (multiple point releases each year)	≈280 k lines (LaTeX3 repo count)
openhub.net
LaTeX team publishes annual updates
teTeX distribution 3.0	abandoned in 2006	≈194 k lines 
dwheeler.com
last widely-used Unix distro; superseded by TeX Live 
ctan.org
Ghostscript (PostScript/PDF back-end)	active (last commit ≈ 2023)	2 083 762 lines 
openhub.net
does cross the million-line threshold but is not abandoned
Why you don’t see “> 1 M & abandoned” in LaTeX-land
Design philosophy
TeX was written to be small and stable; most functionality lives in separate macro packages (LaTeX classes, TikZ, …) that are each much smaller than a full application.
Modular distributions
Even a huge bundle such as TeX Live has millions of lines in aggregate, but that is a collection of thousands of independent packages that evolve on their own schedules. If any single package is orphaned, it is usually well below the one-million-line bar.
Healthy maintenance culture
Core tools (pdfTeX, LuaTeX, XeTeX, BibTeX, dvips, etc.) still receive fixes through the TeX Live and MiKTeX release cycle; when an older project stalls (teTeX, vteX, …), its place is taken quickly by an actively maintained fork or successor.
Take-away
LaTeX itself: still alive, still under a million lines.
Its historic Unix distribution (teTeX): abandoned, but way under a million lines.
The only LaTeX-adjacent project that is both huge and open source is Ghostscript—but it is very much alive.
So if your hunt is specifically for “open-source, > 1 M LOC, and abandoned,” the TeX/LaTeX ecosystem doesn’t provide a match.
