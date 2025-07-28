Trying to build OpenOffice on my macOS 11 Silicon chip Macbook Air has been a nightmare. We will be pivoting to LibreOffice (even though this is defintely going to mean throwing out the tabs feature since LibreOffice already has tabs as I have been told). What are the next steps here?

Thought for 2m 49s
ChatGPT said:
Below is a pragmatic “pivot plan” that has worked for other teams that moved from Apache OpenOffice to LibreOffice on Apple-silicon Macs. Follow the order-of-operations and you should be running (and hacking on) LibreOffice quickly—even on an M1/M2/M3 MacBook Air running macOS 11 “Big Sur”.
1 — Decide whether you really need to build from source
Pre-built universal DMGs already exist for Apple-silicon (“macOS (Apple Silicon)” on the download page). If you only need LibreOffice itself and can postpone code-level customisation, grab the 25.2 series DMG and make sure everything works on your hardware first 
LibreOffice
.
Build from source only if you are:
adding/patching C++ code,
integrating LibreOffice into another application,
or producing a re-branded/licensed distribution.
If you do need a source build, continue with the steps below.
2 — Set up the macOS build tool-chain
Xcode & command-line tools – install from the App Store, launch once so that the licence is accepted 
The Document Foundation Wiki
.
Homebrew dependencies –
brew install autoconf automake libtool gperf gawk pkg-config ccache \
             bison flex coreutils
(Optional) Java: If you care about Base or macros that need a JVM, install a recent JDK; otherwise you can pass --without-java later 
The Document Foundation Wiki
.
3 — Bootstrap the source tree with LODE
The LibreOffice Development Environment (LODE) script installs the right SDKs and checks out the code for you:
git clone https://git.libreoffice.org/lode $HOME/lode
echo 'export LODE_HOME=$HOME/lode' >> ~/.zprofile
echo 'export PATH="$LODE_HOME/opt/bin:$PATH"' >> ~/.zprofile
source ~/.zprofile

cd $LODE_HOME
./setup --dev          # creates  $LODE_HOME/dev/core
The script writes an initial autogen.input for a debug build. Typical Apple-silicon overrides you may add:
--with-archs=ARM64
--with-macosx-sdk=11.3
--with-macosx-version-min=11.0
--enable-release-build
After that:
cd dev/core
./autogen.sh
make check 2>&1 | tee build.log   # first build ~2 h on an M1 Air
open instdir/LibreOfficeDev.app   # test run
LODE commands and paths are documented here 
The Document Foundation Wiki
.
4 — Validate the tabbed interface (Notebookbar)
LibreOffice already ships an optional Notebookbar “Tabbed” UI that groups commands on ribbon-like tabs 
The Document Foundation Wiki
.
Activate it in View ▸ User Interface ▸ Tabbed (or enable experimental features first under Tools ▸ Options ▸ LibreOffice ▸ Advanced). If this meets your requirements, you can retire your custom tabs extension; otherwise you can extend the Notebookbar via XML or the UNO API.
5 — Port any OpenOffice-specific code
UNO interfaces are almost identical, but module names and make-targets have changed. Most extensions compile with minimal tweaks.
Licensing: LibreOffice core is dual-licensed MPL 2 + LGPL v3, not Apache 2. Audit any code you carry over.
6 — Update your Legacy-Modernisation project plan
Timeline block	LibreOffice-specific actions
Days 1–2	Architecture mapping & UNO API survey of the 10 M-line codebase; verify build succeeds on CI (GitHub Actions or Jenkins runner on an M-series Mac).
Days 3–4	Design modern add-ons (cloud storage, real-time collaboration) leveraging LibreOffice SDKs; build a minimal packaging pipeline (make build-non-l10n-package).
Days 5–6	Implement your six new features (OAuth login, Nextcloud integration, analytics dashboard, etc.).
Day 7	QA, notarise the .app, prepare user-migration docs from OpenOffice to LibreOffice.
This aligns neatly with the Enterprise Week grading rubric you’re working from.
7 — Common pitfalls to avoid
Slow rebuilds: enable ccache and raise the cache limit to 30 GB as recommended
The Document Foundation Wiki
.
Unit tests failing over SSH: keep a GUI session open or disable the problematic tests temporarily
The Document Foundation Wiki
.
In-process JVM on Apple-silicon: still flaky; run Base under Rosetta or skip Java until the upstream issue is fully closed 
The Document Foundation Wiki
.
You’re up !
Start with a clean test build via LODE; once LibreOfficeDev.app launches, you can iteratively bring over your custom functionality and proceed with the modernisation tasks. Ping me if any of the steps above block—happy hacking!
You said:
Silly question, but should I do this in a Docker container, or install all this globally, or does it not really matter?
ChatGPT said:

