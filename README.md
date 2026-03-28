# Dokku Ready

`dokku ready` is the Dokku Ready plugin for interactive app provisioning and cleanup.

It is designed to reduce the repetitive Dokku setup flow to a single guided command.

## Commands

### `dokku ready:create <app>`

This command performs the following steps in order:

- creates the app
- prompts for the domain and applies it
- generates `<app>.<public-ip>.sslip.io` automatically if the domain is left empty
- prompts for the container port and maps `http:80:<port>` and `https:443:<port>`
- asks whether Postgres should be installed
- installs the official `postgres` plugin automatically if needed
- creates and links the Postgres service if enabled
- asks whether SSL should be enabled
- installs the official `dokku-letsencrypt` plugin automatically if needed
- if SSL is enabled, builds and deploys a temporary image on the Dokku host without requiring `git push`
- attempts SSL setup

Notes:

- If `postgres` or `dokku-letsencrypt` is missing, Dokku Ready attempts to install the official plugin automatically.
- Automatic plugin installation requires root privileges or passwordless `sudo`.
- Automatic `sslip.io` generation requires a detectable public IPv4 address or a preset `DOKKU_READY_PUBLIC_IP`.
- Let's Encrypt activation may fail until the app is deployed and DNS points to the Dokku server. In that case the plugin only shows a warning.
- The temporary bootstrap deploy exists only to make the app answer HTTP requests so certificate issuance can succeed.

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

Create an app and let Dokku Ready generate an `sslip.io` domain automatically:

```bash
dokku ready:create myapp
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
sudo dokku plugin:install https://github.com/hasan5151/dokku-ready.git --name ready
```

Verify the installation with:

```bash
dokku plugin:list
dokku ready:help
```

## Requirements

- Dokku must already be installed on the target server.
- Automatic plugin installation requires root privileges or passwordless `sudo`.
- Automatic `sslip.io` generation requires outbound network access or a preset `DOKKU_READY_PUBLIC_IP`.

## Environment Overrides

The following optional environment variables can be used to customize behavior:

```bash
DOKKU_READY_PUBLIC_IP=203.0.113.10
DOKKU_READY_DEFAULT_PORT=5000
DOKKU_READY_BOOTSTRAP_BASE_IMAGE=alpine:3.20
DOKKU_READY_BOOTSTRAP_IMAGE_PREFIX=dokku-ready-bootstrap
DOKKU_READY_POSTGRES_PLUGIN_URL=https://github.com/dokku/dokku-postgres.git
DOKKU_READY_LETSENCRYPT_PLUGIN_URL=https://github.com/dokku/dokku-letsencrypt.git
```
