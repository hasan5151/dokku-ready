# Dokku Ready

`dokku ready` is the Dokku Ready plugin for interactive app provisioning and cleanup.

## Commands

### `dokku ready:create <app>`

This command performs the following steps:

- prompts for the domain and applies it
- generates `<app>.<public-ip>.sslip.io` automatically if the domain is left empty
- creates the app
- prompts for the container port and maps `http:80:<port>` and `https:443:<port>`
- asks whether Postgres should be installed, then creates and links the service if enabled
- if SSL is enabled, builds and deploys a temporary image on the Dokku host without requiring `git push`
- attempts SSL setup

Notes:

- If `dokku-letsencrypt` is not installed, the SSL step is skipped.
- Automatic `sslip.io` generation requires a detectable public IPv4 address or a preset `DOKKU_READY_PUBLIC_IP`.
- Let's Encrypt activation may fail until the app is deployed and DNS points to the Dokku server. In that case the plugin only shows a warning.

### `dokku ready:delete <app>`

This command requires typing `yes` before deletion. After confirmation it:

- deletes the app with `apps:destroy --force`
- unlinks and destroys linked Postgres services
- attempts to disable SSL

## Examples

```bash
dokku ready:create blog
dokku ready:delete blog
```

Flags are also available for non-interactive usage:

```bash
dokku ready:create blog --domain blog.example.com --port 5000 --postgres --postgres-service blog-postgres --ssl --email ops@example.com
dokku ready:delete blog --force
```

## Installation

The `install` script automatically sets the required executable permissions.

To install this repository as a Dokku plugin from a local path:

```bash
sudo dokku plugin:install file:///absolute/path/to/dokkuready --name ready
```

To install it from a git repository:

```bash
sudo dokku plugin:install https://github.com/<user>/<repo>.git --name ready
```

Verify the installation with:

```bash
dokku plugin:list
dokku ready:help
```
