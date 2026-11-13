# Creative Commerce Studio

**Turn product masters in Box or Google Drive into shoppable, placement-ready storefront banners without leaving Adobe Experience Workspace.**

An author picks a product by SKU, chooses an authorized packshot from Box or Google Drive, and runs a published **Firefly Creative Production** workflow that produces desktop and mobile compositions. They review the result, then insert a **Creative Product Spotlight** block at the cursor. On the storefront, the block links to the live **Adobe Commerce** product.

> [!WARNING]
> **This is a working demo, not a production-ready integration.**
> - **Firefly runs:** they go through a **local bridge** on one operator's computer, with that operator's own Adobe sign-in, because no Firefly Services API credential was available.
> - **Image storage:** approved images go to App Builder public storage rather than Product Visuals (AEM Assets).
> - **Google Drive:** it runs as an unverified app in Testing mode.
>
> Everything needed to make it production-ready (APIs, credentials, roles, entitlements and approvals) is listed in [Production readiness](#production-readiness) below and in detail in **[docs/PRODUCTION-READINESS.md](docs/PRODUCTION-READINESS.md)**.

---

## Contents

- [What it does](#what-it-does)
- [How it works](#how-it-works)
- [Repository layout](#repository-layout)
- [Requirements](#requirements)
  - [Application requirements](#application-requirements)
  - [Access requirements (to run it)](#access-requirements-to-run-it)
- [Production readiness](#production-readiness)
- [Install](#install)
- [Using it](#using-it)
- [Configuration reference](#configuration-reference)
- [Security](#security)
- [Development](#development)
- [Known limitations](#known-limitations)
- [License](#license)

## What it does

The Workspace library plugin walks the author through five steps:

| Step | What happens |
|---|---|
| **Product** | Exact-SKU lookup against the site's own Commerce catalog endpoint and store context. Shows name, type, stock and the product page link. |
| **Source** | Pick **Box** or **Google Drive** and connect your own account. Browse one allowed campaign folder with thumbnails, choose a PNG or JPEG packshot, and **confirm** that it shows the product and that you may use it. |
| **Prepare** | Runs the published Firefly Creative Production workflow on the **exact file version** you chose: background removal, then desktop and mobile composites. A progress bar shows while it runs. |
| **Review** | Preview both placements. Edit headline, description, CTA label and alt text. Choose a **static** or **animated** presentation. **Accept** stores the images permanently. |
| **Insert** | Inserts a `creative-product-spotlight` block at the cursor. Nothing is published; the author saves and previews as usual. |

On the storefront the block renders the responsive image, which is never cropped, plus the editable HTML copy and a native CTA. The CTA's product link is **resolved from Commerce at runtime**. If the SKU no longer exists, the CTA is hidden instead of linking to a missing product. The optional animated presentation is a one-time entrance, about 1.3 s, and is disabled for visitors who prefer reduced motion.

## How it works

```
Experience Workspace (da.live)
 └─ Library plugin (plugin/, hosted on the site at /tools/creative-commerce-studio/)
     ├─ Commerce catalog ─────────────► storefront GraphQL endpoint + headers from the site's config.json
     ├─ Box / Google Drive ───┐
     ├─ production jobs ──────┼──────► App Builder web actions (backend/)
     │                        │          ├─ author identity: Experience Workspace IMS token, validated with IMS
     │                        │          ├─ Box + Google Drive OAuth (PKCE), encrypted per-author tokens, folder allow-list
     │                        │          ├─ job records (App Builder State) + working files (App Builder Files)
     │                        │          ├─ Firefly Creative Production
     │                        │          │    ├─ api:    REST Workflow API, server-to-server credential   ← production
     │                        │          │    └─ bridge: local script on the operator's computer         ← demo
     │                        │          └─ delivery: Product Visuals / AEM Assets (aem) or App Builder public storage (public)
     └─ insert ───────────────────────► Creative Product Spotlight block (block/, in the site's blocks/)
                                          └─ storefront: live product link via the site's Commerce catalog client
```

The secrets (Box, Google and Adobe credentials, and the encryption key) exist only as App Builder action inputs. The browser never sees a third-party token, and published pages contain only the SKU, the copy and the image URLs.

## Repository layout

| Path | What it is | Where it goes |
|---|---|---|
| [`plugin/`](plugin/) | Experience Workspace **library plugin** (vanilla JS + Spectrum Web Components, Spectrum 2 theme, pre-bundled in `plugin/vendor/`) | Your site repo: `tools/creative-commerce-studio/` |
| [`block/creative-product-spotlight/`](block/creative-product-spotlight/) | The **Creative Product Spotlight** EDS block | Your site repo: `blocks/creative-product-spotlight/` |
| [`backend/`](backend/) | **App Builder** project: web actions, tests, the local Firefly bridge | Deployed with `aio app deploy` |
| [`docs/PRODUCTION-READINESS.md`](docs/PRODUCTION-READINESS.md) | What production use requires, per service | — |
| [`docs/FFCPE-WORKFLOW.md`](docs/FFCPE-WORKFLOW.md) | How to build and publish the Firefly workflow | — |
| [`backend/README.md`](backend/README.md) | Detailed operator guide: every service's setup, deploy, troubleshooting | — |

## Requirements

### Application requirements

| Component | Required? | Used for | Notes |
|---|---|---|---|
| **Adobe Experience Workspace** site on **Edge Delivery Services** (Document Authoring) | Required | Hosts the library plugin; authors insert the block | The plugin uses the Workspace SDK (`https://da.live/nx/utils/sdk.js`) |
| **AEM Boilerplate Commerce** storefront | Required | The block imports `CS_FETCH_GRAPHQL` and `getProductLink` from the site's `scripts/commerce.js` | Any site based on the Commerce boilerplate |
| **Adobe Commerce** catalog: **Adobe Commerce as a Cloud Service** or **Adobe Commerce Optimizer** | Required | SKU lookup and the live product link (`products(skus:)` storefront query) | Tested with Adobe Commerce Optimizer. Uses the site's public `commerce-endpoint` and headers. No admin or catalog writes |
| **Adobe App Builder** (Developer Console project and workspace, I/O Runtime, State, Files) | Required | The backend | Node 22, `aio` CLI |
| **Box** (OAuth 2.0 "User Authentication" app) | At least one of Box / Drive | Source packshots | Free developer accounts work |
| **Google Drive** (Google Cloud OAuth client, Web application, Drive API) | At least one of Box / Drive | Source packshots | Testing-mode OAuth for demos |
| **Firefly Creative Production** | Required | Image production | A **published workflow** with one product-image input and desktop and mobile composite outputs ([docs/FFCPE-WORKFLOW.md](docs/FFCPE-WORKFLOW.md)) |
| **Product Visuals** (AEM Assets as a Cloud Service) with **Dynamic Media with OpenAPI** | Optional (recommended for production) | Durable, governed image delivery | Without it, images go to App Builder public storage |
| Authorized creative | Required | Packshots, backgrounds and templates you are allowed to use and publish | — |

### Access requirements (to run it)

These are the accounts and roles needed to run the extension as it is today, in demo mode. What production needs comes in the next section.

| Who | Needs |
|---|---|
| **Site admin** | Write access to the site's code repo (adds the block, plugin and one config value). Access to the Workspace site config to add a **library** row. |
| **Backend operator** | An Adobe org with **App Builder**, the **Developer** role in that org, a Developer Console project and workspace, and Node 22 with the `aio` CLI. |
| **Box admin** | Creates the Box OAuth app. In enterprise Box, approves it under **Admin Console → Platform Apps**, added by Client ID. |
| **Google admin** | A Google Cloud project with the **Drive API** enabled, and an OAuth consent screen (External, Testing) with authors listed as **test users**. |
| **Firefly operator (bridge)** | Firefly Creative Production **user access** that can run the published workflow, and a computer running `npm run bridge` during use. All runs use this operator's identity. |
| **Authors** | Edit access to the Workspace site, plus a Box and/or Google account with access to the campaign folder. |

## Production readiness

> [!IMPORTANT]
> The code for the production path is already built and tested (`FFCPE_MODE=api`, `DELIVERY_MODE=aem`). What's missing is **access**. The table lists the Adobe Developer Console APIs, credentials and roles each service needs. Full checklist: **[docs/PRODUCTION-READINESS.md](docs/PRODUCTION-READINESS.md)**.

| Service | Developer Console API / credential | Entitlement & roles | Approvals & other work |
|---|---|---|---|
| **Firefly Creative Production** (replaces the local bridge) | **Firefly Services: Creative Production Workflow API** with an **OAuth Server-to-Server** credential (scopes `openid, AdobeID, session, additional_info, read_organizations, firefly_api, ff_apis`). REST contract: `https://run-workflow.adobe.io/openapi.json`. | Firefly Services **API** entitlement in the org that owns the workflows; browser access isn't enough. The **Developer** role on that product profile, granted by a System or Product Admin. | The credential's **technical account must be allowed to run the published workflow**. Confirm quota, rate limits and billing. |
| **Product Visuals / AEM Assets** | **AEM** API access (Assets Author API) with an **OAuth Server-to-Server** credential | AEM as a Cloud Service with **Dynamic Media with OpenAPI**. The **Developer** role in the AEM org. | The AEM environment must **allow the technical account** (Cloud Manager API access configuration), with write and approve rights on the target DAM folder. Agree an approval policy. |
| **App Builder** | App Builder project with a **Production** workspace (I/O Runtime, State, Files) | App Builder entitlement and the **Developer** role in the hosting org | Deploy from CI/CD, use managed secrets with rotation, monitoring, and a data-retention plan. Remove `BRIDGE_TOKEN`. |
| **Identity (Experience Workspace)** | None. Author tokens are validated with IMS. | — | Confirm the Workspace IMS client ID (`IMS_ALLOWED_CLIENT_IDS`, currently `darkalley`). Restrict to customer orgs with `IMS_ALLOWED_ORGS`. |
| **Box** | Box OAuth app (read-only scope) | — | **Enterprise admin approval** in every customer's Box. |
| **Google Drive** | Google OAuth client | — | `drive.readonly` is a **restricted scope**: **Google app verification plus an annual CASA security assessment** (weeks). Alternatively switch to `drive.file` + Google Picker (a code change). Google Workspace admins must allow the app. |
| **Marketplace / distribution** | — | — | Confirm the Experience Workspace marketplace listing and review requirements, per-customer configuration, and a security and privacy review. |

Each service's credential must come from a Developer Console project in **the org that owns that service**. Workspace, App Builder, Firefly and AEM may each be in a different IMS org, and one backend can hold credentials from several of them.

## Install

The short version is below. Every step is detailed in **[backend/README.md](backend/README.md)**.

1. **Create the App Builder project**: Developer Console → *Create project from template* → **App Builder**. Note the workspace's **Runtime namespace**; your backend URL will be `https://<namespace>.adobeioruntime.net/api/v1/web/creative-commerce-studio`.
2. **Set up a source.** For **Box**: an OAuth 2.0 app with redirect `<backend>/box-auth-callback` and a campaign folder ID. For **Google Drive**: an OAuth client (Web) with redirect `<backend>/gdrive-auth-callback`, the Drive API enabled, test users, and a folder ID. Set up one or both.
3. **Build and publish the Firefly workflow** ([docs/FFCPE-WORKFLOW.md](docs/FFCPE-WORKFLOW.md)), and note the workflow ID and the product, desktop and mobile node IDs.
4. **Configure and deploy the backend:**
   ```bash
   cd backend
   npm install -g @adobe/aio-cli
   aio login
   aio app use --global          # or: aio app use <workspace.json>; choose "merge" for .env
   # append the variables from .env.example to the .env that aio created, and fill them in
   npm install && npm test
   aio app deploy --force-deploy
   ```
5. **Add the block and the plugin to your site repo**:
   - copy `block/creative-product-spotlight/` to `blocks/creative-product-spotlight/`
   - copy `plugin/` to `tools/creative-commerce-studio/`; `plugin/build/` is optional, and if you keep it, add `tools/creative-commerce-studio/build/*` to `.hlxignore`
   - in the site's `config.json` (`public.default`), add:
     ```json
     "creative-commerce-studio": { "backend-url": "https://<namespace>.adobeioruntime.net/api/v1/web/creative-commerce-studio" }
     ```
6. **Allow the plugin's origins** in the backend's `ALLOWED_ORIGINS`: `https://da.live`, `https://<branch>--<site>--<org>.preview.da.live` (Workspace serves plugins through this proxy), and your `.aem.page` and `.aem.live` hosts. Redeploy.
7. **Register the plugin**: Workspace site config `https://da.live/config#/<org>/<site>/` → **library** sheet, add a row with title `Creative Commerce Studio` and path `https://main--<site>--<org>.aem.page/tools/creative-commerce-studio/creative-commerce-studio.html`.
8. **Firefly (demo mode)**: on the operator's computer, in `backend/`, run `npm run bridge:login` once, then keep `npm run bridge` running during use. For production, use `FFCPE_MODE=api` with the server-to-server credential instead.

## Using it

1. Open a page in Experience Workspace, then **Library → Creative Commerce Studio**.
2. **Product**: enter an exact SKU and **Look up**.
3. **Source**: choose **Box** or **Google Drive**, connect, open the folder, pick a packshot, and tick the confirmation.
4. **Prepare**: **Run production**. With the bridge, the run starts as soon as the bridge picks it up. The progress bar and elapsed time show while Firefly works, which can take up to about 10 minutes.
5. **Review**: check both placements, edit the copy, choose **Static** or **Animated entrance**, then **Accept and deliver**.
6. **Insert**: **Insert into page**, then save and preview the page as usual.

The block can also be authored by hand. It's a two-column key/value table with these rows: `SKU`, `Headline`, `Description`, `Desktop image`, `Mobile image`, `Image alt`, `CTA label`, `Presentation` (`static` | `animated`). See [block/creative-product-spotlight/README.md](block/creative-product-spotlight/README.md).

## Configuration reference

All settings are App Builder action inputs, read from `backend/.env` at deploy time. Names and placeholders are in [`backend/.env.example`](backend/.env.example). After changing only `.env`, deploy with `aio app deploy --force-deploy`.

| Setting | Required | Purpose |
|---|---|---|
| `ALLOWED_ORIGINS` | yes | Origins allowed to call the actions, including the Workspace `…preview.da.live` proxy |
| `IMS_ALLOWED_CLIENT_IDS` | yes | IMS client IDs whose author tokens are accepted (`darkalley` for Experience Workspace) |
| `IMS_ALLOWED_ORGS` | no | Restrict use to these IMS orgs |
| `TOKEN_ENCRYPTION_KEY` | yes | 32 random bytes, base64. Encrypts stored Box, Google and Firefly tokens |
| `BOX_CLIENT_ID`, `BOX_CLIENT_SECRET`, `BOX_REDIRECT_URI`, `BOX_ALLOWED_FOLDER_ID` | for Box | Box OAuth app and campaign folder |
| `GDRIVE_CLIENT_ID`, `GDRIVE_CLIENT_SECRET`, `GDRIVE_REDIRECT_URI`, `GDRIVE_ALLOWED_FOLDER_ID` | for Drive | Google OAuth client and campaign folder |
| `FFCPE_MODE` | yes | `bridge` (demo), `api` (production) or `mcp` (hosted MCP sign-in, refused by Adobe today) |
| `FFCPE_WORKFLOW_ID`, `FFCPE_PRODUCT_NODE_ID`, `FFCPE_DESKTOP_NODE_ID`, `FFCPE_MOBILE_NODE_ID` | yes | Published workflow and its node IDs. Outputs are matched by node ID |
| `BRIDGE_TOKEN`, `CCS_BACKEND_URL` | bridge mode | Shared secret between the backend and the local bridge; backend URL for the bridge |
| `FFCPE_CLIENT_ID`, `FFCPE_CLIENT_SECRET`, `FFCPE_IMS_ORG_ID`, `FFCPE_IMS_USER_ID`, `FFCPE_SCOPES`, `FFCPE_API_BASE`, `FFCPE_EXECUTE_TEMPLATE` | api mode | Server-to-server credential and REST settings |
| `DELIVERY_MODE` | no | `aem` (Product Visuals) or `public` (App Builder public storage); the default depends on whether `AEM_AUTHOR_URL` is set |
| `AEM_AUTHOR_URL`, `AEM_DELIVERY_URL`, `AEM_CLIENT_ID`, `AEM_CLIENT_SECRET`, `AEM_SCOPES`, `AEM_TARGET_FOLDER` | aem delivery | AEM Assets credential and target folder |

## Security

- **Authors:** each author is identified by their **Experience Workspace IMS token**, validated with IMS. All connections and jobs are keyed by that user, and there's no shared "current user".
- **Box, Google and Firefly tokens:** stored **AES-256-GCM encrypted** in App Builder State, bound to the user, and never sent to the browser. OAuth uses one-time state and PKCE.
- **Action inputs:** marked `final`, so requests can't override configuration. CORS is limited to `ALLOWED_ORIGINS`.
- **The plugin:** it reads the backend URL **only from the site's config**, never from its own URL, so an author's token can only go to the configured backend.
- **Source access:** read-only, and limited to one allowed folder tree per provider. Downloads use the exact file version or revision the author confirmed.
- **Published pages:** they contain no third-party links, tokens, job IDs or account IDs.
- **Reporting:** see [SECURITY.md](SECURITY.md).

## Development

```bash
cd backend && npm install && npm test        # backend unit tests (Jest, all services mocked)
cd plugin/build && npm install && npm run build   # rebuild plugin/vendor/spectrum.js after changing components
```

- The plugin and block are plain ES modules with no build step; the only bundle is the Spectrum Web Components file in `plugin/vendor/`.
- Opened outside Workspace, the plugin runs in a standalone preview: product lookup works, but sources and insertion are disabled.
- Workspace can load the plugin from `http://localhost:3000` with `ref=local` in the library row.

## Known limitations

- **Firefly:** demo mode depends on one operator's computer and sign-in (the local bridge). The hosted MCP sign-in is refused by Adobe for non-local apps.
- **Progress:** the bar reflects Firefly's reported steps when they're available; otherwise it shows an animated bar and the elapsed time.
- **Scope:**
  - one store view per install
  - exact-SKU lookup only
  - PNG and JPEG packshots only
  - one workflow recipe
- **Insertion:** insert-only. The plugin doesn't read or replace existing blocks.
- **Price:** not shown in the spotlight. The block links to the product page, where pricing uses the shopper's context.

## License

[Apache License 2.0](LICENSE).

Third-party components in `plugin/vendor/spectrum.js` keep their licenses; see `plugin/vendor/spectrum.js.LEGAL.txt`.
