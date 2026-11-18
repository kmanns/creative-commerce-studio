# Changelog

## 0.7.0 — 2026-11-18

- **New banner from the Insert step.** **New banner** starts over at Product, and **Another for <SKU>** keeps the product and goes to Source. Connections, the open folder and the workflow choice are kept, and the finished banner stays in Recent jobs. If the banner hasn't been inserted yet, the author is asked first. After inserting, the button reads **Insert again**.
- **README: "Before you start" comes first.** It covers a five-point requirements summary, the application and access requirements, and every value, credential and secret to fill in, with where to get each.
- Changing the product resets the image alt text to the new product's name.

## 0.6.2 — 2026-11-18

- **`npm run workflows:upload` goes through the deployed backend.** The new `workflows-upload-urls` action returns 10-minute write links, and only for the list and `looks/` images. It's protected by `ADMIN_TOKEN`, or by `BRIDGE_TOKEN` in bridge mode. The admin's computer no longer needs storage credentials, which could be refused with `ERROR_BAD_CREDENTIALS`. `--direct` keeps the old route.
- **`ADMIN_TOKEN` setting.** 83 tests.

## 0.6.1 — 2026-11-18

- **Unique scene image names** in the example and docs: `<look>-<channel>-<part>`, for example `looks/workshop/workshop-desktop-background.jpg`.
- **Scene checks** flag a file whose extension doesn't match its real type (renaming `.png` to `.jpg` doesn't convert it), with a fix hint. 77 tests.

## 0.6.0 — 2026-11-18

- **Workflows with scene images** (for example Campaign Looks). A workflow entry can list fixed `inputs`, such as backgrounds and placement masks, that the backend sends with every run from private App Builder storage. Each look is its own entry in the picker.
- **`workflows.json` plus `npm run workflows:upload` / `workflows:check`.** These validate the list and every scene image, then upload them, with no redeploy needed. `FFCPE_WORKFLOWS` still works and takes precedence.
- **Scene image checks before every run:** files present, PNG or JPEG, masks PNG, mask size equal to its background's.
- **Partial results.** Desktop and mobile succeed or fail on their own. The image that worked is kept and shown next to the failed channel's reason.
- **Output matching** also reads node IDs from the enclosing output (`outputs[].node_id` with a nested `content` link).
- **REST request body** follows the published workflow's generated API request: top-level `inputs`, `content: [{ sourceUrl, name }]`.
- **Local bridge:** sends every run input and announces it. Runs with scene images refuse older bridges with a clear message.
- **Tests:** 76.

## 0.5.0 — 2026-11-17

- **Workflow picker on the Prepare step.** Authors choose from an admin-curated list (`FFCPE_WORKFLOWS`), each entry with a name, description and steps. The backend validates the list and refuses unknown keys.
- **Per-job recipe.** Each job records its workflow and node IDs. The MCP route, the REST API, the local bridge and result matching all use the job's own workflow, and the Insert step shows which one made the images.
- **Duplicate-run protection** is now per workflow.
- **Backwards compatible.** Single-workflow installs (`FFCPE_WORKFLOW_ID`) keep working; the optional `FFCPE_WORKFLOW_LABEL` names that workflow.
- **New action** `workflows-list`, which returns labels, descriptions and steps but no node IDs.
- **Tests:** 61, including 8 new workflow tests.

## 0.4.1 — 2026-11-16

- README: screenshots of all five plugin steps from a real end-to-end run (`docs/images/`).
- Plugin: `icon.svg` for the Workspace library row and marketplace listing.
- Plugin: delivery-neutral wording. The Review step no longer says images always go to Product Visuals, and the failure alert reads "Delivery failed".
- Docs: what can and can't replace an AEM API credential (PRODUCTION-READINESS).
- Docs: "Choosing a workflow" (one admin-configured workflow today) and a Roadmap section with the planned admin-curated workflow picker.

## 0.4.0 — 2026-11-13 (contest submission)

First standalone release. Tested end to end in a reference install: SKU, then Box or Google Drive, then Firefly Creative Production through the local bridge, then review, delivery to App Builder public storage, and insertion into an Experience Workspace page.

### Workspace plugin
- Library plugin in five steps (Product → Source → Prepare → Review → Insert), built on Spectrum Web Components with the Spectrum 2 theme.
- Exact-SKU lookup against the site's Commerce catalog.
- **Box** and **Google Drive** sources, with a provider picker, thumbnails and confirmation of the exact file version.
- Production progress bar with elapsed time.
- Static or animated presentation.
- Clear messages for blocked sign-ins, an offline bridge and expired connections.

### Spotlight block
- Responsive images, never cropped, with per-breakpoint sizes.
- Brand-blue display headline and description in any section.
- Live Commerce product link; the CTA is hidden for unknown SKUs.
- One-time animated entrance (about 1.3 s) that respects reduced motion.

### Backend (App Builder)
- Authors identified by their IMS-validated Workspace token.
- Box and Google Drive OAuth with PKCE, encrypted per-author tokens and folder allow-lists.
- Exact-version and exact-revision downloads.
- Firefly modes:
  - `api`: REST Workflow API with a server-to-server credential, for production.
  - `bridge`: a local script with the operator's sign-in, for demos.
  - `mcp`: hosted MCP sign-in, currently refused by Adobe.
- Jobs are idempotent and never resubmitted automatically.
- Outputs are matched by workflow node ID.
- Delivery to Product Visuals (AEM Assets) or to App Builder public storage.
- Jest suite: 53 tests.

### Docs
- README covering requirements and access.
- `docs/PRODUCTION-READINESS.md`.
- Firefly workflow guide.
- Backend operator guide.
