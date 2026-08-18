# Run doc — yaqie.github.io

Static portfolio site (plain HTML/CSS/JS + Bootstrap from CDN). No build step,
no package manager, no dependencies to install.

## Reproduce artifacts

None. There is no build output and no env/config files. A fresh checkout is
directly servable. The portfolio data lives in `data/portfolio.json` and is
fetched at runtime via jQuery AJAX (so a real HTTP server is required — opening
`index.html` via `file://` will not work).

## Run the server

Start a static file server from the project root. Requires `python3`
(`/usr/bin/python3`).

**Important (macOS TCC):** this project lives under `~/Documents`, which macOS
protects with TCC permissions. Processes started via `launchctl submit` do NOT
inherit that access and die with `PermissionError: [Errno 1] Operation not
permitted` when touching the project files. `nohup ... &` + `disown` also gets
reaped when the command finishes. The reliable way is a **double-fork** from a
shell that already has TCC access (the command runner's shell does):

```bash
python3 - <<'EOF'
import os, subprocess, sys
log = "<project-root>/.freebuff/preview.log"
if os.fork() == 0:
    os.setsid()
    with open(log, "a") as out, open(os.devnull) as devnull:
        subprocess.Popen([sys.executable, "-m", "http.server", "8137", "--bind", "127.0.0.1"],
                         cwd="<project-root>",
                         stdout=out, stderr=subprocess.STDOUT, stdin=devnull)
    os._exit(0)
EOF
```

- Port `8137` is the project's default for the preview. Pick a free port if taken.
- Verify it survived the shell exit: `lsof -nP -iTCP:8137 -sTCP:LISTEN` and
  `curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:8137/index.html` (expect 200).
- To stop: `kill <pid>` (pid from lsof, or the launchd label if used).
