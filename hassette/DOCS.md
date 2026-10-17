# Hassette

[Hassette](https://github.com/NodeJSmith/hassette) runs Home Assistant automations written as async, typed
Python apps. This add-on runs it next to Home Assistant, connects it with no token to paste, and puts its
web UI in the sidebar.

Full framework docs: <https://hassette.readthedocs.io/>

## Installation

1. Add this repository to Home Assistant's app store:

   [![Add repository to Home Assistant](https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg)](https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2FNodeJSmith%2Fhassette-addon)

   Or open **Settings → Apps → App store**, choose **Repositories** from the ⋮ menu, and add
   `https://github.com/NodeJSmith/hassette-addon`. (Older Home Assistant versions call apps "add-ons".)
2. Install **Hassette**.
3. Turn on **Watchdog** on the add-on's Info tab (see [Watchdog](#watchdog)).
4. Start the add-on, then open **Hassette** from the sidebar.

On first start, the add-on fills an empty config folder with a commented `hassette.toml` and an `apps/` folder
holding one example app that logs a line. If the folder already holds anything, nothing is written.

## Where your files live

The add-on's config folder is mounted at `/config` inside the container. From Samba, File Editor or Studio Code
Server, it's under `app_configs/` in a folder ending in `_hassette`. It holds:

- `hassette.toml`: hassette's settings
- `apps/`: your apps
- `.web_api_token`: the web API token (see [Web API token](#web-api-token))

A `uv.lock` project placed in this folder is installed on every start, the same as in the Docker image.

The telemetry database and the package cache live in the add-on's private `/data` folder, which you don't
edit.

## The web UI

The sidebar entry serves the UI through Home Assistant's ingress, behind Home Assistant's own login. There
is no separate hassette login there.

The sidebar entry is shown to admins only. That hides the entry; it doesn't restrict access. Any logged-in
Home Assistant user who has the ingress URL can use the full UI, including starting, stopping and reloading
apps.

### Direct port (optional)

Port 8126 serves the same UI and the web API directly, for the `hassette` CLI or a browser outside Home
Assistant. It is off until you enter a host port in the add-on's **Network** section. The direct port always
requires the web API token.

## Configuration

| Option | What it does |
|---|---|
| `log_level` | Sets hassette's log level. When set, it overrides `logging.log_level` in `hassette.toml`. Leave it unset to use `hassette.toml`, or hassette's default (`INFO`). |
| `python_packages` | Extra Python packages installed on every start, one requirement per entry, such as `requests` or `httpx>=0.27`. |

`python_packages` accepts plain requirement specifiers only. URL and VCS specifiers (`git+https://…`) and
environment markers with quoted values (`; python_version > '3.11'`) aren't supported. For those, or for a real project with pinned dependencies, put a `uv.lock` project in the config
folder instead.

Everything else goes in `hassette.toml`, as described in the
[configuration docs](https://hassette.readthedocs.io/).

### Settings the add-on owns

The add-on sets these on every start. Environment variables outrank `hassette.toml`, so editing them there
has no effect:

- `base_url` and `token`: hassette reaches Home Assistant through Supervisor's proxy
  (`http://supervisor/core`) with Supervisor's token.
- `web_api.host`, `web_api.port`, `web_api.run`, `web_api.run_ui` and `web_api.auth_enabled`: the web server
  always listens on `0.0.0.0:8126` with the UI and auth on, which the sidebar, the health check and discovery
  rely on.
- `web_api.trusted_proxies`: the ingress gateway (`172.30.32.2`), so sidebar requests skip the token check.
- `web_api.token_file`: `/config/.web_api_token`.
- `database.path`: `/data/hassette.db`, so backups capture it (see [Backups](#backups)).

**Don't set `web_api.auth_token` in the add-on.** It isn't overridden, but a configured value wins over the
token file, so the integration's pairing would carry a token that no longer works. hassette logs a warning
when this happens.

## Web API token

The direct port, the `hassette` CLI and the Home Assistant integration authenticate with the web API token.
hassette generates it on first start and writes it to `.web_api_token` in the config folder. Open that file
to copy the token. It's a hidden file, so turn on "show hidden files" in Samba or your editor if you don't
see it.

To rotate it, delete `.web_api_token` and restart the add-on. hassette writes a new token. An integration
paired through the Discovered card picks it up on its own (see
[Pairing the integration](#pairing-the-integration)). Anything else using the old token, such as the CLI or
an integration you added by hand, needs the new one.

The token is included in backups, like the rest of the config folder.

## Pairing the integration

The [Hassette integration](https://github.com/NodeJSmith/hass-hassette), installed through HACS, turns each
app into a Home Assistant device you can start, stop and reload.

Once the integration is installed and the add-on is running, Home Assistant shows a **Discovered** Hassette
card under **Settings → Devices & services**. Click **Configure** and confirm. The add-on supplies the
address and token, so there's nothing to type.

Pairing sends the web API token from the add-on to Home Assistant through Supervisor's discovery, and the
integration keeps it in its config entry. Only confirm a Hassette card you expect.

The card appears only after hassette is serving. If the token rotates or the add-on's address changes, an
integration paired this way reads the new details from the add-on the next time a request fails, and updates
itself. An integration you added by hand doesn't; update its token with **Reconfigure**.

## Watchdog

Turn on **Watchdog** from install. It restarts hassette if it crashes, exits on a fatal error, or fails its
health check three times in a row. Without it, a stopped add-on stays stopped until you start it again.

Restarts back off and stop after 10 attempts in 30 minutes, so a persistent failure leaves the add-on stopped
with the reason in the log.

Watchdog never restarts hassette because Home Assistant is down. hassette waits and reconnects on its own.

## Startup

Dependencies install on every start, not only the first: the `uv.lock` project if there is one, then
`python_packages`. The add-on keeps a package cache, so later starts are quick.

If the package index can't be reached, for example when Home Assistant boots before the network is up, the
add-on installs from the cache instead. If that fails too, hassette starts anyway, and the log shows a banner
naming the install that didn't run. Apps that need the missing package fail to import, and the log shows why.

A config error or a dependency conflict stops the add-on, with a banner in the log saying what to fix. Fix
it, then restart the add-on from its page.

## Backups

Backups run while hassette keeps running. Before each backup, the add-on writes a consistent snapshot of
the telemetry database, and the backup stores that snapshot in place of the live database files. A restore
brings history back up to the moment of the snapshot. The package cache is left out of backups; it refills
on the next start.

## Support

Report problems at <https://github.com/NodeJSmith/hassette/issues>. Include the add-on's log.

## License

MIT
