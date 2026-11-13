# Security

## Reporting a vulnerability

Please **don't open a public issue** for security problems. Use GitHub's **Report a vulnerability** (Security → Advisories) on this repository, or contact the maintainer privately. Include steps to reproduce and the affected component (`plugin/`, `block/` or `backend/`).

## How credentials are handled

- **No secrets in this repository.** All credentials (Box, Google, Adobe server-to-server, `TOKEN_ENCRYPTION_KEY`, `BRIDGE_TOKEN`) are App Builder action inputs, read from a local `.env` at deploy time. `.env` is git-ignored, and `backend/.env.example` contains names only.
- **Never paste credentials into chats, issues or pull requests.** If a token was exposed, treat it as compromised and rotate it:
  - **Box:** reset the Client Secret.
  - **Google:** reset the OAuth client secret.
  - **Adobe:** rotate the credential's client secret.
  - **`aio login` tokens:** run `aio logout` and then `aio login` again.
- **Author tokens:** the plugin sends the author's Experience Workspace IMS token to the configured backend only. The backend validates it with IMS and never stores it.
- **Third-party tokens:** Box, Google and Firefly tokens are stored AES-256-GCM encrypted per author in App Builder State and are never returned to the browser.
- **Local bridge (demo mode):** it keeps its Adobe sign-in in `~/.ccs-bridge/firefly.json` (mode 600) on the operator's computer. Delete that file and remove `BRIDGE_TOKEN` when you stop using bridge mode.

## Scope notes

- Box and Google access is read-only and limited to one allowed folder tree.
- CORS is limited to `ALLOWED_ORIGINS`. Action inputs are `final`.
- Published storefront pages contain only the SKU, the authored copy and the image URLs.
