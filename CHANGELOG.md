# Changelog

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