Thought for 3m 41s
Short answer
For macOS-targeted LibreOffice builds (the thing that turns into LibreOfficeDev.app and later a signed DMG) you really want to build natively on macOS with Xcode + Homebrew/LODE.
Docker is handy—but only when you are happy with a Linux build (e.g. head-less conversion service, CI sanity checks) or when you need a throw-away test sandbox.
Why native is the default on Apple-silicon Macs
LibreOffice’s macOS tool-chain calls the Apple-supplied clang, codesign, install_name_tool, etc. Those come with Xcode Command-Line Tools, which Apple’s licence forbids redistributing in a public Docker image. In practice there is no official “macOS + Xcode” container, and community hacks such as docker-osx are heavyweight VMs rather than lightweight containers 
Stack Overflow
.
The upstream build docs explicitly assume a host installation of Xcode ≥ 11.3
The Document Foundation Wiki
. LODE simply automates that assumption.
Only a native build can be notarised and signed, which you will need before shipping to end users or distributing through MDM.
When a Docker container does make sense
Good Docker use-cases	Caveats on macOS
Running LibreOffice for server-side conversion (e.g. soffice ↦ PDF) behind an API. LinuxServer’s lscr.io/linuxserver/libreoffice image publishes multi-arch tags including arm64v8, so it runs fine on an M-series Mac host 
LinuxServer
.	Container builds are Linux binaries—you can’t copy the resulting executables into a .app.
Continuous-integration sanity builds of cross-platform code on GitHub Actions or GitLab CI.	Use a Linux runner; you’ll still need a separate macOS runner for your real .app artifacts.
Isolating add-on experiments that do not touch the platform layer.	You lose macOS-specific UI widgets, accessibility hooks, and any code that talks to Core Services.
Practical workflow on your M-series MacBook Air
Option	What you install	Pros	Cons
Native (recommended)	Xcode + Homebrew packages (autoconf, ccache, …) + lode	• Exactly what upstream expects
• Codesigning & notarisation work
• Fast (all cores, no emulation)	• Homebrew clutters /opt/homebrew (use brew bundle to keep it tidy)
Docker (Linux image)	Docker Desktop + lscr.io/linuxserver/libreoffice	• Zero host deps beyond Docker
• Easy to wipe/reset	• Builds run inside qemu if you target x86 → slow
• Result is a Linux build, not a macOS .app
Full macOS VM (Parallels or Apple’s Virtualization.framework)	A second macOS install inside a VM	• 100 % isolation, reproducible	• Needs ≈ 60 GB disk & macOS licence; slower I/O
Suggested next steps for your pivot
Stick with the native LODE flow you already started: it is the path of least friction for delivering a Big Sur-compatible, signed LibreOffice fork.
Use a Brewfile in your repo so teammates can do brew bundle and get exactly the same dependencies, then remove them later with brew bundle cleanup.
Spin up a separate Linux-based Docker job in CI if you need headless regression tests. Keep it in parallel with (not instead of) your macOS job.
If you really want extra isolation on your laptop, create a second Homebrew prefix (HOMEBREW_PREFIX=$HOME/.brew-lo) and let LODE use that—no root access, easy to delete.
That way you get the ease of Docker where it helps and a build that Apple will actually let you ship. Happy hacking!
You said:
I think I do need that my work is platform agnostic as part of the project requirements
ChatGPT said:

