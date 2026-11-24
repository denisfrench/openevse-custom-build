# Custom OpenEVSE controller firmware build

A GitHub Actions workflow that builds the OpenEVSE controller firmware
([OpenEVSE/open_evse](https://github.com/OpenEVSE/open_evse)) with a few build
flags changed, **without forking it**. Nothing from upstream is stored here: each
run checks out the ref you choose, edits `platformio.ini` in a temporary
checkout, builds, and publishes the hex file.

Default changes (each can be switched off per run):

| Change | Flag |
|---|---|
| Add front button and menus | `-D BTN_MENU` |
| Remove Mennekes lock support | `-D MENNEKES_LOCK` |
| Remove cable temperature monitoring (`dev` only) | `-D CABLE_TEMPERATURE_MONITORING` |
| Remove DC relay PWM hold | `-D RELAY_PWM` |

## Files

- `.github/workflows/build-openevse.yml`: the workflow
- `README.md`: this file

## One-time setup

1. Create a **public** GitHub repository (public repos get free Actions minutes).
2. Add the workflow at `.github/workflows/build-openevse.yml` (**Add file > Create new file**, typing the path creates the folders), and commit it.

## Running a build

1. Open the **Actions** tab, select **Build OpenEVSE controller firmware**, click **Run workflow**.
2. Fill in the inputs, then click **Run workflow**:

| Input | Meaning |
|---|---|
| `ref` | Upstream tag, branch or commit SHA. Use a full SHA for `dev`, which keeps changing. |
| `pio_env` | `m328p_LCD_WIFI` for 9.0.0 / master, `m328p_core_lcd` for `dev`. |
| toggles | Switch each flag change on or off. |

3. Open the finished run. The **Summary** shows flash and RAM use at the top, with a
   warning if flash is at or above 90% or RAM at or above 80% (edit `FLASH_WARN_PCT`
   and `RAM_WARN_PCT` in the workflow to change these). Flash or RAM over the chip's
   limit fails the run.
4. Download the artifact from the bottom of the run page. It contains the hex file,
   `build.log`, and `BUILD-INFO.txt` (upstream commit, changes applied, sizes, SHA-256).

## Flashing

The output is the same kind of file as an upstream release hex. Using the command
from upstream's `flash.bat` (USBasp programmer):

```
avrdude -c USBasp -p m328p -B0.5 -U flash:w:<your-file.hex>
```

## Updating

Run the workflow again with the new tag. If upstream changes its file layout, the
run stops with an error instead of building with unexpected flags.

## Notes

- This is an unofficial build, not tested by upstream, and a successful build does
  not prove it works on hardware.
- The version string is unchanged (for example `D9.0.0.legacy`). Use the SHA-256 in
  `BUILD-INFO.txt` to tell it apart from a stock build.
- PlatformIO 6.1.5 is pinned to match upstream's CI.

## Troubleshooting

| Error | Meaning |
|---|---|
| `env '...' not found at ref ...` | Wrong `pio_env` for that ref. |
| `-D ... not found` | Upstream moved or renamed the flag; review before building. |
| `... is not in the compile commands` / `still in the compile commands` | A flag change didn't reach the compiler; check `build.log`. |
| `could not read ... from the PlatformIO output` | PlatformIO's size lines changed format; check `build.log`. |
