# Hassette

[Hassette](https://github.com/NodeJSmith/hassette) runs Home Assistant automations written as async, typed
Python apps. This add-on runs it next to Home Assistant, connects it with no token to paste, and puts its
web UI in the sidebar.

The full add-on guide lives on the Hassette docs site:
<https://hassette.readthedocs.io/en/stable/pages/getting-started/addon/>

## Quick start

1. Install **Hassette**.
2. Turn on **Watchdog** on the Info tab. It restarts hassette if it crashes or stops answering.
3. Start the add-on, then open **Hassette** from the sidebar.

On first start, the add-on fills an empty config folder with a commented `hassette.toml` and an example
app. From Samba, File editor or Studio Code Server, the folder is under `app_configs/`, in a folder ending
in `_hassette`. Put your apps in its `apps/` folder.

If you use the [Hassette integration](https://github.com/NodeJSmith/hass-hassette), Home Assistant shows a
**Discovered** Hassette card under **Settings → Devices & services**. Choose **Configure** and confirm.

## More

- [Setup](https://hassette.readthedocs.io/en/stable/pages/getting-started/addon/): install, first app,
  pairing the integration
- [Configuration](https://hassette.readthedocs.io/en/stable/pages/getting-started/addon/configuration/):
  options, files, settings the add-on owns, Python packages, the direct port, the web API token
- [Watchdog, Backups & Troubleshooting](https://hassette.readthedocs.io/en/stable/pages/getting-started/addon/operating/)

Report problems at <https://github.com/NodeJSmith/hassette/issues>. Include the add-on's log.