Thought for 2m 22s
Below is a pragmatic roadmap that keeps your source tree, build flags, and feature-work identical—while still spitting out first-class installers for macOS, Windows and Linux. Think of it as “one repo, many runners.”
1 — Standardise the build recipe (LODE)
Put one canonical autogen.input under ./ci/ and reuse it everywhere (add only minimal per-platform overrides).
Use LODE on every OS; it hides 95 % of the distro quirks and already supports macOS & Windows out-of-the-box, with Linux coming via the classic configure path 
The Document Foundation Wiki
.
For Windows, LODE still expects a local Visual Studio ≥ 2019 + Cygwin; anything older is no longer supported 
The Document Foundation Wiki
.
Tip: hand the two prerequisites to Chocolatey and let LODE do the rest—lines 37-38 show that it automates everything else 
The Document Foundation Wiki
.
2 — Pick the right build hosts
Target artefact	Where to build	Why	Notes
macOS (arm64 + universal2)	GitHub Actions runner macos-14 (arm) or a self-hosted Intel Mac if you still need x86_64	Apple-signed tool-chain, notarisation	macos-latest on GA is now ARM-only; macos-13 is the last free x86_64 host 
Reddit
Windows 10/11 x64 & ARM64	GA windows-latest (x64) with VS 2022 or a cross-compile container (Ubuntu + Wine + MSVC) if you want everything in Linux	Official releases are still built natively; cross-compile is possible but fiddly	Wine + MSVC recipe exists; you map MSVC into Wine and call the MSVC tool-chain from Linux 
iMath
Linux	GA ubuntu-latest or a Dockerfile	Fast, trivial cache reuse	If you only need a runtime container, you can even skip the build and pull the maintained lscr.io/linuxserver/libreoffice multi-arch image (arm64 & x86_64) 
LinuxServer
3 — Wire it together with a CI matrix
# .github/workflows/build.yml (sketch)
strategy:
  matrix:
    os:   [ubuntu-latest, macos-14, windows-latest]
    arch: [x86_64]   # expand to arm64 where GA offers it
  fail-fast: false

steps:
- uses: actions/checkout@v4

# macOS & Windows → run the LODE bootstrap
- name: LODE bootstrap
  if: matrix.os != 'ubuntu-latest'
  run: |
    git clone https://git.libreoffice.org/lode $HOME/lode
    $HOME/lode/setup --prereq
    $HOME/lode/setup --dev

# Linux → classic dependencies or your own Dockerfile
- name: Linux deps
  if: matrix.os == 'ubuntu-latest'
  run: sudo apt-get -y install $(cat scripts/ubuntu-deps.txt)

# Common build
- run: |
    cd ${{ steps.checkout.outputs.path }}/dev/core
    ./autogen.sh --srcdir=$(pwd) --with-config=ci
    make -j$(nproc) build-nocheck
Add per-job artefact upload (make dmg, make msi, make appimage) and you have a fully platform-agnostic pipeline.
4 — Keep your code platform-agnostic
Stay in the UNO layer – implement your features as UNO C++ helpers or Python/Java extensions; the loader picks the right binary at runtime.
Notebookbar customisation lives in XML and ships identically across OSes—no conditional code needed.
Avoid direct Core Graphics / Win32 calls; if you must, wrap them behind a pre-processor switch and stub on the other platforms.
5 — Packaging & distribution checklist
Artefact	Tool	Signing	Where to stash
macOS DMG	make dmg → codesign → notarize (Apple ID token)	Yes	GitHub Release or Munki
Windows MSI	--enable-64-bit → make msidist	Optional (signtool)	Winget / SCCM
Linux	make appimage or flatpak-builder	No	Flatpak, Snap, plain .deb/.rpm
TL;DR
Native build per OS = fewer headaches (Apple tooling, VS MSVC, etc.).
Docker & Wine are extras—great for headless Linux services and experimental cross-compile, but not a replacement for notarised macOS or signed Windows artefacts.
Unify the recipe with LODE + one autogen.input; drive it via a GitHub Actions matrix so every push produces DMG + MSI + AppImage automatically.
Follow this and you’ll meet the “platform-agnostic” requirement without re-architecting your codebase—or your sanity. Happy hacking!
You said:
Let's say I'm starting totally from the beginning. Please synthesize all of this into a list of ordered tasks I can complete to achieve the local build
ChatGPT said:

