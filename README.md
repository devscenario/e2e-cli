# e2e-cli

Installs the [Dev Scenario](https://devscenario.com) CLI, `devscenario`, and puts it on `PATH`, so a workflow can run your Dev Scenario tests on an Android emulator or iOS simulator.

## Usage

```yaml
- uses: actions/setup-java@v4
  with:
    distribution: corretto
    java-version: '17'

- uses: devscenario/e2e-cli@v1
  with:
    version: 0.2.0   # optional: the latest release when omitted

- run: devscenario run e2e --platform android --pretty --artifact-root reports
```

## The `devscenario` CLI

```
devscenario run <input> [options]
```

### What it runs

| Input | Runs |
|---|---|
| Project folder (has `devscenario-project.json`) | The project's regression list (`regression.json`), with its test users, param profiles and translations. Build the list in Dev Scenario › Home › Regression builder and push it. |
| Regression bundle `.zip` | A bundle exported from Dev Scenario › Regression builder › Export bundle. |
| `flow.yml` | One flow. |
| Folder of flows | Every `*.yml` in it. |

### Options

| Option | Description |
|---|---|
| `--platform android\|ios\|both` | Where to run. Android when omitted. |
| `--device ID` | Device serial or simulator UDID. The first connected device (or booted simulator) when omitted. |
| `--ios-physical` | Run on a USB-connected iPhone/iPad (needs `--platform ios --device UDID`). |
| `--user NAME` | Project folder: test user for scenarios that don't pin one. The project's only user when omitted. |
| `--profile NAME` | Project folder: param profile for scenarios and chains that don't pin one. The project's only profile when omitted. |
| `--android-driver-mode uiautomator-shell\|in-app-sdk` | How Android is driven. UIAutomator (any APK) by default; `in-app-sdk` is much faster when the app includes the Dev Scenario SDK. |
| `--ios-driver-mode wda\|in-app-sdk` | How iOS is driven. WebDriverAgent by default. |
| `--android-cross-app-fallback on\|off` | With `in-app-sdk`: fall back to UIAutomator outside your app (system dialogs). On by default. |
| `--app-id ID` | App package / bundle id for flows that don't launch one. |
| `--filter-tag TAG` | Only flows whose `tags:` contain `TAG`. |
| `--env KEY=VALUE` | A value flows can use; repeat for more. |
| `--timeout MS` | Time limit for each flow. |
| `--wait-timeout-ms MS` | Default wait for elements (`assertVisible`, `tapOn`, …) when a command sets none. |
| `--shard-split N` / `--shard-all N` | Spread flows over N devices, or run every flow on all N. |
| `--with-mocks` | Bundle only: serve the bundle's API mocks and pass `DEVTOOL_MOCK_BASE_URL` to the run. |
| `--artifact-root DIR` | Where reports and screenshots go. |
| `--run-id ID` | Name of the run in reports. |
| `--pretty` | Readable progress lines instead of one JSON event per line. |
| `--timing-summary` | Print the slowest commands at the end. |

`devscenario --help` prints the same list.

### Reports and exit codes

Under `--artifact-root`: `junit.xml` (one test case per flow; works with GitHub, GitLab, Jenkins and test-management tools), `run.json` and `events.jsonl` (the whole run, step by step), and screenshots of failures.

| Exit code | Meaning |
|---|---|
| `0` | Every flow passed. |
| `1` | A flow failed. |
| `2` | Wrong usage or input (unknown option, missing file, no regression list). |

### Examples

```bash
# The project in e2e/, on the first Android device, through the in-app SDK
devscenario run e2e --android-driver-mode in-app-sdk --pretty --artifact-root reports

# Same project as another test user and profile
devscenario run e2e --user bob@example.com --profile staging

# iOS simulator
devscenario run e2e --platform ios --artifact-root reports

# Only smoke-tagged flows from a folder, split over two emulators
devscenario run flows --filter-tag smoke --shard-split 2
```

The workflow still needs a device: start an Android emulator (e.g. `reactivecircus/android-emulator-runner`) or boot an iOS simulator (`xcrun simctl boot`) and install your app before `devscenario run`.

## Action inputs

| Input | Default | Description |
|---|---|---|
| `version` | latest | CLI version, e.g. `0.2.0` (`cli-v0.2.0` works too). Pin it, and update on purpose. |
| `token` | `github.token` | Used to look up the latest release without hitting API rate limits. |

## Action outputs

| Output | Description |
|---|---|
| `version` | The installed version. |
| `path` | Folder holding `devscenario` (already on `PATH`). |

Needs Java 17 or later on the runner. Releases: [devscenario/devtool-releases](https://github.com/devscenario/devtool-releases/releases). Docs: [Run tests in CI](https://devscenario.com/docs/ci/).
