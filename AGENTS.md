# AGENTS.md — ESP32 Home Assistant Touchscreen Display

Tool-agnostic project brief for AI coding agents (Claude Code, Codex, Cursor,
Copilot, and anything else that reads `AGENTS.md`) working in this repository.

## Project overview

An [ESPHome](https://esphome.io/) configuration for a 2.4" ESP32 touchscreen
(ILI9341 display + XPT2046 touch controller) that sits on a desk and shows
live Home Assistant telemetry — clock, ComEd 5-minute electricity pricing,
indoor/outdoor temperature, HVAC action state, and door contact sensors —
without opening the Home Assistant app. Everything is pulled live via
ESPHome's native Home Assistant API integration: no MQTT, no polling hacks.

There is no application code to build — `esp-display.yml` **is** the
deliverable, compiled and flashed by the ESPHome toolchain.

## Repo structure

```
.
├── .github/
│   ├── dependabot.yml            # GitHub Actions updates only, monthly, grouped
│   └── workflows/
│       └── release-on-merge.yml  # creates a GitHub release + zip artifact on every merged PR to main
├── CONTRIBUTING.md
├── LICENSE                       # Unlicense (public domain)
├── README.md
├── esp-display.yml               # the ESPHome device configuration (the deliverable)
├── images/
│   └── IMG_4459.jpeg             # photo used in README
└── secrets.example.yaml          # template for secrets.yaml (gitignored — never commit the real file)
```

## Non-negotiables

- **`esp-display.yml` is the single source of truth.** Validate changes with
  `esphome config esp-display.yml` (or `esphome run esp-display.yml` against
  real hardware) before opening a PR.
- **Never commit `secrets.yaml`.** It's gitignored on purpose. Only edit
  `secrets.example.yaml`, and keep its placeholder values placeholders —
  real Wi-Fi/API/OTA credentials belong in a local `secrets.yaml` or the
  Home Assistant ESPHome dashboard's secrets store, never in git history.
- **Entity IDs in `esp-display.yml` are specific to the maintainer's Home
  Assistant instance** (`sensor.comed_5_minute_price`,
  `climate.great_room`, `binary_sensor.front_entrance_contact`, etc.). If you
  add, rename, or remove an entity the config depends on, update the
  README's "Prerequisites" entity list in the same change — the two are
  documented as a pair.
- **Font files aren't checked in.** The three Roboto `.ttf` files
  (`Roboto-Bold`, `Roboto-Medium`, `Roboto-Regular`) are expected to sit
  alongside `esp-display.yml` at upload time, downloaded by the user from
  Google Fonts. Don't add binary font files to the repo.
- **Every PR merged to `main` triggers `release-on-merge.yml`**, which
  creates a GitHub release named after the PR title. Keep PR titles
  descriptive and Conventional-Commit-formatted (see below) — they end up
  user-facing on the Releases page, one release per merge.

## Commit conventions

**Every commit must follow [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/).**

```
<type>(<optional scope>): <imperative description>

<optional body explaining why>

<optional footers>
```

- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`,
  `ci`, `chore`, `revert`.
- Scopes used here: `display`, `sensors`, `ha` (Home Assistant integration),
  `wifi`, `ci`, `docs`.
- Imperative mood, no trailing period, subject ≤ 72 chars.
- Breaking = a required entity ID changed or removed, or a config option a
  user's existing `secrets.yaml`/entities wouldn't satisfy anymore. Use
  `type(scope)!:` and/or a `BREAKING CHANGE:` footer.
- One logical change per commit. **PR titles use the same format** — every
  merge becomes a release name via `release-on-merge.yml`, so a sloppy title
  ships straight to the Releases page.

```
feat(display): add HVAC action icon to the layout
fix(sensors): correct outdoor feels-like entity id
docs: document required font downloads
ci(deps): bump actions/checkout in release-on-merge
```

## CI / automation

- **Dependabot** (`.github/dependabot.yml`) tracks the `github-actions`
  ecosystem only — the ESPHome YAML isn't a package manifest it can version.
  Monthly, grouped, commit prefix `ci`.
- **`release-on-merge.yml`** fires on every PR merged to `main`: it archives
  the repo at the merge commit and publishes a GitHub release tagged
  `pr-<number>-<run>`, named after the PR title, with auto-generated release
  notes. There's no opt-out per PR — every merge is a release.

## License

[Unlicense](LICENSE) — public domain. By contributing you agree your
contributions are released under the same terms.
