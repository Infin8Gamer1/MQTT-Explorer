# MQTT Explorer – macOS local setup

Step-by-step guide to run and develop MQTT Explorer on macOS.

## Prerequisites

- **Node.js** ≥ 18  
  - Install via [nodejs.org](https://nodejs.org/) or with [nvm](https://github.com/nvm-sh/nvm):  
    `nvm install 18 && nvm use 18`
- **npm** (comes with Node.js)
- **Git** (for cloning; Xcode Command Line Tools or Homebrew)

---

## Global Yarn vs project-scoped setup

**Is global Yarn a concern?**  
Installing `yarn` globally is common and works fine. The main downsides:

- **Version drift** – Different projects may expect different Yarn versions (e.g. 1.x vs 2.x/3.x).
- **One version for everything** – Your global `yarn` is shared across all Node projects.

**Do you need an “environment” like in Python?**  
Not in the same way. In Node:

- **Dependencies are already per-project.** Every project has its own `node_modules/`; there’s no global package pile that conflicts between projects. You don’t need a venv-style isolation for *packages*.
- What *is* worth scoping per project is the **runtime and package manager**:
  - **Node version** – This project expects Node ≥ 18. Other projects might need 16 or 20.
  - **Yarn (optional)** – You can avoid a global Yarn install and use a project-friendly approach instead.

**Recommended: use a Node version manager**  
Using **nvm**, **fnm**, or **Volta** lets you switch Node version per project (e.g. `nvm use 18` in this repo). That’s the closest to “an environment per project” and avoids Node-version conflicts.

**Optional: avoid global Yarn with Corepack**  
If you prefer not to install Yarn globally, use [Corepack](https://nodejs.org/api/corepack.html) (built into Node 16.10+):

```bash
corepack enable
corepack prepare yarn@1.22.22 --activate   # or whatever version the project uses
```

Then `yarn` in this directory will use that version. You can also use `npx yarn` to run Yarn via npm without a global install (slightly slower first run).

The steps below use **global Yarn** for simplicity; use the Corepack approach if you want to keep everything project-scoped.

---

## 1. Clone the repo (if needed)

```bash
git clone https://github.com/thomasnordquist/MQTT-Explorer.git
cd MQTT-Explorer
```

---

## 2. Install Yarn globally

```bash
npm install -g yarn
```

Check:

```bash
yarn --version
```

---

## 3. Install dependencies

From the project root:

```bash
yarn
```

This installs root deps and runs the root `install` script, which installs the `app` subproject (see root `package.json`).

---

## 4. Run the app

### Option A – Run from built sources (like a normal user build)

```bash
yarn build
yarn start
```

- `yarn build`: compiles TypeScript and builds the app bundle.
- `yarn start`: launches the Electron app.

### Option B – Develop with hot reload

```bash
yarn dev
```

This runs the app dev server and Electron in parallel so you get live reload while coding.

---

## 5. (Optional) Run tests

- **Unit tests**

  ```bash
  yarn test
  ```

- **UI tests** (need a local [Mosquitto](https://mosquitto.org/) MQTT broker):

  1. Install Mosquitto (e.g. `brew install mosquitto`), start it.
  2. In one terminal:

     ```bash
     ./node_modules/.bin/chromedriver --url-base=wd/hub --port=9515 --verbose
     ```

  3. In another:

     ```bash
     yarn build
     node dist/src/spec/webdriverio.js
     ```

---

## Quick reference

| Goal              | Command     |
|-------------------|------------|
| Install deps      | `yarn`     |
| Build app         | `yarn build` |
| Run built app     | `yarn start` |
| Develop (hot reload) | `yarn dev` |
| Run unit tests    | `yarn test` |

---

## Project layout (from README)

- **`app/`** – rendering logic (UI)
- **`backend/`** – models, tests, connection handling
- **`src/`** – Electron main process and bindings  
- MQTT client: [mqttjs/MQTT.js](https://github.com/mqttjs/MQTT.js)

---

## Package for macOS distribution

To create a distributable macOS app (.dmg file):

### Prerequisites

- **Code signing** (optional but recommended):
  - Apple Developer account (for App Store or notarized builds)
  - Code signing certificate installed in Keychain
  - Provisioning profile in `res/MQTTExplorerdmg.provisionprofile` (for DMG builds)
  
  **Note:** For local testing without signing, you can build unsigned, but macOS may show security warnings.

### Steps

1. **Prepare a clean release build:**
   ```bash
   yarn prepare-release
   ```
   This creates a clean build in `build/clean/` with production dependencies only.

2. **Package for macOS:**
   ```bash
   yarn package mac
   ```
   This builds a **DMG** file (disk image) for macOS.

3. **Find your package:**
   DMGs are under `build/clean/build/` (e.g. `build/clean/build/MQTT Explorer-0.4.0-beta.7.dmg` and `…-arm64.dmg`).

   **Note:** A GitHub token is *not* required to build. It’s only used if you want to publish the built DMGs to GitHub Releases. Without `GH_TOKEN`, the script builds the DMGs and stops (no upload).

### Alternative: Direct electron-builder (without prepare-release)

If you want to skip the clean build step and package directly:

```bash
# First build the app
yarn build

# Then use electron-builder directly
npx electron-builder --mac dmg
```

The output will be in `build/` directory.

### Package formats

The `package.ts` script supports these macOS formats (edit the script to enable):

- **`dmg`** (default) – Disk image installer (most common for macOS)
- **`zip`** – Compressed app bundle (commented out in code)
- **`mas`** – Mac App Store package (requires App Store provisioning profile)

### Code signing and notarization

If you have an Apple Developer account:

1. **Set environment variables** (or configure in `package.json`):
   ```bash
   export APPLE_ID="your@email.com"
   export APPLE_APP_SPECIFIC_PASSWORD="your-app-specific-password"
   export APPLE_TEAM_ID="your-team-id"
   ```

2. The build will automatically:
   - Sign the app with your certificate
   - Notarize it with Apple (if configured)
   - Create a hardened runtime build

**Without signing:** The app will build but macOS Gatekeeper may warn users when they first open it.

---

## Troubleshooting

- **“node: command not found” or wrong Node version**  
  Use Node ≥ 18. With nvm: `nvm use 18` (or install 18 and set default).

- **“yarn: command not found”**  
  Run `npm install -g yarn` and ensure your PATH includes the npm global bin directory (often `~/.npm-global/bin` or similar).

- **Electron or build errors**  
  Try a clean install:
  ```bash
  rm -rf node_modules app/node_modules backend/node_modules
  yarn
  yarn build
  ```

- **Permission errors on global install**  
  Prefer installing yarn without `sudo` (e.g. use nvm or set npm’s `prefix` to a user directory).
