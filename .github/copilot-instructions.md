## 🎯 Purpose
This repository contains Home Assistant add-ons maintained by the project owner. Each add-on is self-contained in its own folder (e.g. `hamc-server-bedrock`, `hamc-server-java`, `msrewards`) and follows the alexbelgium Home Assistant add-on template.

## 🧩 Big-picture architecture
- One add-on per top-level directory. Each add-on typically contains:
  - `config.yaml` — add-on metadata, `options` (user-facing config) and `schema` (validation). The `slug` and `version` fields are authoritative for releases.
  - `build.yaml` — the `build_from` base image(s) used when building for different architectures.
  - `Dockerfile` — the add-on image build steps (adds `/rootfs`, installs packages, sets entrypoint).
  - `rootfs/` — the runtime file-system contents (e.g. `/etc/cont-init.d/*` scripts, service files, templates).

## 🔧 Key workflows (how to build / run / publish)
- CI publish uses `.github/workflows/publish.yml` which authenticates to GitHub Container Registry (ghcr.io) and calls `home-assistant/builder` to publish multi-arch images to GHCR. Ensure the workflow includes `permissions: packages: write` and either uses `GITHUB_TOKEN` (recommended) or a PAT with `packages:write` permission.
- Local/manual build: the Dockerfile accepts typical build-args (`BUILD_FROM`, `BUILD_VERSION`, etc.) and is consistent with `build.yaml`. For quick tests you can use `docker build` with `--build-arg BUILD_FROM=...` or run the `home-assistant/builder` image locally; ensure your local `docker login ghcr.io` is authenticated for pushing.
- To test behavior inside Home Assistant, the repository includes a VS Code task `Start Home Assistant` (see `.vscode/tasks.json`) to run the Supervisor locally when available.

> Note: Add-ons specify their image name in `config.yaml` (e.g. `image: ghcr.io/williamcorsel/{arch}-hamc-bedrock`) — update these to point at `ghcr.io/<owner>/...` when migrating from Docker Hub.

## 🧭 Project-specific conventions & patterns
- All add-ons use the alexbelgium template pattern: helper scripts in `.templates/` (e.g. `automatic_packages.sh`, `banner.sh`) and `bashio` functions inside `rootfs/etc/cont-init.d/*.sh` scripts.
- Configuration -> `config.yaml` options are converted to env vars and validated by the supervisor. Use `schema:` in `config.yaml` to see valid values and types.
- Persistent data is expected under `/config` (see `map:` in `config.yaml`) and migration scripts (e.g. `00-migrate_data.sh`) may move old data into `/addon_configs`.
- Changelog / versioning: update `config.yaml` → `version` and add a line in `CHANGELOG.md` on releases.

## 🔗 External integrations & dependencies
- Uses external templates pulled from `https://raw.githubusercontent.com/alexbelgium/hassio-addons/...` and also the `home-assistant/builder` GH action.
- Example external base images: `itzg/minecraft-*` for the server add-ons (see `build.yaml` in `hamc-server-*`).
- The `msrewards` add-on uses Python + Selenium and installs `chromium` + `chromium-chromedriver` in the image (see `Dockerfile` and `rootfs/requirements.txt`). Local testing of `msrewards` should account for browser driver dependencies.

## 🐞 Debugging tips (quick, repo-specific)
- Check logs emitted by `bashio::log.*` in container `cont-init` scripts. Look at `rootfs/etc/cont-init.d/run.sh` for runtime commands (e.g. `python /ms_rewards_farmer.py ...`).
- To run `msrewards` locally for development: install the `rootfs/requirements.txt` in a virtualenv and run `python rootfs/ms_rewards_farmer.py --help` or mimic the entrypoint args shown in `rootfs/etc/cont-init.d/run.sh`.

## ✅ Do / Don't for PRs & agent actions
- DO: Keep add-on `version` and `CHANGELOG.md` in sync. Run a full build via the GH Action (`.github/workflows/publish.yml`) for release validation.
- DO: Respect `config.yaml` schema when adding options; reference the upstream image docs for platform-specific env vars (e.g. itzg images).
- DON'T: Assume a global test suite exists — there are no unit tests here; prefer manual/integration testing in a Supervisor or by building the container.

---
If you'd like, I can iterate this file to expand examples (e.g. exact `docker build` flags, a templated `home-assistant/builder` CLI example, or a short checklist for releasing an add-on). Which area should I expand? ✅
