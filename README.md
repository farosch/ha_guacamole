# Guacamole Home Assistant Add-on

This add-on runs Apache Guacamole (web UI), guacd, and an internal MariaDB database inside a single add-on container. Data is stored under `/data` and survives updates.

## Features
- Direct network access on host port 8080 (same LAN as Home Assistant)
- Persistent data: MySQL datadir and Guacamole config under `/data`
- First-run DB initialization and schema setup

## Options
- `db_user` (string): Database user (default: `guacamole`)
- `db_password` (password): Database password (default: `changeme`)
- `admin_password` (password): Initial Guacamole admin password (default: `guacadmin`). Applied on first run.

## Installation
1. In Home Assistant, add this repository as an Add-on repository.
2. Open the "Guacamole" add-on and click Install.
3. Optionally edit configuration to set `db_password`.
4. Start the add-on.

## Access
- URL: http://YOUR_HA_HOST:YOUR_PORT/guacamole/

## Network Port
- The add-on exposes container port 8080. The external host port now defaults to 8321 and can be changed in the add-on's Network tab.
- Example: leave `8080/tcp` → `8321` (default), then open http://YOUR_HA_HOST:8321/guacamole/

## Default Credentials (first run)
- Username: `guacadmin`
- Password: `guacadmin`
Change this immediately after first login under Guacamole settings.

## Data Locations
- MariaDB: `/data/mysql`
- Guacamole Home: `/data/guacamole`
  - `guacamole.properties`
  - `extensions/`
  - `lib/`

## Notes
- The add-on binds services locally; external ports are not exposed. Use Ingress to access Guacamole.
- On first start, the add-on initializes the database, creates the schema, and installs the JDBC extension.
