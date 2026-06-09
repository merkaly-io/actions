# merkaly-io/actions

Reusable GitHub Actions for Merkaly repositories — setup, deploy, publish, and release.

## Actions

### `setup`

Checkout, enable corepack, set up Node 24 with pnpm cache, and install dependencies.

```yaml
- uses: merkaly-io/actions/setup@master
```

No inputs.

---

### `deploy`

Build and deploy to Railway, then record a GitHub deployment.

```yaml
- uses: merkaly-io/actions/deploy@master
  with:
    railway_project_id: ${{ vars.RAILWAY_PROJECT_ID }}
    railway_environment_id: ${{ vars.RAILWAY_ENVIRONMENT_ID }}
    railway_service_id: ${{ vars.RAILWAY_SERVICE_ID }}
    railway_token: ${{ secrets.RAILWAY_TOKEN }}
```

| Input | Required | Default | Description |
|---|---|---|---|
| `railway_project_id` | yes | — | Railway project UUID |
| `railway_environment_id` | yes | — | Railway environment UUID |
| `railway_service_id` | yes | — | Railway service UUID |
| `railway_token` | yes | — | Railway API token |
| `environment` | no | `production` | GitHub environment name shown in the deployment record |

**Required job permission:**
```yaml
permissions:
  deployments: write
```

**Flow:**
1. Builds the app (`pnpm build`)
2. Creates a GitHub deployment and marks it `in_progress`
3. Deploys to Railway (`railway up --ci`)
4. Resolves the Railway service URL (`railway domain`)
5. Updates the GitHub deployment to `success` (with `environment_url`) or `failure`

---

### `publish`

Validate that the tag matches `package.json` version, configure the npm registry, and publish.

```yaml
- uses: merkaly-io/actions/publish@master
```

No inputs. Requires `NODE_AUTH_TOKEN` to be set (via `actions/setup-node` registry-url or a secret).

---

### `release`

Generate a changelog from git history and publish a GitHub Release.

```yaml
- uses: merkaly-io/actions/release@master
```

No inputs. Requires `contents: write` permission on the calling job.

---

## Typical workflow

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      deployments: write
    steps:
      - uses: merkaly-io/actions/deploy@master
        with:
          railway_project_id: ${{ vars.RAILWAY_PROJECT_ID }}
          railway_environment_id: ${{ vars.RAILWAY_ENVIRONMENT_ID }}
          railway_service_id: ${{ vars.RAILWAY_SERVICE_ID }}
          railway_token: ${{ secrets.RAILWAY_TOKEN }}

  release:
    needs: deploy
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: merkaly-io/actions/release@master
```