Thought for 2m 18s
“Day-Zero” task list for a local, Apple-silicon LibreOffice build
Follow the steps in order—don’t skip the restarts—and you’ll end up with a working LibreOfficeDev.app that you built yourself on macOS 11.
#	Task	Key command(s) / action
1	Install Xcode 15 (or current) from the App Store, launch it once, then verify Preferences ▸ Locations ▸ Command Line Tools is populated.
The Document Foundation Wiki
2	Homebrew & build utilities
bash<br>/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"<br>brew install autoconf automake libtool gperf gawk pkg-config ccache bison flex coreutils<br> (If LODE asks for something else, install it the same way.) 
The Document Foundation Wiki
3 (-opt.)	Java JDK — only if you’ll use Base/macros; otherwise you can add --without-java later. 
The Document Foundation Wiki
4	Clone LODE and enter the folder bash<br>git clone https://gerrit.libreoffice.org/lode ~/lode<br>cd ~/lode<br>
The Document Foundation Wiki
5	Add LODE to your shell bash<br>echo 'export LODE_HOME=$HOME/lode' >> ~/.zprofile<br>echo 'export PATH="$LODE_HOME/opt/bin:$PATH"' >> ~/.zprofile<br>exec zsh # reload profile<br> 
The Document Foundation Wiki
6	Sanity-check prerequisites bash<br>./setup --prereq<br> (installs any missing SDK helpers)	
7	Bootstrap the dev tree bash<br>./setup --dev<br> – creates ~/lode/dev/core and a starter autogen.input. 
The Document Foundation Wiki
8 (-opt.)	Tweak dev/core/autogen.input if you need release flags or min-version overrides, e.g. text<br>--with-archs=ARM64<br>--with-macosx-version-min=11.0<br>--enable-release-build<br>--without-java<br>	
9	Configure & compile ```bash
cd ~/lode/dev/core
./autogen.sh
make -j$(sysctl -n hw.ncpu) check 2>&1	tee build.log
```
The Document Foundation Wiki
10	Run your build bash<br>open instdir/LibreOfficeDev.app<br>
The Document Foundation Wiki
11	Switch on the Tabbed (Notebookbar) UI inside LibreOffice: View ▸ User Interface ▸ Tabbed. 
LibreOffice Help
12 (-opt.)	Speed up future rebuilds with a 30 GB ccache and the required env-var: bash<br>ccache --max-size 30G<br>echo 'export CCACHE_CPP2=YES' >> ~/.zprofile<br> 
The Document Foundation Wiki
That’s it!
You now have a fully local Apple-silicon build. From here you can:
iterate on source code (make alone is faster after the first pass);
create a signed installer (make dmg) once you have an Apple Developer ID;
replicate the same LODE workflow on Windows or Linux runners for a platform-agnostic pipeline later.
You said:
**Install Xcode 15 (or current)** from the App Store, launch it once, then verify **Preferences ▸ Locations ▸ Command Line Tools** is populated.
What does this mean, what preferences?
ChatGPT said:

Thought for 45 seconds
It’s just telling you to open Xcode’s own settings and make sure the Command Line Tools dropdown isn’t blank.
How to check it
Launch Xcode once (this also lets it finish the one-time “Installing additional components…” step).
From the menu bar choose
Xcode ▸ Preferences… (Ventura/Sonoma: Xcode ▸ Settings…).
Shortcut: ⌘,
In the Preferences/Settings window click the Locations tab.
The very first field is Command Line Tools.
Pick the current Xcode in the dropdown, e.g. “Xcode 15.2 (15C5001f)”.
If it already shows your Xcode version, you’re good; if it’s empty, select the version so macOS knows where clang, git, etc. live. 
Ask Different
Close the window. You can confirm in Terminal with
xcode-select -p          # should print /Applications/Xcode.app/Contents/Developer
xcodebuild -version      # should match the version you picked
Why this matters
LODE (and Homebrew) invoke the tool-chain through xcode-select. If the dropdown is blank, Apple’s CLI tries to download the stand-alone Command Line Tools every time—or simply fails—causing mysterious “no developer tools found” or build errors. Setting it once prevents that headache.
You said:
I got to step 9 in the task list, but received this error/output:

georgejeffreys@Mac core % ./autogen.sh
Running './configure --enable-debug --srcdir=/Users/georgejeffreys/lode/dev/core --enable-option-checking=fatal'
********************************************************************
*
*   Running LibreOffice build configuration.
*
********************************************************************

