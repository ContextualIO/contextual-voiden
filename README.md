# contextual-voiden

A [Voiden](https://voiden.md) workspace for calling the Contextual platform APIs. Each request is a plain-text `.void` file, so you can browse, diff and review the requests like any other file in git.

This repo has 180+ requests across four APIs:

| Folder | API | Base URL variable |
| --- | --- | --- |
| [Contextual/Auth-API](Contextual/Auth-API) | Organizations, users, roles, clients, client grants, invitations, resource servers | `{{authUrl}}` |
| [Contextual/Logs-API](Contextual/Logs-API) | Logs, sessions, backlog | `{{logsUrl}}` |
| [Contextual/Native-Object-API](Contextual/Native-Object-API) | Agents, connections, flows and generic `[typeId]` records (CRUD, query, page, actions, audit trail, versions, scheduled actions, templates) | `{{noUrl}}` |
| [Contextual/Native-Object-Registry-API](Contextual/Native-Object-Registry-API) | Object type definitions and their versions | `{{noRegistryUrl}}` |

The requests were imported from [Contextual.postman_collection.json](Contextual.postman_collection.json), which is kept here as the source of truth.

---

## 1. Install Voiden

Voiden is a free desktop API client that works offline. It runs on macOS, Windows and Linux.

**macOS (Homebrew)**

```sh
brew install --cask voiden
```

**Any OS:** download the installer for your platform from <https://voiden.md/download>. On macOS, choose the Apple Silicon or Intel build to match your machine.

Full install docs: <https://docs.voiden.md/docs/getting-started-section/getting-started/installation/>

## 2. Open the workspace

```sh
git clone <this-repo-url> contextual-voiden
```

Then in Voiden, choose **Open Folder** and select the `contextual-voiden` directory. The request tree in the sidebar matches the `Contextual/` folder layout.

### Lock the project every time you launch Voiden

The `.void` files in this repo are shared request templates. Normally Voiden saves every edit straight to disk, so filling in an ID, changing the `x-org-id` header or pasting a test body would change the shared file and appear in `git status`.

To avoid that, **always work in [Lock Mode](https://docs.voiden.md/docs/core-features-section/lock-mode/lock-mode-overview/)**. Each time you launch Voiden and open this project:

1. Look at the **lock icon** in the status bar at the bottom of the Voiden window.
2. If it isn't active, click it to lock the project.
3. Use Voiden as usual. You can still edit requests, switch environments and send requests, but Voiden keeps everything in memory and writes nothing to disk, including response history.

Your per-request edits are discarded when you unlock or close Voiden, and the `.void` files stay exactly as they are in git.

A new clone always opens unlocked, because the lock state isn't committed. Voiden saves it locally in `.voiden/.voiden-projects`, which is gitignored. Voiden may remember your lock between launches, but check the icon every time anyway. It's the only thing between a quick test and a dirty working tree.

Unlock only when you want to change a shared request on purpose. See [Adding or changing requests](#adding-or-changing-requests).

## 3. Choose an environment

The environments are committed in [.voiden/env-public.yaml](.voiden/env-public.yaml), so they're ready as soon as you open the folder. Choose one in Voiden's environment selector:

| Environment | Stack | Auth0 tenant |
| --- | --- | --- |
| **[001] Dev-001** | `dev-001.dev.contextual.io` | `contextual-dev.us.auth0.com` |
| **[101] QA-001** | `qa-001.qa.contextual.io` | `contextual-qa.us.auth0.com` |
| **[201] Prod-001** | `prod-001.prod.contextual.io` | `contextual.us.auth0.com` |

Every request builds its URL from `{{domainSuffix}}`, so switching environments points all requests at that stack. The shared base URLs (`noUrl`, `noRegistryUrl`, `authUrl`, `logsUrl`) are defined once, at the top of the file.

**Personal or secret values** (tokens, client secrets, your own org ID) go in `.voiden/env-private.yaml`. That file is gitignored, as is everything else in `.voiden/` except the public env files. Never put secrets in `env-public.yaml`.

> ⚠️ **Prod-001 is live production.** Requests that create, update or delete data (`Create-*`, `Update-*`, `Patch-*`, `Delete-*`, `Execute-*`, `Clear-backlog`, `Flush-all-sessions`) take effect immediately. Double-check the selected environment before you send one.

## 4. Sign in

All requests use **OAuth 2.0 Authorization Code** against Auth0. The auth is set up once in [Contextual/.voiden-inherited.void](Contextual/.voiden-inherited.void), which is committed, and every request under `Contextual/` inherits it. There's nothing to configure. You only need to get a token:

1. Open the inherited auth for `Contextual/`, or any request, which shows the inherited auth.
2. Click **Get token**. A browser window opens and you sign in with your Contextual account.
3. Voiden stores the token, sends it as `Authorization: Bearer …` on every request, and refreshes it automatically.

The auth fields use environment variables (`{{authDomain}}`, `{{clientId}}`, `{{audience}}`), so the same config works for every environment. Get a new token after you switch environments.

For reference, the committed config is:

| Field | Value |
| --- | --- |
| Grant type | Authorization Code |
| Auth URL | `https://{{authDomain}}/authorize?audience={{audience}}` |
| Token URL | `https://{{authDomain}}/oauth/token` |
| Client ID | `{{clientId}}` |
| Client secret | *(none; public client)* |
| Callback URL | `http://localhost:9090/callback` |
| Add token to | Header, prefix `Bearer` |

The callback listens on `localhost:9090`, so make sure nothing else is using that port when you sign in.

## 5. Send a request

1. In the sidebar, open a request such as `Contextual/Auth-API/Users-Current/Get-current-user.void`.
2. **Path parameters:** folders named in square brackets (`[organizationId]`, `[clientId]`, `[id]`, `[typeId]` …) contain requests with path variables. Fill them in from the request's path-params table, where they show as `—` until you set them.
3. **`x-org-id` header:** Native Object API requests send an `x-org-id` header, currently set to `nasser`. Change it to your own org/tenant ID before you send.
4. **Body:** `Create-*`, `Update-*`, `Query-*` and `Page-*` requests have a JSON body block. Query and page requests default to `{}`.
5. Press **Send**.

Because you're in Lock Mode, the values you fill in for steps 2–4 aren't saved to the `.void` file. Nothing shows up in `git status`, and you start from clean templates next session. If you notice modified `.void` files you didn't mean to change, you were probably unlocked. Revert them with `git restore Contextual/`.

A good first check is **Auth-API → Users-Current → Get-current-user**. It confirms that your token and environment are working.

## Repo layout

```
.
├── Contextual/                      # Voiden requests (.void files), one per endpoint
│   ├── .voiden-inherited.void       # shared OAuth2 config, inherited by every request
│   ├── Auth-API/
│   ├── Logs-API/
│   ├── Native-Object-API/
│   └── Native-Object-Registry-API/
├── .voiden/
│   ├── env-public.yaml              # Dev / QA / Prod environments (committed)
│   ├── env-private.yaml             # your secrets and overrides (gitignored)
│   └── history/                     # local response history, unlocked sessions only (gitignored)
└── Contextual.postman_collection.json   # original Postman export
```

## Adding or changing requests

- **Unlock first.** Changes only save while Lock Mode is off. Turn it off, make the change in Voiden (it writes straight to the `.void` file), then **turn Lock Mode back on**. Voiden can remember the unlocked state between launches, so if you forget, later sessions will save edits to disk. Edits you made while locked are not kept, so redo them after unlocking.
- **Keep templates generic.** Before committing, make sure the file has no IDs, org names or bodies from a one-off test, and leave placeholders such as `—` in path params. Commit only the change you meant to make.
- **New endpoint:** create the `.void` file in the matching API/resource folder, use the existing base-URL variables (`{{noUrl}}`, `{{authUrl}}` …) and don't hard-code hostnames.
- **Re-import from Postman:** if the Postman collection changes, replace `Contextual.postman_collection.json` and re-import it with Voiden's Postman import. Check the diff carefully, because a re-import regenerates the `uid`s in every file.
- **Environments and auth:** changes to `.voiden/env-public.yaml` or `Contextual/.voiden-inherited.void` affect everyone, so review them like any shared config.
- **Secrets:** never commit tokens, client secrets or anything in `env-private.yaml`.
