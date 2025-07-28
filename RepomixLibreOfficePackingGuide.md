How do I use Repomix to pack the entire LibreOffice repo into a file I can feed to my prompts? Is there an MCP server, or what?

Thought for 46 seconds
ChatGPT said:
Here’s the quick-start workflow that people use to hand the entire LibreOffice codebase to an LLM, plus where the MCP server fits in.
1 Install (or just run) Repomix
# No local install needed
npx repomix@latest            # runs whatever version is current
# …or, if you want it on your PATH:
npm install -g repomix
Repomix
2 Clone LibreOffice once (optional but faster)
git clone https://git.libreoffice.org/core libreoffice-core
cd libreoffice-core
Why clone?
Repomix can pull a remote repo directly, but cloning locally first lets you re-run repomix or tweak include/exclude patterns without re-downloading 8-10 GiB every time.
3 Pack the whole repo
# Inside libreoffice-core
repomix --compress --style xml             # produces repomix-output.xml (~70 % smaller)
Key flags
flag	why you probably want it
--compress	Uses Tree-sitter to strip method bodies and keep signatures / comments – drops token count ~70 % on huge bases. 
Repomix
--style xml (default)	Keeps structure headers that Claude/ChatGPT parse well.
--ignore "**/*.png,**/*.jpg,**/*.git/**"	Extra safety for binaries if the repo has any. (Repomix already honours .gitignore.)
--top-files-length 0	Skip the “largest-file” summary if you only need raw code.
Packing LibreOffice (≈ 10 M LOC) with compression typically yields a 90–120 MB XML file that fits in Claude 3 “Artifacts” upload or GPT-4o’s file-context slot. Without --compress, expect ~400 MB+.
Remote one-liner (slower, no local checkout):
npx repomix@latest \
  --remote https://git.libreoffice.org/core \
  --compress
Repomix
4 Feed it to your prompt
Most assistants accept:
This file contains the entire LibreOffice source tree. 
Start by giving me a high-level architecture map, then …
Claude and GPT automatically chunk long XML; for smaller-context models you can:
split the output (split -l 20000 repomix-output.xml chunk_) and feed pieces
or run a second Repomix pass with --include "sw/**,vcl/**" etc. to discuss one module at a time.
5 (Optional) Run Repomix as an MCP server
If your IDE or agent framework speaks Model Context Protocol you can let the model pull code on demand instead of uploading files manually.
# Quick test
npx repomix@latest --mcp
Integration snippets
VS Code / Cursor:
code --add-mcp '{"name":"repomix","command":"npx","args":["-y","repomix","--mcp"]}'
Docker variant for sandboxed environments shown in docs.
Repomix
Once registered, your agent can call tools such as pack_codebase, pack_remote_repository, grep_repomix_output, etc., without you ever creating a standalone file. 
Repomix
Gotchas & tips
Runtime – LibreOffice takes ~15-20 min on a fast SSD + Node 20.
Memory – leave ~6 GB free (compression needs ASTs in RAM).
Secrets – Repomix’s Secretlint pass aborts if it finds hard-coded keys; add --no-security-check only if you’re sure. 
Repomix
Updating – npx repomix@latest always pulls the newest version; pin a version for reproducibility (npx repomix@0.15.4).
That’s all you need—pack, upload (or let MCP do it), and start prompting!