checking build system type... aarch64-apple-darwin24.5.0
checking host system type... aarch64-apple-darwin24.5.0
checking for product name... LibreOfficeDev
checking for package version... 26.2.0.0.alpha0+
checking for product version... 26.2
checking for grep that handles long lines and -e... /usr/bin/grep
checking for egrep... /usr/bin/grep -E
checking for grep... (cached) /usr/bin/grep
checking for GNU Make... /usr/bin/make
checking for sed... /usr/bin/sed
checking for pkgconf... /opt/homebrew/bin/pkgconf
checking whether to use link-time optimization... no
checking whether to enable gcov... no
configure: VCL platform has a usable dynamic loading API: yes
configure: VCL platform uses freetype+fontconfig: no
configure: VCL platform uses headless plugin: no
configure: VCL platform uses X11: no
checking for explicit AFLAGS... no
checking for explicit CFLAGS... no
checking for explicit CXXFLAGS... no
checking for explicit OBJCFLAGS... no
checking for explicit OBJCXXFLAGS... no
checking for explicit LDFLAGS... no
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables... 
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether the compiler supports GNU C... yes
checking whether gcc accepts -g... yes
checking for gcc option to enable C11 features... none needed
checking for stdio.h... yes
checking for stdlib.h... yes
checking for string.h... yes
checking for inttypes.h... yes
checking for stdint.h... yes
checking for strings.h... yes
checking for sys/stat.h... yes
checking for sys/types.h... yes
checking for unistd.h... yes
checking whether byte ordering is bigendian... no
checking for library containing dlsym... none required
checking pkg-config is at least version 0.9.0... yes
checking for library containing backtrace... none required
checking whether to record build config... no
checking whether build target is Release Build... no
checking whether build should auto use hardening compiler flags... no
checking whether to build a Community flavor... yes
checking whether to sign windows build... no
checking for gawk... gawk
checking for gawk... /opt/homebrew/bin/gawk
checking for bash... /bin/sh
checking for pigz... no
checking for gzip... /usr/bin/gzip
checking for GNU or BSD tar... tar
checking for tar's option to strip components... --strip-components
checking how to build and package galleries... internal src images for desktop
checking build with or without template files... enable all templates
checking for ccache... /opt/homebrew/bin/ccache
checking whether version of ccache is suitable... yes, 4.11.3
checking gcc home... /usr
checking for gcc... /usr/bin/gcc
checking whether to build with Java support... yes
checking what macOS SDK to use... macOS SDK 15.5 at /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX15.5.sdk
checking what minimum version of macOS to require... 11.0
checking that macosx-version-min-required is coherent with macos-with-sdk... yes
checking whether Xcode is new enough... yes (16.4)
checking what C compiler to use... /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -target arm64-apple-macos -mmacosx-version-min=11.0 -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX15.5.sdk
checking what C++ compiler to use... /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang++ -target arm64-apple-macos -stdlib=libc++ -mmacosx-version-min=11.0 -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX15.5.sdk
checking whether to do code signing... no
checking whether to create a Mac App Store package... no
checking whether to sandbox the application... no
checking what macOS app bundle identifier to use... org.libreoffice.script
checking whether to treat the installation as read-only... yes
checking for -isystem ... yes
checking whether the compiler is actually Clang... yes
checking whether Clang is new enough... assumed yes (Apple Clang)
checking whether clang is already ccached... no
checking whether ccache clang supports -ggdb2... yes
checking whether to build with additional debug utilities... no
checking whether to do a debug build... yes
checking for -fuse-ld=lld linker support... no 
checking for -fuse-ld=gold linker support... no 
checking for -fuse-ld=mold linker support... no 
checking for linker that is used... unknown
checking for -Bsymbolic-functions linker support... not found 
checking for --gc-sections linker support... not found 
checking for -dead_strip linker support... found 
checking whether ccache clang supports -Xclang -debug-info-kind=constructor... yes
checking whether to generate debug information... yes
checking whether to compile with optimization flags... no
checking whether to build with .NET support... yes
checking for dotnet... /opt/homebrew/bin/dotnet
checking for .NET SDK 8 or higher... yes
checking which package format to use... none
checking which help to build... no
checking if we need to build the help index tooling... yes
checking whether to enable xapian-omega support for online help... no
checking whether to include the XML-help support... yes
checking for helppack integration... no integration
checking which lockfile binary to use... internal
checking for getopt.h... yes
checking for paths.h... yes
checking for sys/param.h... yes
checking for utime... yes
checking for utimes... yes
checking for syslog.h... yes
checking whether to turn warnings to errors... no
checking whether to have assert() failures abort even without --enable-debug... yes
checking whether to build the pagein binaries for oosplash... no
checking whether to enable CPDB support... no
checking whether to enable CUPS support... no
checking which fontconfig to use... not tested
checking whether we want to fetch tarballs... yes, if we use them
checking whether to include MySpell dictionaries... no
checking whether to use dicts from external paths... no
checking whether to enable pch feature... no
checking the GNU Make version... configure: error: failed (/usr/bin/make version >= 4.0 needed)
Error running configure at ./autogen.sh line 338.
georgejeffreys@Mac core % 
ChatGPT said:

