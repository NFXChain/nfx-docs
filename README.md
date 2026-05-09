# NFX Chain Documentation

This repository contains the source for the official NFX Chain documentation, published at [https://nfxchain.io](https://nfxchain.io).

## 📚 Structure

```
nfx-docs/
├── index.html          # GitHub Pages entry point
├── README.md           # Project overview
├── assets/
│   ├── styles.css      # Main stylesheet
│   ├── scripts.js      # Navigation & interactivity
│   └── images/         # Screenshots, diagrams
└── docs/
    ├── overview.md
    ├── architecture.md
    ├── faq.md
    ├── guides/
    │   ├── installation.md
    │   ├── deployment.md
    │   └── configuration.md
    ├── modules/
    │   ├── core.md
    │   ├── bindings.md
    │   ├── go.md
    │   └── wallet.md
    ├── examples/
    │   └── index.md
    └── api/
        ├── go.md
        ├── c.md
        └── rpc.md
```

## 🚀 Publishing to GitHub Pages

### 1. Move to Repository Root

For GitHub Pages to work, either:

**Option A** — Repository root (requires moving files):
```bash
# Clone gh-pages branch or move to root
git mv docs/ .
git rm -r nfx-docs/
# Adjust paths in index.html accordingly
```

**Option B** — `gh-pages` branch + `/docs` folder on main (simpler):
```bash
# Keep structure as-is, but set GitHub Pages source to:
# Branch: main  |  Folder: /nfx-docs
# Settings → Pages → Build & deployment → Source
```

### 2. GitHub Pages Settings

In GitHub repository:
1. **Settings** → **Pages**
2. **Build and deployment** → **Source**: `Deploy from a branch`
3. **Branch**: `main` (or `gh-pages`)
4. **Folder**: `/nfx-docs` (if docs folder) or `/root` (if moved to root)
5. Save

Site publishes at: `https://<username>.github.io/nfx-chain/`

### 3. Custom Domain (Optional)

Add `CNAME` file to `nfx-docs/`:

```
docs.nfxchain.io
```

In repo Settings → Pages → Custom domain.

---

## 🛠️ Local Development

Edit files locally, then preview:

```bash
# Simple HTTP server (Python)
cd nfx-docs
python3 -m http.server 8000

# Or use live-server (requires npm)
npx live-server nfx-docs/

# Open http://localhost:8000
```

---

## 📝 Adding Content

### New Guide

1. Create `docs/guides/my-guide.md`
2. Add link to `index.html` navigation:
   ```html
   <li><a href="#guides" class="nav-link">My Guide</a></li>
   ```
3. Add section `id="guides"` in main content

### Update Existing Doc

Edit `.md` file. Links use relative paths:
```markdown
[Core Module](../docs/modules/core.md)
```

---

## 🎨 Styling

CSS in `assets/styles.css`. Dark theme by default.

Variables (CSS custom properties):

```css
:root {
    --primary: #3498db;      /* Primary blue */
    --secondary: #2ecc71;    /* Success green */
    --background: #0a0e17;   /* Dark background */
    --surface: #1a1f2e;      /* Card background */
    --text: #e8eef7;         /* Main text */
    --text-secondary: #94a3b8;
}
```

To change theme, edit `:root` values.

---

## 🔗 Link Checking

Before submitting PR:

```bash
# Check for broken links (requires html-proofer)
gem install html-proofer
htmlproofer nfx-docs/ --check-html --disable-external

# Or use linkchecker (Node.js)
npx linkchecker http://localhost:8000
```

---

## 🤝 Contributing

1. Fork this repository
2. Create feature branch
3. Make changes
4. Test locally
5. Submit PR to `main` branch

All documentation is under **MIT License** (same as codebase).

---

## 📧 Contact

For documentation issues: [GitHub Issues](https://github.com/NFXChain/nfx-chain/issues?q=label%3Adocs)

---

*Last updated: May 2026 | NFX Chain Team*
