# dokku ready

Interactive app bootstrap and cleanup plugin for dokku.

## Requirements

- dokku 0.20.x+
- root privileges or passwordless `sudo` for automatic plugin installation
- outbound network access for automatic `sslip.io` generation unless `DOKKU_READY_PUBLIC_IP` is set

## Installation

```shell
# install from git
sudo dokku plugin:install https://github.com/hasan5151/dokku-ready.git --name ready

# install from a local checkout
sudo dokku plugin:install file:///absolute/path/to/dokkuready --name ready
```

## Commands

```text
ready:create <app> [--domain <domain>] [--port <port>] [--postgres|--no-postgres] [--postgres-service <service>] [--ssl|--no-ssl] [--email <email>] # interactively create or reconcile an app with domain, ports, optional postgres, and optional ssl
ready:delete <app> [-f|--force]                                                                                                                 # display a destructive summary, then remove the app and related postgres resources
ready:help                                                                                                                                      # display help for the ready plugin
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to `ready:help`. Please consult the `ready:help` command for any undocumented commands.

### Basic Usage

### create or reconcile an app

```shell
# usage
dokku ready:create <app> [--domain <domain>] [--port <port>] [--postgres|--no-postgres] [--postgres-service <service>] [--ssl|--no-ssl] [--email <email>]
```

Dokku Ready performs the following steps in order:

- creates the app if it does not already exist
- prompts for a domain and applies it
- generates `<app>.<public-ip>.sslip.io` automatically if the domain is left empty
- prompts for the container port and maps `http:80:<port>` and `https:443:<port>`
- asks whether postgres should be installed
- installs the official `postgres` plugin automatically if needed
- creates or safely reuses a postgres service if enabled
- links the postgres service to the app when needed
- asks whether ssl should be enabled
- installs the official `dokku-letsencrypt` plugin automatically if needed
- deploys a temporary bootstrap app without requiring `git push`
- attempts ssl enablement

Dokku Ready is idempotent:

- if the app already exists, it continues and reconciles the requested state
- if the app already has a domain, port mapping, linked postgres service, or active ssl, those values are reused as defaults
- if the default postgres service name already exists, it is reused when safe, otherwise a suffixed name such as `blog-postgres-1` is chosen

Create an app interactively:

```shell
dokku ready:create blog
```

Run non-interactively:

```shell
dokku ready:create blog --domain blog.example.com --port 5000 --postgres --postgres-service blog-postgres --ssl --email ops@example.com
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

Notes:

- automatic plugin installation uses `dokku plugin:install`, so it must run with elevated privileges
- automatic `sslip.io` generation requires a detectable public IPv4 address or a preset `DOKKU_READY_PUBLIC_IP`
- let's encrypt activation may fail until dns points to the dokku server and the bootstrap deploy is reachable
- the temporary bootstrap deploy exists only to make the app answer HTTP requests so certificate issuance can succeed

### delete an app and related resources

```shell
# usage
dokku ready:delete <app> [-f|--force]
```

Before deletion, Dokku Ready displays a destructive summary that includes:

- the target app
- configured domains
- proxy port mappings
- ssl status
- removable postgres services

After confirmation it:

- disables letsencrypt if managed by dokku
- unlinks and destroys linked postgres services
- removes an orphan default postgres service such as `<app>-postgres` when it exists and has no remaining links
- destroys the app with `apps:destroy --force`

Delete an app interactively:

```shell
dokku ready:delete blog
```

Force deletion without confirmation:

```shell
dokku ready:delete blog --force
```

### update the plugin

```shell
# usage
sudo dokku plugin:update ready
```

`plugin:update` is the supported dokku workflow for git-backed plugins. It fetches the latest commit and runs the plugin `update` trigger.

Verify the installed version:

```shell
sudo git -C /var/lib/dokku/plugins/available/ready log --oneline -1
```

### environment overrides

The following optional environment variables can be used to customize behavior:

```shell
DOKKU_READY_PUBLIC_IP=203.0.113.10
DOKKU_READY_DEFAULT_PORT=5000
DOKKU_READY_BOOTSTRAP_BASE_IMAGE=alpine:3.20
DOKKU_READY_BOOTSTRAP_IMAGE_PREFIX=dokku-ready-bootstrap
DOKKU_READY_POSTGRES_PLUGIN_URL=https://github.com/dokku/dokku-postgres.git
DOKKU_READY_LETSENCRYPT_PLUGIN_URL=https://github.com/dokku/dokku-letsencrypt.git
```
