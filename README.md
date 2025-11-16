---

# **Camoufox — Complete Documentation**

Camoufox is a Firefox-based anti-detect browser with **fingerprint generation**, **fingerprint injection**, and **environment configuration**, designed for Playwright automation workflows in Python.

> **Project status:** Active development. Some features may change and certain hardening patches may be maintained externally.

---

# **Table of Contents**

1. Introduction
2. Quick Start
3. Installation
4. CLI Reference
5. Python API
6. Parameters & Configuration
7. Execution Contexts (Isolated vs Main World)
8. Proxies & GeoIP
9. Headless vs Virtual Display (Linux)
10. Profiles & Persistence
11. Remote Server (Playwright WebSocket Mode)
12. Fingerprint System
     • Navigator
     • Fonts
     • Screen & Window
     • Canvas
     • WebGL
     • WebRTC
     • Headers
13. Architecture
14. Troubleshooting
15. FAQ
16. Version & Compatibility
17. Security, Privacy & Telemetry
18. Acceptable Use
19. Legal
20. Development
     • Build (CLI)
     • Build (Docker)
     • Leak Debugging Workflow
21. Contributing
22. Code of Conduct
23. Changelog

---

# **1. Introduction**

Camoufox is a custom-built Firefox automation browser with engine-level fingerprint spoofing and realistic environment simulation. It integrates directly with **Playwright**, exposing both **sync** and **async** Python APIs.

Key goals:

* Provide **realistic, coherent fingerprints** (via BrowserForge-style profiles)
* Maintain **Playwright compatibility**
* Reduce JavaScript-observable anomalies through **engine-level patches**
* Provide **clear, safe defaults** without promising absolute invisibility

Camoufox does **not** provide site-specific bypasses and should be used lawfully.

---

# **2. Quick Start**

### **Install**

```bash
pip install -U "camoufox[geoip]"
```

### **Fetch the browser**

```bash
camoufox fetch
```

### **Sync usage**

```python
from camoufox.sync_api import Camoufox

with Camoufox() as browser:
    page = browser.new_page()
    page.goto("https://example.com")
```

### **Async usage**

```python
import asyncio
from camoufox.async_api import Camoufox

async def main():
    async with Camoufox() as browser:
        page = await browser.new_page()
        await page.goto("https://example.com")

asyncio.run(main())
```

---

# **3. Installation**

### **Package installation**

```bash
pip install -U "camoufox[geoip]"
```

### **Fetch browser build**

```bash
camoufox fetch
```

### **Remove browser builds**

```bash
camoufox remove
```

### **OS Dependencies**

| OS      | Dependencies                                                  |
| ------- | ------------------------------------------------------------- |
| Linux   | `libgtk-3`, `libnss3`, `xvfb` (if using `headless="virtual"`) |
| macOS   | Xcode Command Line Tools                                      |
| Windows | Visual C++ runtime (if missing)                               |

---

# **4. CLI Reference**

### **Download browser**

```bash
camoufox fetch
```

### **Remove builds**

```bash
camoufox remove
```

### **Start WebSocket automation server**

```bash
python -m camoufox server
```

### **Show path**

```bash
camoufox path
```

### **Show version**

```bash
camoufox version
```

---

# **5. Python API**

Import:

```python
from camoufox.sync_api import Camoufox
from camoufox.async_api import Camoufox as AsyncCamoufox
```

Both versions mirror Playwright’s API.

---

# **6. Parameters & Configuration**

Below are **Camoufox-specific** parameters. All Playwright Firefox launch options are also supported.

---

## **Device Rotation**

| Parameter      | Type                 | Description                                                            |
| -------------- | -------------------- | ---------------------------------------------------------------------- |
| `os`           | Optional[List/str]   | Target OS: `"windows"`, `"macos"`, `"linux"` or list for random choice |
| `fonts`        | Optional[List[str]]  | Additional font families                                               |
| `screen`       | Optional[Screen]     | Constrain generated screen dimensions                                  |
| `webgl_config` | Optional[(str, str)] | Fixed WebGL `(vendor, renderer)`                                       |

---

## **Configuration**

| Parameter            | Type                          | Description                                  |
| -------------------- | ----------------------------- | -------------------------------------------- |
| `humanize`           | Optional[bool/float]          | Human-like cursor movement                   |
| `headless`           | Optional[bool/"virtual"]      | `'virtual'` uses Xvfb on Linux               |
| `addons`             | Optional[List[str]]           | Paths to extracted browser add-ons           |
| `exclude_addons`     | Optional[List[DefaultAddons]] | Remove bundled add-ons                       |
| `window`             | Optional[(int,int)]           | Window size                                  |
| `main_world_eval`    | Optional[bool]                | Allow `mw:` script execution in page context |
| `enable_cache`       | Optional[bool]                | Enable page request caching                  |
| `persistent_context` | Optional[bool]                | Use persistent Playwright profile            |

---

## **Location & Language**

| Parameter | Type               | Description                               |
| --------- | ------------------ | ----------------------------------------- |
| `geoip`   | Optional[str/bool] | Auto-set timezone, locale, region from IP |
| `locale`  | Optional[str/list] | One or more locales                       |

---

## **Toggles**

| Parameter      | Description                                                               |
| -------------- | ------------------------------------------------------------------------- |
| `block_images` | Disable images                                                            |
| `block_webrtc` | Disable WebRTC                                                            |
| `block_webgl`  | Disable WebGL                                                             |
| `disable_coop` | Disable Cross-Origin-Opener-Policy (useful for Turnstile/iframe clicking) |

---

## **Config Override Example**

