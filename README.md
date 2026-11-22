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
    version: 0.3.2   # optional: the latest release when omitted

- run: devscenario run e2e --platform android --pretty --artifact-root reports
```

## The `devscenario` CLI

```
devscenario run <input> [options]
```

### What it runs

| Input | Runs |
|---|---|
| Project folder (has `devscenario-project.json`) | One of the project's suites (`suites.json`), with its param profiles and translations: the only suite, or the one `--suite` names. `--scenario` runs one scenario or chain instead. |
| `flow.ds` | One flow. |
| Folder of flows | Every `*.ds` in it. |

CLI 1.9.0 and older also take a regression bundle `.zip` exported from Dev Scenario; later releases run the project folder instead.

### Options

| Option | Description |
|---|---|
| `--platform android\|ios\|both` | Where to run. Android when omitted. |
| `--device ID` | Device serial or simulator UDID. The first connected device (or booted simulator) when omitted. |
| `--ios-physical` | Run on a USB-connected iPhone/iPad (needs `--platform ios --device UDID`). |
| `--suite NAME` | Project folder: the suite to run. Needed when the project has more than one. |
| `--scenario NAME` | Project folder: run this scenario or chain instead of a suite. |
| `--profile NAME` | Project folder: param profile for scenarios and chains that don't pin one. The project's only profile when omitted. |
| `--android-driver-mode uiautomator-shell\|in-app-sdk` | How Android is driven. UIAutomator (any APK) by default; `in-app-sdk` is much faster when the app includes the Dev Scenario SDK. |
| `--ios-driver-mode wda\|in-app-sdk` | How iOS is driven. WebDriverAgent by default; `in-app-sdk` is much faster when the app includes the Dev Scenario SDK. |
| `--android-cross-app-fallback on\|off` | With `in-app-sdk`: fall back to UIAutomator outside your app (system dialogs) and when the SDK can't connect. On by default. `off` uses only the SDK, and a failed SDK connection fails the run. |
| `--app-id ID` | App package / bundle id for flows that don't launch one. |
| `--filter-tag TAG` | Only flows whose `tags:` contain `TAG`. |
| `--env KEY=VALUE` | A value flows can use; repeat for more. |
| `--timeout MS` | Time limit for each flow. |
| `--wait-timeout-ms MS` | Default wait for elements (`assertVisible`, `tapOn`, …) when a command sets none. |
| `--shard-split N` / `--shard-all N` | Spread flows over N devices, or run every flow on all N. |
| `--with-mocks` | Project folder: start the mock server (seeded from `mocks/`) even when no scenario or chain selects a mock group. Mocks a scenario or chain selects start by themselves, and the run gets `DEVTOOL_MOCK_BASE_URL`. |
| `--no-mocks` | Never start the mock server. |
| `--artifact-root DIR` | Where reports and screenshots go. |
| `--run-id ID` | Name of the run in reports. |
| `--pretty` | Readable progress lines instead of one JSON event per line. |
| `--timing` / `--timing-summary` | Print the time per flow and every step sorted by cost at the end. |

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

# Same project with another param profile
devscenario run e2e --profile staging

# iOS simulator
devscenario run e2e --platform ios --artifact-root reports

# Only smoke-tagged flows from a folder, split over two emulators
devscenario run flows --filter-tag smoke --shard-split 2
```

The workflow still needs a device: start an Android emulator (e.g. `reactivecircus/android-emulator-runner`) or boot an iOS simulator (`xcrun simctl boot`) and install your app before `devscenario run`.

## Action inputs

| Input | Default | Description |
|---|---|---|
| `version` | latest | CLI version, e.g. `0.3.2` (`cli-v0.3.2` works too). Pin it, and update on purpose. |
| `token` | `github.token` | Used to look up the latest release without hitting API rate limits. |

## Action outputs

| Output | Description |
|---|---|
| `version` | The installed version. |
| `path` | Folder holding `devscenario` (already on `PATH`). |

Needs Java 17 or later on the runner. Releases: [devscenario/devtool-releases](https://github.com/devscenario/devtool-releases/releases). Docs: [Run tests in CI](https://devscenario.com/docs/ci/).
