# Version Repository Setup Guide

**Purpose:** This repository serves version data for My Smart Home OS updates.

---

## 🎯 What This Repository Does

This repository must be published via **GitHub Pages** to serve the version data JSON file that your custom OS and Supervisor use to check for updates.

**Required URL:** `https://my-smart-homes.github.io/version-data/data.json`

---

## 📋 Setup Instructions

### 1. Enable GitHub Pages

1. Go to repository settings: https://github.com/my-smart-homes/version/settings/pages
2. Under "Source", select:
   - **Source:** Deploy from a branch
   - **Branch:** `main` 
   - **Folder:** `/ (root)`
3. Click **Save**
4. Wait 1-2 minutes for deployment

### 2. Verify Deployment

Check that the file is accessible:
```bash
curl https://my-smart-homes.github.io/version-data/data.json
```

Should return the JSON content.

### 3. Update DNS (If Using Custom Domain)

If you want to use `my-smart-homes.github.io/version-data/`:
- Repository name must be: `version-data` (not `version`)
- Or set up a custom domain in Pages settings

---

## 🔄 Current Configuration Issue

**Problem:** Your repository is named `version` but the code expects `version-data`

**Your code expects:**
```
https://my-smart-homes.github.io/version-data/data.json
```

**Currently deployed at:**
```
https://my-smart-homes.github.io/version/data.json
```

### Solution Options:

#### Option A: Rename Repository (Recommended)
1. Go to Settings → General
2. Scroll to "Repository name"
3. Change from `version` to `version-data`
4. GitHub Pages URL will automatically become:
   `https://my-smart-homes.github.io/version-data/data.json` ✅

#### Option B: Update Code References
Change all references from `/version-data/` to `/version/`:

**Files to update:**
1. `operating-system/buildroot-external/rootfs-overlay/usr/sbin/hassos-supervisor`
   ```bash
   # Change line ~54 from:
   https://my-smart-homes.github.io/version-data/data.json
   # To:
   https://my-smart-homes.github.io/version/data.json
   ```

2. `supervisor/supervisor/const.py`
   ```python
   # Change line 22 from:
   URL_HASSIO_VERSION = "https://my-smart-homes.github.io/version-data/data.json"
   # To:
   URL_HASSIO_VERSION = "https://my-smart-homes.github.io/version/data.json"
   ```

**Recommendation:** Option A (rename repo) is cleaner and matches the expected structure.

---

## 📝 File Structure

```
version-data/  (or version/)
├── data.json                 ← Main version file (REQUIRED)
├── stable.json               ← Legacy format (optional)
├── dev.json                  ← Dev channel (optional)
├── beta.json                 ← Beta channel (optional)
├── apparmor*.txt             ← AppArmor profiles
└── README.md
```

---

## 🔧 Updating Versions

### When You Release New Builds

Edit `data.json` and update the relevant versions:

```json
{
  "supervisor": "2025.12.3",       ← Update when Supervisor changes
  "homeassistant": {
    "generic-x86-64": "2024.12.0"  ← Update when Core changes
  },
  "hassos": {
    "generic-x86-64": "13.7.8"     ← Update when OS changes
  }
}
```

Then commit and push:
```bash
git add data.json
git commit -m "Update to Core 2024.12.1"
git push origin main
```

GitHub Pages will automatically update within ~1 minute.

---

## ⚠️ Critical Notes

### Version Format
- **Supervisor:** Semantic versioning (e.g., `2025.12.3`)
- **Core:** Semantic versioning (e.g., `2024.12.0`)
- **OS:** Version from `operating-system/buildroot-external/meta` (e.g., `13.7.8`)

### Container Registry URLs
All `images` entries point to `ghcr.io/my-smart-homes/*`:
- ✅ Correct: `"core": "ghcr.io/my-smart-homes/{machine}-my-smart-homes"`
- ❌ Wrong: `"core": "ghcr.io/home-assistant/{machine}-homeassistant"`

### OTA Update URL
Points to GitHub releases:
```json
"ota": "https://github.com/my-smart-homes/operating-system/releases/download/{version}/{os_name}_{board}-{version}.raucb"
```

You'll need to create GitHub releases with the `.raucb` files attached.

---

## 🧪 Testing

### Test Version Endpoint
```bash
# Should return JSON
curl https://my-smart-homes.github.io/version-data/data.json

# Pretty print
curl -s https://my-smart-homes.github.io/version-data/data.json | jq .
```

### Test From Running System
```bash
# SSH into your device
ssh root@homeassistant.local

# Check what version Supervisor fetches
curl -s https://my-smart-homes.github.io/version-data/data.json | jq .supervisor
```

---

## 🚀 Deployment Checklist

- [ ] Repository enabled for GitHub Pages
- [ ] Repository named `version-data` (or code updated to match)
- [ ] `data.json` file exists and is valid JSON
- [ ] All `images` URLs point to `ghcr.io/my-smart-homes/*`
- [ ] Versions match what's actually built
- [ ] URL is accessible: `https://my-smart-homes.github.io/version-data/data.json`
- [ ] Container images exist in registry for all listed versions

---

## 📊 Version Matching

Ensure versions in `data.json` match:

| Component | data.json Version | Must Match |
|-----------|-------------------|------------|
| **Supervisor** | `supervisor: "2025.12.3"` | Tag in `ghcr.io/my-smart-homes/amd64-hassio-supervisor` |
| **Core** | `homeassistant.generic-x86-64: "2024.12.0"` | Tag in `ghcr.io/my-smart-homes/generic-x86-64-my-smart-homes` |
| **OS** | `hassos.generic-x86-64: "13.7.8"` | `VERSION_SUFFIX` in `operating-system/buildroot-external/meta` |

---

## 🐛 Troubleshooting

### Issue: 404 Not Found
**Cause:** GitHub Pages not enabled or wrong URL
**Fix:** Enable Pages, wait 2 minutes, check URL matches repo name

### Issue: Supervisor can't fetch versions
**Cause:** URL mismatch between code and actual deployment
**Fix:** Either rename repo to `version-data` or update code to use `/version/`

### Issue: Updates not working
**Cause:** Container images don't exist for listed versions
**Fix:** Ensure all images are pushed to `ghcr.io/my-smart-homes/*`

### Issue: OS update fails
**Cause:** `.raucb` file doesn't exist at OTA URL
**Fix:** Create GitHub release with `.raucb` file attached

---

**Created:** December 24, 2025  
**Last Updated:** December 24, 2025