Thought for 18 seconds
autogen.sh is complaining because it’s picking up macOS’s BSD make 3.81 at
/usr/bin/make, but LibreOffice’s build scripts need GNU Make ≥ 4.0.
How to fix it on your Apple-silicon Mac
What to do	Command (run in Terminal)	Why it works
1. Install GNU Make via Homebrew	bash<br>brew install make<br>	Homebrew’s make formula ships the current GNU Make (today it’s 4.4).
2. Expose it as make (not just gmake)	bash<br># Add the gnubin shim directory to the front of your PATH<br>echo 'export PATH="/opt/homebrew/opt/make/libexec/gnubin:$PATH"' >> ~/.zprofile<br>exec zsh # reload your shell<br>	Homebrew installs the binary as gmake, plus a gnubin/ directory that symlinks it to make. Pre-pending that directory makes which make resolve to GNU Make instead of /usr/bin/make.
-or-, quick one-off alternative	bash<br>export MAKE=/opt/homebrew/bin/gmake<br>./autogen.sh # rerun with correct make<br>	Sets the MAKE env-var just for this session; useful if you don’t want to touch your shell profile yet.
Verify it worked
which make        # → /opt/homebrew/opt/make/libexec/gnubin/make
make --version    # → GNU Make 4.x
Then re-run:
cd ~/lode/dev/core
./autogen.sh      # should pass the make check now
make -j$(sysctl -n hw.ncpu) check
Update your Day-Zero task list
Insert “brew install make” into step 2 so the next person (or CI runner) hits this dependency up-front. Everything else stays the same. Happy compiling!
You said:
Okay, very cool! I've got LibreOffice built. New question: how do I organize the documents in tabs? I was under the impression that that was a feature Libre has, though perhaps I misunderstood
ChatGPT said:

Thought for 48 seconds
The short version
LibreOffice’s “Tabbed UI” (Notebookbar) is a toolbar layout— it puts commands on ribbon-like tabs inside each window.
What LibreOffice doesn’t have (yet) is a “one-window-with-a-tab-per-document” feature like a web browser or VS Code. Every file still opens in its own macOS window.
What’s really available today
Feature	What it looks like	How to turn it on
Tabbed UI / Notebookbar
Tabs for commands inside one document	Ribbon-style tabs: **File	Home
Sheet tabs (Calc)	Tabs below the grid for worksheets inside one spreadsheet file	Built-in, no extra steps
No MDI / document tabs (Writer, Calc, Impress, Draw…)	–	Not implemented; still tracked as feature request tdf#37134 “Document-per-tab”
bugs.documentfoundation.org
Window menu & ⌘` (macOS)	Quick list / cycle of open LibreOffice windows	Window ▸ <document-name> or press ⌘`
macOS Stage Manager / Mission Control	OS-level grouping & tiling of windows	macOS feature (system settings)
Two recent Ask LibreOffice threads (Jan 2025 & May 2025) confirm the situation: “To the best of my knowledge, you can’t” open Writer docs in tabs—only separate windows. 
Ask LibreOffice
Ask LibreOffice
Work-arounds while we wait for tdf#37134
Need	Pragmatic option
Fast switching between many docs	Window ▸ <document> menu, or macOS Command + ` cycle (similar to Alt + Tab but scoped to one app).
Keep screen real estate	• Use macOS Tile Window to Left/Right to snap two documents side-by-side.
• Enable Stage Manager to group your LibreOffice windows together.
True tabbed interface for reference files	• Run Collabora Online / LibreOffice Online in a browser tab; those instances give you a browser-level tab per file.
• Or try a tiling window manager (Yabai, Amethyst) to auto-tile LibreOffice windows.
Sheets in Calc	Embed multiple tables in one .ods workbook and navigate via Calc’s own sheet tabs.
If tabs are a must-have for your project
Track the upstream bug — subscribe to tdf#37134 so you get email when someone finally lands an MDI/tab implementation. 
bugs.documentfoundation.org
Consider an extension idea — but be aware that existing UNO APIs can’t re-parent top-level VCL windows, so an extension alone can’t create real document tabs today.
Patch-level contribution — if your team is willing to dive into VCL (the LibreOffice UI toolkit), you could prototype a GtkNotebook / NSTabView / QTabBar wrapper for each document frame and post it to Gerrit; design discussions happen on the libreoffice@lists.freedesktop.org dev list.
Bottom line
Right now LibreOffice gives you command tabs, not document tabs. Stick with separate windows (plus macOS window-management tricks) or run the browser edition if you absolutely need multiple files in one tabbed container. If tabs are critical to your deliverable, plan on upstream development work—or rethink the UX so the Notebookbar plus quick window switching meets the requirement.
