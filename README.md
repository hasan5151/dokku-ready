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
- if SSL is enabled, deploys a temporary bootstrap app on the Dokku host without requiring `git push`
- attempts SSL setup

Notes:

- If the app already exists, Dokku Ready continues and reconciles the requested state instead of failing immediately.
- If `postgres` or `dokku-letsencrypt` is missing, Dokku Ready attempts to install the official plugin automatically.
- Automatic plugin installation requires root privileges or passwordless `sudo`.
- Automatic `sslip.io` generation requires a detectable public IPv4 address or a preset `DOKKU_READY_PUBLIC_IP`.
- Let's Encrypt activation may fail until the app is deployed and DNS points to the Dokku server. In that case the plugin only shows a warning.
- The temporary bootstrap deploy exists only to make the app answer HTTP requests so certificate issuance can succeed.
- If the default Postgres service name already exists, Dokku Ready reuses it when safe, or allocates a suffixed name such as `blog-postgres-1`.

### `dokku ready:delete <app>`

This command requires typing `yes` before deletion. After confirmation it:

- shows a deletion summary including domains, ports, SSL state, and removable Postgres services
- deletes the app with `apps:destroy --force`
- unlinks and destroys linked Postgres services
- removes an orphan default Postgres service such as `<app>-postgres` when it exists and has no links
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

Real flow example:

```text
$ sudo dokku ready:create blog
Domain (leave empty to generate an sslip.io domain):
-----> Generated domain: blog.203.0.113.10.sslip.io
Container port [5000]:
Install Postgres? [Y]: y
Postgres service name [blog-postgres]:
Attempt SSL setup? [Y]: y
Let's Encrypt email (optional):
-----> Creating app: blog
-----> Configuring domain: blog.203.0.113.10.sslip.io
-----> Mapping http/https to container port 5000
-----> Creating postgres service: blog-postgres
-----> Linking postgres service to app
-----> Deploying temporary bootstrap app
-----> Attempting SSL enablement
=====> App ready
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

To update the plugin on the server after new commits are pushed:

```bash
sudo dokku plugin:update ready
```

`plugin:update` is the supported Dokku workflow for git-backed plugins and runs the plugin `update` trigger after fetching the latest commit.

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
