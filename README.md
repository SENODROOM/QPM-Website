<div align="center">

<br/>

# ⚛&nbsp;&nbsp;QPM

### Quantum Package Manager

**The registry where Quantum Language modules are discovered, published, and installed.**

<sub>A dark-matter web registry with a glowing glass surface — Express + MongoDB at the core, Google Drive as the vault, and a `qpm install` away from anywhere.</sub>

<br/>

<code>qpm&nbsp;install&nbsp;quantum-core</code>

<br/><br/>

![Node.js](https://img.shields.io/badge/Node.js-18%2B-5FA04E?logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express-4-000000?logo=express&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose%208-47A248?logo=mongodb&logoColor=white)
![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=black)
![Vite](https://img.shields.io/badge/Vite-5-646CFF?logo=vite&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-6366f1)

</div>

<br/>

<div align="center">

[The aura](#-the-aura) ·
[Architecture](#-architecture) ·
[Layout](#-repository-layout) ·
[Quick start](#-quick-start) ·
[Environment](#-environment) ·
[API](#-api-reference) ·
[Publishing](#-publishing-a-package) ·
[Data model](#-data-model) ·
[Roadmap](#-roadmap)

</div>

---

## ✦ The aura

QPM is the package backbone of the **Quantum Language** ecosystem — think `npm`, tuned for Quantum and wrapped in the QuantumLogics house style.

|  | |
|---|---|
| **Registry, not just a site** | Package documents are served in an npm-compatible shape (`dist-tags`, `versions`, `dist.tarball`), so any npm-style client — including the `qpm` CLI — can read straight from it. |
| **Drive as the vault** | Every published `.tgz` streams into Google Drive over OAuth and streams back out on install. No object-storage bill, no extra infra. |
| **Metadata that moves fast** | Names, versions, keywords, dependency graphs, and download counts live in MongoDB behind text indexes. |
| **Guests welcome, authors protected** | Publish anonymously to the community namespace, or sign in and the registry locks the package name to your account. |
| **The surface** | React + Vite front end: deep-space background, indigo → cyan → violet glow, frosted-glass cards, `Fira Code` terminal panels, and a burst of confetti the moment a package goes live. |

> **Palette** &nbsp;`#6366f1` indigo · `#06b6d4` cyan · `#a855f7` violet · `#10b981` emerald  
> **Type** &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`Outfit` / `Plus Jakarta Sans` / `Fira Code`

---

## ⚙ Architecture

```
   qpm CLI  ·  browser
        │
        ▼
┌───────────────────┐        /api        ┌────────────────────────┐
│   Frontend        │ ─────────────────▶ │   Backend              │
│   React 18 + Vite │                    │   Express (ESM)        │
│   :3000 (proxy)   │ ◀───────────────── │   :8000                │
└───────────────────┘                    └───────────┬────────────┘
                                                     │
                        ┌────────────────────────────┼────────────────────────────┐
                        ▼                            ▼                            ▼
                ┌───────────────┐          ┌──────────────────┐         ┌──────────────────┐
                │   MongoDB     │          │  JWT + bcryptjs  │         │  Google Drive    │
                │  package &    │          │  auth / identity │         │  .tgz tarballs   │
                │  user metadata│          │                  │         │  (OAuth stream)  │
                └───────────────┘          └──────────────────┘         └──────────────────┘
```

| Layer | Stack |
|-------|-------|
| **Frontend** | React 18 · Vite 5 · React Router 6 · `lucide-react` · `canvas-confetti` |
| **Backend**  | Node.js + Express 4 (ESM) · Mongoose 8 · `jsonwebtoken` · `bcryptjs` · `multer` |
| **Data**     | MongoDB — metadata · Google Drive API — package archives |
| **Auth**     | JWT (7-day tokens) · bcrypt password hashing |

---

## 📂 Repository layout

A meta-repo that stitches two services together as **git submodules**:

```
qpm/
├── backend/    → QuantumLogicsLabs/QPM-Website-Backend    · Express registry API
└── frontend/   → QuantumLogicsLabs/QPM-Website-Frontend   · React registry UI
```

```bash
# clone with submodules
git clone --recurse-submodules https://github.com/QuantumLogicsLabs/QPM-Website.git qpm

# already cloned flat?
git submodule update --init --recursive
```

---

## 🚀 Quick start

### 1 · Backend — the registry API

```bash
cd backend
npm install
cp .env.example .env      # fill in the values from the table below
npm run dev               # node --watch server.js  →  http://localhost:8000
```

The API boots even with an empty `.env`:

- **No `MONGO_URI`?** It falls back to `mongodb://127.0.0.1:27017/qpm_registry`, and keeps running if Mongo is offline.
- **No Google credentials?** Uploads return mock Drive IDs, so the full publish flow works locally without touching Google.

Health check → `GET http://localhost:8000/api/health`

### 2 · Frontend — the registry UI

```bash
cd frontend
npm install
npm run dev               # vite  →  http://localhost:3000
```

The dev server proxies `/api` → `http://localhost:8000`, so run the backend alongside it.  
Production build → `npm run build` (outputs to `dist/`).

---

## 🔑 Environment

`backend/.env`

| Key | Required | Notes |
|-----|:--------:|-------|
| `PORT` | – | Defaults to `8000`; the Vite proxy expects this. |
| `MONGO_URI` | ★ | MongoDB connection string. Falls back to a local instance. |
| `JWT_SECRET` | ★★ | Signs auth tokens. A hardcoded dev fallback exists — **always override it in production.** |
| `GOOGLE_OAUTH_CLIENT_ID` | Drive | From a Google Cloud OAuth 2.0 client. |
| `GOOGLE_OAUTH_CLIENT_SECRET` | Drive | Paired with the client ID. |
| `GOOGLE_OAUTH_REFRESH_TOKEN` | Drive | Long-lived — mint it once with the helper script below. |
| `GOOGLE_DRIVE_FOLDER_ID` | Drive | The folder every tarball is uploaded into. |

<sub>★ recommended · ★★ required in production · Drive = required for real tarball storage</sub>

<details>
<summary><b>Wiring up Google Drive storage</b></summary>

<br/>

1. In the [Google Cloud Console](https://console.cloud.google.com/), enable the **Google Drive API** and create an **OAuth 2.0 Client ID** (Desktop app). Put the client ID / secret in `.env`.
2. Mint a refresh token — run once, approve the consent screen, paste the result into `.env`:
   ```bash
   cd backend
   node src/scripts/get-refresh-token.js
   ```
3. Create a Drive folder for packages, open it, and copy the ID from the URL
   (`drive.google.com/drive/folders/`**`<THIS_PART>`**) into `GOOGLE_DRIVE_FOLDER_ID`.

Refresh tokens don't expire unless revoked — this is a one-time setup.

</details>

---

## 📡 API reference

Base URL — `http://localhost:8000/api`

| Method | Endpoint | Auth | Purpose |
|:------:|----------|:----:|---------|
| `GET`  | `/health` | – | Service, database, and storage status |
| `POST` | `/auth/signup` | – | Register `{ username, email, password, bio? }` → JWT |
| `POST` | `/auth/login` | – | Login `{ emailOrUsername, password }` → JWT |
| `GET`  | `/auth/me` | Bearer | The authenticated user |
| `GET`  | `/registry/search?q=<term>` | – | Search by name, description, or keyword (top 50 by downloads) |
| `GET`  | `/registry/:name` | – | npm-compatible package document (`dist-tags`, `versions`, `time`) |
| `GET`  | `/registry/:name/-/:file` | – | Download the `.tgz` (streamed from Drive; bumps the download counter) |
| `GET`  | `/registry/user/my-packages` | Bearer | Packages owned by the caller |
| `POST` | `/registry/publish` | Optional Bearer | Publish a version — `multipart/form-data` (`file`) or JSON (`fileBase64`) |

---

## 📦 Publishing a package

**As a guest** — ownership goes to the community namespace:

```bash
curl -X POST http://localhost:8000/api/registry/publish \
  -F "name=quantum-utils" \
  -F "version=1.0.0" \
  -F "description=Utility functions for Quantum Language" \
  -F "keywords=quantum,utils,math" \
  -F "file=@quantum-utils-1.0.0.tgz"
```

**As an author** — add your token and the name is locked to your account (re-publishing under a name owned by someone else is rejected with `403`):

```bash
curl -X POST http://localhost:8000/api/registry/publish \
  -H "Authorization: Bearer <YOUR_JWT>" \
  -F "name=quantum-utils" -F "version=1.1.0" \
  -F "file=@quantum-utils-1.1.0.tgz"
```

Grab your token from the **Profile** page in the UI, or from any `login` / `signup` response. Every `version` must be unique within a package.

---

## 🗃 Data model

```
User ──owns──▶ Package ──has many──▶ PackageVersion
```

| Entity | Key fields |
|--------|-----------|
| **User** | `username` (unique) · `email` (unique) · `passwordHash` · `bio` · `avatarUrl` |
| **Package** | `name` (unique, lowercased) · `description` · `keywords[]` · `license` · `repository` · `homepage` · `readme` · `latestVersion` · `downloads` · `owner → User` |
| **PackageVersion** | `package → Package` · `version` · `dependencies` (map) · `readme` · `tarballName` · `driveFileId` · `fileSize` · `shasum` · `publishedBy → User` — unique on `(package, version)` |

---

## 🔗 Registry compatibility

`GET /api/registry/:name` returns the same top-level shape an npm-style client expects:

```jsonc
{
  "name": "quantum-core",
  "dist-tags": { "latest": "2.1.0" },
  "versions": {
    "2.1.0": {
      "name": "quantum-core",
      "version": "2.1.0",
      "dependencies": { "quantum-math": "^1.0.0" },
      "dist": {
        "tarball": "http://localhost:8000/api/registry/quantum-core/-/quantum-core-2.1.0.tgz",
        "shasum": "verified",
        "unpackedSize": 20480
      }
    }
  },
  "time": { "2.1.0": "2026-08-29T12:00:00.000Z" }
}
```

That makes the `qpm` CLI a thin client: resolve name → read `dist.tarball` → fetch → unpack.

---

## 🛣 Roadmap

- [ ] Real tarball `shasum` / integrity verification on publish
- [ ] Scoped packages (`@scope/name`)
- [ ] Yank / deprecate a version
- [ ] Per-package README rendering (Markdown → HTML) in the UI
- [ ] Public download analytics
- [ ] First-class `qpm` CLI in this workspace

---

## 📄 License

[MIT](LICENSE) © QuantumLogicsLabs
