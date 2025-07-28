New problem that I hadn't considered: my Mac is too new to build OpenOffice, it seems. At least, I am having a great deal of trouble doing so. What are my options here?
ChatGPT said:
Yeah—trying to build Apache OpenOffice on a new Mac (Apple Silicon) is notoriously difficult. The project relies on an outdated build system (dmake + legacy autoconf), old versions of dependencies, and assumes an x86_64 environment. This makes it a painful choice for direct compilation on M1/M2/M3 hardware.
You have a few solid options:
✅ Option 1: Use a Linux Virtual Machine or Docker Container
Recommended if you want to build OpenOffice as-is.
Spin up an x86_64 Ubuntu VM using UTM or Parallels.
Or use Docker with an x86_64 Ubuntu base image and install dependencies there.
Example Docker base:
FROM ubuntu:20.04
RUN dpkg --add-architecture i386 && apt update && apt install -y build-essential ...
This avoids Mac-specific issues and gives you a reproducible environment.
✅ Option 2: Use Prebuilt Binaries for Exploration
Recommended if you're primarily trying to analyze and understand the legacy system.
Download the official .dmg installer for OpenOffice from openoffice.org.
Use it to explore the UI and exported features.
Then clone the source and analyze the architecture, file structure, and business logic without building.
⚠️ Option 3: Switch to LibreOffice
If you're struggling too much, consider switching to LibreOffice—which is better maintained and may be easier to build in modern environments.
LibreOffice is a fork of OpenOffice, and still shares much of the legacy architecture.
They offer developer build instructions for macOS including M1/M2 support.
⚠️ Option 4: Switch to Another Project Path
If your focus is more on modernization than specifically OpenOffice, consider one of the other project tracks like:
Plone (Python) or
SugarCRM (PHP)
These are easier to run and debug on modern machines.
Summary Table:
Option	Build?	Analyze?	Modernize?	Mac M1-friendly?
Linux VM / Docker	✅	✅	✅	✅
Prebuilt Binaries	❌	✅	✅ (partial)	✅
LibreOffice fork	✅	✅	✅	✅ (mostly)
Switch project path	✅	✅	✅	✅
Let me know how deep you want to go into build/debugging, or if you'd rather pivot to a faster path focused on modernization and delivery.
