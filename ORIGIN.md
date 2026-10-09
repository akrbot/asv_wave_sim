# asv_wave_sim in asv_ws - origin and local changes

This directory is a **workspace package derived from gz-waves**, not a pristine
third-party checkout. It is its own git repository (started fresh from the
imported files, not upstream's history) and a **git submodule** of the asv_ws
`src` repository, which records which commit of it the workspace uses.

| | |
|---|---|
| Upstream | [srmainwaring/asv_wave_sim](https://github.com/srmainwaring/asv_wave_sim) (Rhys Mainwaring) |
| Imported from | a clone of upstream at `~/Downloads/asv_wave_sim`, which keeps the full upstream history |
| This repository | first commit `901c654` = upstream `ca8629d` + the local changes below |
| Upstream commit | `ca8629d` - "Prevent duplicate textures being loaded in displacement map" |
| Imported | 2026-10-09 |
| Licence | GPL-3.0 (upstream's; see `LICENSE`, `LICENSE_THIRDPARTY`). It depends on CGAL and FFTW, both GPL |
| Built as | `gz-waves1`, a plain CMake package colcon builds with the rest of the workspace. `gz-waves-models/` carries `COLCON_IGNORE`: resources only, used in place by `asv_gz_waves` |
| Used by | `asv_gz_waves` (the `fft_ocean` model and `open_sea_fft` world) |

## Local changes from upstream

All four are marked in the source with an `asv_ws local change` / `asv_ws
local fix` comment. The same set, as a diff against `ca8629d`, is in
`asv_gz_waves/patches/gz-waves-asv_ws.patch` - use it to carry the changes
onto a newer upstream.

| # | File | Change | Why |
|---|---|---|---|
| 1 | `gz-waves/src/systems/hydrodynamics/Hydrodynamics.{hh,cc}` | System class renamed `gz::sim::systems::Hydrodynamics` -> `WavesHydrodynamics` | The old name is also gz-sim's stock drag system; the plugin loader keys systems by that name, so a hull carrying both could get either |
| 2 | `gz-waves/CMakeLists.txt` | `GZ_VERSION` defaults to `harmonic` when unset | Upstream defaults to Garden, so a plain `colcon build` of the workspace failed |
| 3 | `gz-waves/include/gz/waves/Algorithm.hh` | `unordered_unique` also clears its `unique` output | Each wave-parameter change appended another full set of wave frequencies; the simulation slowed with every sea-state message (0.94x -> 0.46x real time after 600) |
| 4 | `gz-waves/CMakeLists.txt` | Builds against the OgreNext of the gz-rendering in use (ROS 2 Jazzy's vendored 2.3.3) | gz-cmake picked Ubuntu's OgreNext 2.3.1 first; the GUI could not load the render extension (`libOgreNextMain.so.2.3.1: cannot open shared object file`) |

Details, and how each was found: `docs/07-waves-wind-and-sea-state.md` §2.5
and §3.

## Updating from upstream

```bash
# in a scratch directory
git clone https://github.com/srmainwaring/asv_wave_sim.git upstream && cd upstream
git checkout <new commit>
git apply --3way /path/to/asv_ws/src/asv_gz_waves/patches/gz-waves-asv_ws.patch
# resolve any conflicts, then replace this directory's contents with the
# result (keeping this file), rebuild, and refresh the patch:
#   git diff <new commit> > asv_gz_waves/patches/gz-waves-asv_ws.patch
# then commit here, and in the parent repository commit the moved
# submodule pointer:  git add asv_wave_sim && git commit
```

After an update, re-run the checks in `docs/07` §3: the slowdown test
(per-step cost flat after many `SetWindVelocity` calls), `ldd` on
`libgz-waves1-rendering-ogre2.so` listing only the vendored OgreNext, and
the wind -> Hs calibration (`asv_sea_state/config/gz_waves_calibration.yaml`),
which is only valid for the gz-waves FFT code it was computed with.