```python
with Camoufox(config={
    "webrtc:ipv4": "123.45.67.89",
    "webrtc:ipv6": "abcd:1234::1"
}) as browser:
    page = browser.new_page()
```

---

# **7. Execution Contexts**

### **Isolated world (default)**

* Scripts run safely outside page context
* Not visible to site JavaScript
* Safe for DOM reads

### **Main world**

Used only when editing DOM:

```python
page.evaluate("mw:document.querySelector('h1').remove()")
```

**Warning:** detectable and higher-risk.

---

# **8. Proxies & GeoIP**

Ensure timezone, locale, WebRTC IP, and region match the proxy exit IP.

### **Example**

```python
with Camoufox(
    geoip=True,
    proxy={
        "server": "http://proxy.example:8080",
        "username": "user",
        "password": "pass",
    }
) as browser:
    page = browser.new_page()
    page.goto("https://www.browserscan.net")
```

---

# **9. Headless vs Virtual Display (Linux)**

### **Headless:**

```python
Camoufox(headless=True)
```

### **Virtual Display (recommended for realism):**

```python
Camoufox(headless="virtual")
```

Install Xvfb:

```bash
sudo apt install xvfb
```

---

# **10. Profiles & Persistence**

Persistent session:

```python
with Camoufox(persistent_context=True) as browser:
    page = browser.new_page()
```

Recommended for:

* logged-in sessions
* multi-step flows
* long-running tasks

---

# **11. Remote Server (Playwright WebSocket)**

Start server:

```bash
python -m camoufox server
```

Python launch:

```python
from camoufox.server import launch_server
launch_server(port=1234, ws_path="hello")
```

Connect via Playwright:

```python
browser = p.firefox.connect("ws://localhost:1234/hello")
```

---

# **12. Fingerprint System**

Camoufox fingerprints multiple domains:

---

## **Navigator**

Properties:

* userAgent
* platform
* deviceMemory
* hardwareConcurrency
* languages
* maxTouchPoints

---

## **Fonts**

* Must match OS expectations
* Avoid unrealistic custom font lists

---

## **Screen & Window**

* Keep width/height consistent with OS and device class
* Use realistic window sizes

---

## **Canvas**

* Patched via Skia
* Stable fingerprints
* Rotate per-session, not per-request

---

## **WebGL**

* Use valid `(vendor, renderer)` tuple
* Removing WebGL entirely can be suspicious

---

## **WebRTC**

* Expose IP consistent with proxy region
* Or disable (`block_webrtc=True`)

---

## **Headers**

* Align Accept-Language with `locale`
* Ensure UA hints match OS

---

# **13. Architecture**

Camoufox modifies Firefox at the engine level:

* C++ patches for WebGL, Canvas, WebRTC, Navigator
* Script isolation model for DOM interaction
* Runtime toggles for fingerprint controls
* Playwright integration layer for sync/async control

---

# **14. Troubleshooting**

| Symptom                              | Cause               | Fix                            |
| ------------------------------------ | ------------------- | ------------------------------ |
| Turnstile/checkbox cannot be clicked | COOP isolation      | `disable_coop=True`            |
| Headless increases detections        | Pure headless       | use `headless="virtual"`       |
| Wrong timezone/locale                | Proxy mismatch      | `geoip=True` and set locale    |
| WebGL flagged                        | Wrong renderer      | Provide correct `webgl_config` |
| Fonts inconsistent                   | Wrong font families | Use OS-native fonts            |
| Add-ons missing                      | Persistent disabled | `persistent_context=True`      |

---

# **15. FAQ**

**Is Camoufox undetectable?**
No. It reduces anomalies, but detection may still occur.

**Does Camoufox bypass site X?**
No site-specific bypass instructions are provided.

**Node.js support?**
Use Remote Server (`python -m camoufox server`).

**Where are builds stored?**
In Camoufox’s internal cache.

---

# **16. Version & Compatibility**

| Component    | Version |
| ------------ | ------- |
| Camoufox     | x.y.z   |
| Base Firefox | N       |
| Playwright   | 1.xx    |
| Dataset      | varies  |

Pin versions in production.

---

# **17. Security, Privacy & Telemetry**

* Firefox telemetry is minimized
* Persistent profiles store data locally
* No analytics added by Camoufox
* Use proxies legally and ethically

---

# **18. Acceptable Use**

You may **not** use Camoufox to:

* Violate Terms of Service
* Automate without authorization
* Evade security controls
* Engage in fraud or harm

Allowed:

* QA testing
* Performance/load testing with permission
* Research
* Accessibility automation

---

# **19. Legal**

Camoufox is provided **as-is**, without warranty.
You are responsible for legal compliance and respecting third-party terms.

---

# **20. Development**

---

## **Build (CLI)**

```bash
make dir
make bootstrap
python3 multibuild.py --target linux windows macos --arch x86_64 arm64 i686
```

Artifacts generated in `dist/`.

---

## **Build (Docker)**

```bash
docker build -t camoufox-builder .
docker run -v "$(pwd)/dist:/app/dist" camoufox-builder --target <os> --arch <arch>
```

---

## **Leak Debugging Workflow**

1. Reproduce on minimal script
2. Capture JS-visible signals
3. Compare with stock Firefox
4. Disable toggles to isolate (`block_webgl`, `block_webrtc`, etc.)
5. Verify across multiple personas

---

# **21. Contributing**

* Fork
* Make feature branch
* Add/update tests
* Document user-visible changes
* Submit PR with clear reproduction steps

---

# **22. Code of Conduct**

Be respectful, collaborative, and professional.
Harassment is not tolerated.

---

# **23. Changelog**

```
[Unreleased]
- Consolidated single-file documentation

[x.y.z]
- Initial release
```

