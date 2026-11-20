# Media Vision API

**Independent native metadata API and IMDb database manager for Media Vision.**

Media Vision API is a **separate product** from Media Vision Core. It runs in its own Debian 13 LXC
with its own configuration, named API keys, IMDb datasets, build/download schedules, updater,
backups and recovery. Its **metadata API** and **administration interface** listen on different
ports.

This repository distributes verified, versioned **Linux x64 native binaries and Proxmox
installers**. It is not a mirror of the private source repository.

[Latest MV API Stable release](https://github.com/zemerdon/media-vision-api-native/releases/latest)
· [Media Vision Core](https://github.com/zemerdon/media-vision-native) · [API release
history](https://github.com/zemerdon/media-vision-api-native/releases)

## What Media Vision API does

| Capability | Details |
| --- | --- |
| Metadata service | Independent, key-protected API on port **9292** |
| Administration | Browser WebUI on port **9393** for settings, keys, database operations, status and updates |
| IMDb datasets | Configurable source URLs, download/build and independent schedule controls |
| Client keys | Generate and manage named keys; secrets/configuration are owned by the API |
| Data safety | Independent configuration/keys and optional IMDb SQLite backup/restore |
| Software updates | Dedicated verified Stable release channel, native updater and rollback |
| Optional artwork service | Separately pinned/versioned **MVArtwork** component, managed by the API updater |

> **Important security limitation:** The standalone administration WebUI on port **9393 currently
> has no password login**. Anyone who can reach it can administer API keys and database operations.
> **Keep 9393 on a trusted management network and never expose it directly to the internet.** The
> separate metadata API on port **9292** still requires client keys.

## Quick installation — both one-liners

Run on the **Proxmox VE host as root**. These commands create **two independent new LXCs**, not
Docker containers, and should **not** be used to overwrite or upgrade an existing installation.

**1. Install Media Vision API** — this repository:

```bash
bash -c "$(curl -fsSL https://github.com/zemerdon/media-vision-api-native/releases/latest/download/bootstrap-native.sh)"
```

**2. Install Media Vision Core** — the separate Series/Movies application:

```bash
bash -c "$(curl -fsSL https://github.com/zemerdon/media-vision-native/releases/latest/download/bootstrap-native.sh)"
```

Installing **API first** makes it convenient to connect a fresh Core afterwards, but the products
can be installed, updated and restarted independently. Core can also be installed without an API
configured.

Both bootstraps select versioned **Stable** packages from the official GitHub release manifests and
verify archive sizes and SHA-256 hashes before creating a new LXC.

### Default LXC resources

| Setting | Media Vision API | Media Vision Core |
| --- | --- | --- |
| Guest | Native Debian 13 LXC | Native Debian 13 LXC |
| CPU | 4 vCPU | 4 vCPU |
| RAM | 4,096 MiB | 4,096 MiB |
| Root disk | 20 GiB minimum | 16 GiB default |
| Network | `vmbr0` / DHCP | `vmbr0` / DHCP |
| Administration / WebUI | `http://<api-ip>:9393` | `http://<core-ip>:9191` |
| Data API | `http://<api-ip>:9292` (client keys required) | Managed separately |

Prerequisites: a functioning Proxmox VE host, active container and template storage, network
connectivity to GitHub, and `whiptail` on Proxmox for the interactive menus. The installers use
native Proxmox commands (`pct`, `pvesh` and `pvesm`); **do not run inside an LXC**.

The installer offers **Default Install**, **Advanced Install**, **User Defaults** and **Settings**.
Advanced mode supports storage/network selection, static IP, hostname, sizing, timezone and optional
root SSH. Password-based root SSH is **off by default**.

**Plan storage before importing IMDb data.** The 20 GiB minimum is only an installation baseline,
not a guarantee that a large IMDb database plus download/build staging and backup snapshots will
fit. Choose a larger disk or other supported storage when your dataset size requires it.

## First run and connecting Core

1. Open the API administration WebUI on your **private management network**: `http://<api-ip>:9393`.
2. Configure the IMDb dataset source URLs, choose the datasets you need, and start a manual
   download/build when ready. **The installer does not ship an IMDb database, automatically build
   one or enable an automatic refresh schedule.**
3. Create a named client API key as required. The data API on `http://<api-ip>:9292` is **not** the
   same service as the passwordless admin port.
4. Install Media Vision Core using its own one-liner, then open `http://<core-ip>:9191`.
5. In Core, open **MV API** and connect to the independent **administrator URL**, normally
   `http://<api-ip>:9393`. Verify connection and independently configure any metadata client
   credentials required by your integration.

Core itself contains two operationally independent managers, **Series** and **Movies**. This API
provides supporting metadata services; it does not borrow either manager's indexers, download
clients, queue, monitoring or root folders.

## Using the WebUI

The standalone API WebUI provides its own administration pages for:

- **Overview / status:** health, connectivity, installed version and current database availability.
- **API keys:** generate, view/manage named client access and revoke keys.
- **IMDb Database:** configure sources, inspect database/dataset sizes and integrity, manually
  download/build/rebuild and manage independent refresh schedules.
- **Updates:** inspect published Stable releases and apply verified API software updates through the
  API's own updater.
- **Backup / restore:** export API-owned configuration and hashed keys, and separately select the
  IMDb SQLite database for snapshot/export/restore.

A newly installed API does not silently copy indexes, configuration or key material from another
LXC. Published binaries contain **no private data**.

## Updating and backup safety

**API updates are separate from Core updates.** Update each product using its own verified Stable
release/channel; do not assume a Core upgrade also upgrades API software. The API's native updater
and rollback flow preserve its own state. MVArtwork software has its own pinned version managed
separately through API controls.

For a fresh installation or migration:

1. Export and verify the API **configuration and API-key backup**.
2. If you need the exact indexed data, also export and verify the **full IMDb SQLite backup**. This
   needs enough **free temporary space** for a consistent snapshot and extraction; a compressed
   ZIP's size alone is not sufficient for capacity planning.
3. Preserve independent **MVArtwork** data/manual locks if you use that optional component; it is
   not included in a Core backup.
4. Keep the old API LXC intact while you install a new API LXC, restore state and verify both
   `:9292` and `:9393`.
5. Independently restore Media Vision Core in its own LXC, then confirm the connection points to the
   **new** API host.

Core's backup ZIP does **not** include your API database, keys or MVArtwork state. The API's backup
does not include Core's Series/Movies operational settings. Neither product installer copies your
real media library or changes existing Sonarr, Radarr or Prowlarr installations.

## Release files and integrity

The [latest Stable API release](https://github.com/zemerdon/media-vision-api-native/releases/latest)
includes:

- `bootstrap-native.sh` — permanent, new-install Proxmox entry point.
- `media-vision-api-<version>-linux-x64.tar.gz` — self-contained native API software.
- `media-vision-api-native-proxmox-<version>.tar.gz` — versioned Proxmox installer.
- `release-manifest.json` — identity, channel, versioned download URLs, sizes, source SHA and
  SHA-256 checksums.

Official release bundles and checksums are preserved per version. Review the [API release
notes](https://github.com/zemerdon/media-vision-api-native/releases) for changes; install Core from
its **separate** [release
repository](https://github.com/zemerdon/media-vision-native/releases/latest).

## Troubleshooting

| Symptom | Check |
| --- | --- |
| Installer fails on root or Proxmox tools | Run on the **Proxmox host**, as root, not inside Debian/LXC |
| Visual installer will not open | Install `whiptail` and use an interactive terminal |
| API administration page unreachable | Check API LXC networking, service and port **9393** on trusted LAN |
| Metadata calls are unauthorized | Use the correct named client key against **9292**; admin port is different |
| Empty index after installation | Expected until you configure/import/build IMDb datasets |
| Database build or backup fails for space | Increase available API storage; check free space for *uncompressed* SQLite staging |
| Core shows API disconnected | Enter the API **administrator URL** (`:9393`) under Core **MV API** settings |
| A Core restore did not restore API data | Restore **API state separately**; the two backup systems do not overlap |

**Links:** [API Stable
downloads](https://github.com/zemerdon/media-vision-api-native/releases/latest) · [Core installer
and guide](https://github.com/zemerdon/media-vision-native#readme) · [API release
history](https://github.com/zemerdon/media-vision-api-native/releases)

---

*Media Vision API is an independent metadata service and native release product. This public
repository carries verified software packages and installers, not the private source or a copy of
user datasets.*
