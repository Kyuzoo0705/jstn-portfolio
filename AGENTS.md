# AGENTS.md

Static single-page portfolio: HTML + CSS + inline JS. No build system, no package manager, no tests, no CI.

## Layout

- `index.html` — all markup; all JavaScript lives inline in one `<script>` block at the bottom. Keep it that way: don't add a separate `.js` file or a bundler.
- `css/style.css` — all styling, linked with a cache-busting query (`css/style.css?v=28`).
- `images/` — portrait, project screenshot, and favicon. Use lowercase filenames with no spaces.
- `README.md` is UTF-8.

## Verifying changes

No lint/test to run. Preview with a static server (`python3 -m http.server 8000 --bind 127.0.0.1`) and open the URL in a browser. Check both themes and the phone-width layout — the site is developed and viewed on a mobile device.

Useful checks:

```sh
curl -s -H "Content-Type: text/html; charset=utf-8" --data-binary @index.html \
  "https://validator.w3.org/nu/?out=json" > /data/data/com.termux/files/usr/tmp/opencode/validate.json

python3 - <<'PY'
import re
from pathlib import Path
html = Path("index.html").read_text(encoding="utf-8")
scripts = re.findall(r"<script(?![^>]*src=)[^>]*>(.*?)</script>", html, re.S)
Path("/data/data/com.termux/files/usr/tmp/opencode/inline.js").write_text(scripts[-1], encoding="utf-8")
PY
node --check /data/data/com.termux/files/usr/tmp/opencode/inline.js
```

## Gotchas

- **Cache busting:** after changing `css/style.css`, bump the `?v=` number on its `<link>` in `index.html`, or returning visitors get stale CSS.
- **Theming:** two themes defined as CSS custom properties — `:root` (purple, dark, default) and `[data-theme="gold"]` (light). Never hardcode colors outside the tokens; add or extend variables and use `var(--...)`. The toggle flips `html[data-theme]`; an explicit choice is saved in localStorage key `portfolio-theme`, and with no saved choice it follows the system `prefers-color-scheme`. Theme switching uses the View Transitions API with a plain fallback — don't break that path.
- **Icons:** Lucide via CDN. Add icons as `<i data-lucide="icon-name">`; they render only after `lucide.createIcons()` near the bottom of the inline script. The call is guarded so a CDN failure does not stop the rest of the script.
- **Navigation:** the mobile menu uses `aria-expanded`, `aria-controls`, Escape-to-close, and outside-click close. Section links set `aria-current="location"`. Keep the states in sync when editing.
- **Motion:** sections are always visible. Don't add scroll-triggered reveal observers or card slide-ins. Keep the short profile load animation and the theme transition.
- **Performance choices are deliberate:** e.g. the sticky header uses near-opaque color-mix instead of `backdrop-filter` because blur repaints every scroll frame and lags on mobile; the mobile menu is fully opaque for the same reason. Don't reintroduce blur-heavy effects without a reason.
- **Content:** sections marked `<!-- PLACEHOLDER -->` contain sample content the owner will replace with real material — preserve their structure; don't polish the copy as if it were final. The first project is the portfolio itself and uses a real screenshot; the second project is still a placeholder.
- **Images:** `images/profile.jpg` is the optimized 680×850 portrait (58 KB). The original 1.2 MB `profile.png` is only in git history; don't re-add it.
