# Apache Guacamole Home Assistant Add-on

<img src="guacamole/logo.png" alt="Guacamole Add-on" width="220" />

This add-on runs Apache Guacamole (web UI), guacd, and an internal MariaDB database in a single container. Data is stored under `/data` and survives updates.

## Features

- Direct LAN access via mapped host port (default 8321)
- Persistent data: MariaDB datadir and Guacamole config under `/data`
- Automatic first-run initialization with generated credentials

## Installation

### 1-Click Setup

[![Open your Home Assistant instance and show the add add-on repository dialog with a specific repository URL pre-filled.](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Ffarosch%2Fha_guacamole)

### Manual Installation

1. In Home Assistant, add a repository.
2. Use `https://github.com/farosch/ha_guacamole/` as URL.
3. Open the "Guacamole" add-on and click Install.
4. Start the add-on. No configuration is required.

### DEV Builds

#### 1-Click Setup

[![Open your Home Assistant instance and show the add add-on repository dialog with a specific repository URL pre-filled.](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Ffarosch%2Fha_guacamole%2Ftree%2Fdevelop)

#### Manual Installation

1. In Home Assistant, add a repository.
2. Use `https://github.com/farosch/ha_guacamole/tree/develop` as URL.
3. Open the "Guacamole" add-on and click Install.
4. Start the add-on. No configuration is required.

You can also use this

## Access

- URL: http://YOUR_HA_HOST:YOUR_PORT/guacamole/
- Default port mapping: container `8080` → host `8321` (editable in the Add-on Network tab)

## First-Run Credentials

- Guacamole Admin: `guacadmin / guacadmin`
- Database:
  - Name: `guacamole_db`
  - User: `guacamole`
  - Password: randomly generated

On the first install, the add-on prints all credentials once to the Supervisor log and saves the database credentials to `/data/guacamole/credentials.env`.

> IMPORTANT: The database password is shown only once in the log on initial install. It is persisted in `/data/guacamole/credentials.env` and referenced by `/data/guacamole/guacamole.properties`.

## Change Admin Password

After logging in as `guacadmin`, change the password in the Guacamole UI (Settings → Users).

## Data Locations

- MariaDB: `/data/mysql`
- Guacamole Home: `/data/guacamole`
  - `guacamole.properties`
  - `credentials.env`
  - `extensions/`
  - `lib/`
