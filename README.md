# setup-devscenario

Installs the [Dev Scenario](https://devscenario.com) CLI, `devscenario`, and puts it on `PATH`, so a workflow can run your Dev Scenario tests on an Android emulator or iOS simulator.

## Usage

```yaml
- uses: actions/setup-java@v4
  with:
    distribution: corretto
    java-version: '17'

- uses: devscenario/setup-devscenario@v1
  with:
    version: 0.2.0   # optional: the latest release when omitted

- run: devscenario run e2e --platform android --pretty --artifact-root reports
```

`devscenario run` takes your Dev Scenario project folder (it runs the project's regression list), a regression bundle `.zip`, a flow `.yml`, or a folder of flows.

### Android emulator

```yaml
jobs:
  regression:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: corretto
          java-version: '17'
      - uses: devscenario/setup-devscenario@v1

      - name: Build the app
        run: ./gradlew :app:assembleDebug

      - name: Enable KVM
        run: |
          echo 'KERNEL=="kvm", GROUP="kvm", MODE="0666"' | sudo tee /etc/udev/rules.d/99-kvm.rules
          sudo udevadm control --reload-rules
          sudo udevadm trigger --name-match=kvm

      - uses: reactivecircus/android-emulator-runner@v2
        with:
          api-level: 34
          arch: x86_64
          script: |
            adb install -r app/build/outputs/apk/debug/app-debug.apk
            devscenario run e2e --platform android --pretty --artifact-root reports

      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: regression
          path: reports/
```

## Inputs

| Input | Default | Description |
|---|---|---|
| `version` | latest | CLI version, e.g. `0.2.0` (`cli-v0.2.0` works too). Pin it, and update on purpose. |
| `token` | `github.token` | Used to look up the latest release without hitting API rate limits. |

## Outputs

| Output | Description |
|---|---|
| `version` | The installed version. |
| `path` | Folder holding `devscenario` (already on `PATH`). |

Needs Java 17 or later on the runner. Releases: [devscenario/devtool-releases](https://github.com/devscenario/devtool-releases/releases). Docs: [Run tests in CI](https://devscenario.com/docs/ci/).
