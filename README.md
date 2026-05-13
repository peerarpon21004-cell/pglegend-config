# PGLEGEND — Remote Config

Static JSON + HTML served via **GitHub Pages**, fetched by the iOS app at every cold launch. Lets you toggle maintenance mode and rotate URLs **without resubmitting the app**.

## 📁 Files in this directory

| File | Purpose |
|---|---|
| `config.json` | The actual config the app fetches. **This is the only file the app reads directly.** |
| `maintenance.html` | Web page shown when `maintenance.active = true` |
| `privacy.html` | (Add later) Privacy policy page — link from `policy_url` |
| `README.md` | This file |

---

## 🚀 Setup (one-time, ~10 minutes)

### 1. Create a new public GitHub repo
- Go to https://github.com/new
- Name it `pglegend-config` (or any name, just remember it)
- Make it **Public**
- Don't add README / .gitignore (we'll push our own)

### 2. Push these files
```bash
cd /Users/spacewatch/Documents/project-app/PGLEGEND/RemoteConfig
git init
git add config.json maintenance.html README.md
git commit -m "initial remote config"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/pglegend-config.git
git push -u origin main
```

### 3. Turn on GitHub Pages
- Repo → **Settings** → **Pages**
- Source: **Deploy from a branch**
- Branch: **main** / root (`/`)
- Save

GitHub will show your URL — something like:
```
https://YOUR_USERNAME.github.io/pglegend-config/
```

Your files will be live at:
- Config: `https://YOUR_USERNAME.github.io/pglegend-config/config.json`
- Maintenance page: `https://YOUR_USERNAME.github.io/pglegend-config/maintenance.html`

> ⏰ First publish takes ~1-2 minutes. Look for the green checkmark on the Actions tab.

### 4. Update the iOS app
Open `PGLEGEND/Services/RemoteConfigService.swift` and change:
```swift
static let configURL = URL(string: "https://spacewatch.github.io/pglegend-config/config.json")!
```
↓ replace with your URL ↓
```swift
static let configURL = URL(string: "https://YOUR_USERNAME.github.io/pglegend-config/config.json")!
```

Also edit `config.json` and change the `maintenance.url` to point to *your* `maintenance.html`:
```json
"url": "https://YOUR_USERNAME.github.io/pglegend-config/maintenance.html"
```

---

## 🛠️ How to use it after setup

### Turn maintenance ON (block users from using the app)
1. Edit `config.json` → set `"active": true`
2. `git add config.json && git commit -m "maintenance on" && git push`
3. Wait ~30-60 seconds → users opening the app now see the maintenance page
4. (Optional) Edit `maintenance.html` to update the message, push again — instant

### Turn maintenance OFF
1. Edit `config.json` → set `"active": false`
2. Commit + push
3. ~30-60s later, users get the app normally

### Change the message
- Edit `maintenance.html` directly (HTML/CSS — easy)
- `git push` → live instantly (no JSON change needed; same URL)

---

## ⚠️ Important: App Store review safety

When you submit your app to Apple, **`maintenance.active` MUST be `false`**.

If Apple's reviewer opens the app and sees the maintenance page, they'll reject the submission because they can't evaluate the actual gameplay. They typically review within 24-48 hours, so don't toggle maintenance during that window.

You can verify by visiting the config URL in a browser:
- https://YOUR_USERNAME.github.io/pglegend-config/config.json

If it shows `"active": false`, you're safe to submit.

---

## 🔒 What if the user has no internet?

The app times out the fetch after 3 seconds. If it fails:
- It uses the **bundled fallback** baked into the app (`RemoteConfig.bundledFallback` in code) which has `active = false`
- The user gets the game normally

This guarantees the app always launches even if:
- The user is offline
- GitHub is having an outage
- You broke the JSON syntax

The maintenance kill-switch is **opportunistic**, not enforced. That's a good thing — it can't lock users out permanently.

---

## 📝 JSON schema reference

```json
{
  "maintenance": {
    "active": false,                              // true = block app, show webview
    "url": "https://.../maintenance.html",        // page to show (HTTPS required)
    "title_th": "ปิดปรับปรุงระบบ",                // optional, not currently displayed
    "title_en": "Under Maintenance"               // optional
  },
  "policy_url": "https://.../privacy.html"        // optional — for future Settings link
}
```

---

## 💡 Tips

- **Don't break the JSON.** Use https://jsonlint.com/ before pushing.
- **Keep the URL short** and HTTPS only.
- **Test before launching.** Toggle `active: true` in your repo, run the app — you should see the webview.
- **Don't put secrets here.** This config is public on the internet.
