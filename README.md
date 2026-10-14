<div align="center">

# Nazi Zombies: Portable on Android

**Build the NZ:P engine on your phone and run it inside idTech4A++. No PC needed.**

![Platform](https://img.shields.io/badge/platform-Android%20arm64-3DDC84?logo=android&logoColor=white)
![Built with](https://img.shields.io/badge/built%20with-Termux-1f1f1f)
![Engine](https://img.shields.io/badge/engine-FTEQW-e8590c)
![App](https://img.shields.io/badge/app-idTech4A%2B%2B%20v1.1.0harmattan73-0b7285)
![Tested](https://img.shields.io/badge/tested-Galaxy%20A32%205G-2f9e44)
![Licence](https://img.shields.io/badge/licence-GPL--2.0--or--later-1c7ed6)

[Overview](#overview) ·
[Build it](#step-0-install-the-app) ·
[Fixes](#fixes-for-things-you-will-probably-notice) ·
[Troubleshooting](#troubleshooting) ·
[Licences](#licences-and-credits)

</div>

> [!NOTE]
> Unofficial guide. It is not affiliated with the NZ:P Team, the FTEQW project or the idTech4A++ author.
> It only describes how their software fits together.

---

## Contents

1. [Overview](#overview)
2. [What is in this repository](#what-is-in-this-repository)
3. [Step 0: Install the app](#step-0-install-the-app)
4. [Step 1: Termux setup](#step-1-termux-setup)
5. [Step 2: Get the source](#step-2-get-the-source-sparse-clone-about-285-mb-instead-of-13-gb)
6. [Step 3: Make Termux's toolchain behave like Android's](#step-3-make-termuxs-toolchain-behave-like-androids)
7. [Step 4: Apply the NZ:P patches](#step-4-apply-the-nzp-patches)
8. [Step 5: Make the engine a standalone "nzp" game, and drop Vulkan](#step-5-make-the-engine-a-standalone-nzp-game-and-drop-vulkan)
9. [Step 6: Configure and build](#step-6-configure-and-build)
10. [Step 7: Install the library into the app](#step-7-install-the-library-into-the-app)
11. [Step 8: Game data](#step-8-game-data)
12. [Fixes for things you will probably notice](#fixes-for-things-you-will-probably-notice)
13. [Why an older game-code build?](#why-an-older-game-code-build)
14. [Troubleshooting](#troubleshooting)
15. [Known limits](#known-limits)
16. [Licences and credits](#licences-and-credits)

---

## Overview

[Nazi Zombies: Portable](https://github.com/nzp-team/nzportable) (NZ:P) runs on a Quake engine called FTEQW.
The idea here is to build a custom **`libfteqw.so`** (FTEQW plus NZ:P's engine changes) **inside Termux** and drop it into the
already-installed **idTech4A++** app, using that app's *Load local engine library* option. idTech4A++ already provides
the touch controls, so nothing else needs porting.

Tested on a Samsung Galaxy A32 5G (arm64, Android 12, Mali-G57, 1600x720 landscape).

### How the pieces fit

| Piece | What it is | Where it comes from |
|---|---|---|
| **idTech4A++ app** | Android launcher with touch controls. It loads an engine `.so`. | GitHub release **v1.1.0harmattan73**. Use the GitHub APK, not F-Droid, which is older. |
| **`libfteqw.so`** | The FTEQW engine, built on your phone | Built in Termux from idTech4A++'s source tree plus the patches in this repository |
| **NZ:P assets** | Models, maps, sounds | `pc-nzp-assets.zip` from the NZ:P assets repository |
| **NZ:P game code** | `qwprogs.dat`, `csprogs.dat`, `menu.dat` | `nzp-fte-qc.zip` (an FTE build of NZ:P's QuakeC, see [Why an older game-code build?](#why-an-older-game-code-build)) |

> [!TIP]
> **Why not just compile `nzp-team/fteqw`?** That repository has no glue for the idTech4A++ app (`sys_android.c`,
> `gl_vidandroid.c`, the `q3e/` layer). Those exist only in idTech4A++'s own copy of FTEQW, so we take **its** source
> and apply NZ:P's engine changes on top as a patch.

---

## What is in this repository

| File | Purpose | Licence |
|---|---|---|
| `README.md` | This guide | GPL-2.0-or-later |
| `nzp-fteqw-engine.patch` | NZ:P's engine changes (fog command rewrite, `charset` instead of `conchars`, `scripts/` write sandbox, `saveconfig` from game code, `EF_FULLBRIGHT`, bolt model paths) applied to idTech4A++'s FTEQW | GPL-2.0-or-later |
| `nzp-fteqw-cursor.patch` | Menu cursor fix for touch ([Fix 4](#fix-4-the-menu-cursor-is-offset)) | GPL-2.0-or-later |
| `nzp-fteqw-settings.patch` | Makes your settings survive a restart ([Fix 3](#fix-3-the-settings-you-change-come-back-to-default-after-a-restart)) | GPL-2.0-or-later |
| `nzp-fte-qc.zip` | NZ:P game code built for FTEQW, with its licence and source notice | GPL-2.0 |
| `LICENSE` | GNU GPL version 2 | |
| `LICENSE-GPL-3.0.txt`, `LICENSE-LGPL-2.0-OpenAL-Soft.txt`, `LICENSE-CC-BY-SA-4.0.txt` | Licence texts for idTech4A++ (GPL-3.0), OpenAL Soft (LGPL-2.0) and the NZ:P assets (CC BY-SA 4.0) | |
| `THIRD_PARTY_LICENSES.md` | Every component, its licence, and the notices that must travel with a built library | |

Put the three `.patch` files and `nzp-fte-qc.zip` in your phone's **Downloads** folder before you start.
The built `libfteqw.so` is optional to share; see [Licences and credits](#licences-and-credits) first.

---

## Step 0: Install the app

Install the **GitHub** APK of idTech4A++ **v1.1.0harmattan73** from
<https://github.com/glKarin/com.n0n3m4.diii4a/releases>.

> [!IMPORTANT]
> The `.so` you build must come from the **same version** as the app.

## Step 1: Termux setup

Use the current Termux from F-Droid or GitHub, not the old Play Store build.

```sh
termux-setup-storage
pkg update -y && pkg upgrade -y
pkg install -y git clang cmake ninja make python binutils ndk-sysroot vulkan-headers unzip curl
```

If a package name isn't found, skip it and carry on. The build tells you if something is really missing.

## Step 2: Get the source (sparse clone, about 285 MB instead of 1.3 GB)

```sh
cd ~
git clone --depth 1 --branch v1.1.0harmattan73 --filter=blob:none --sparse \
  https://github.com/glKarin/com.n0n3m4.diii4a.git
cd com.n0n3m4.diii4a
git sparse-checkout set --no-cone \
  /Q3E/src/main/jni/CMakeLists.txt \
  /Q3E/src/main/jni/q3e/ /Q3E/src/main/jni/common/ \
  /Q3E/src/main/jni/deplibs/ /Q3E/src/main/jni/depincs/ \
  /Q3E/src/main/jni/fteqw/ \
  /doom3/neo/externlibs/openal-soft/ \
  /doom3/neo/sys/android/

# One small header lives outside the folders above. If this prints "No such file", fetch it directly:
ls doom3/neo/sys/android/sys_android.h || {
  mkdir -p doom3/neo/sys/android
  git show HEAD:doom3/neo/sys/android/sys_android.h > doom3/neo/sys/android/sys_android.h
}
```

## Step 3: Make Termux's toolchain behave like Android's

Three small edits to the source tree. Each one fixes a build failure that comes from building on Termux instead of the
Android NDK.

```sh
cd ~/com.n0n3m4.diii4a/Q3E/src/main/jni

# fluidsynth's test/doc folders aren't in the repo and FTEQW doesn't use fluidsynth
sed -i 's|^add_subdirectory (fluidsynth)|#add_subdirectory (fluidsynth)|' deplibs/CMakeLists.txt

# SDL2 wants the NDK's cpu-features.c (Termux has no NDK); FTEQW doesn't use SDL on Android
sed -i 's|^set(Q3E_SDL TRUE)|set(Q3E_SDL FALSE)|' CMakeLists.txt

# Nothing calls the Oboe helper, and linking it drags in libc++_shared.so, which the app can't supply
sed -i 's|^\(\s*\)set(FTE_LIBS \${FTE_LIBS} oboeaudio)|\1#set(FTE_LIBS ${FTE_LIBS} oboeaudio)|' fteqw/CMakeLists.txt
grep -n oboeaudio fteqw/CMakeLists.txt    # the line must now start with #
```

## Step 4: Apply the NZ:P patches

```sh
cd ~/com.n0n3m4.diii4a
for p in engine cursor settings; do
  f=~/storage/downloads/nzp-fteqw-$p.patch
  git apply --check --whitespace=nowarn "$f" && git apply --whitespace=nowarn "$f" && echo "applied: $p"
done
```

You should see three `applied:` lines. If a file in Downloads got a different name (for example
`nzp-fteqw-engine (1).patch`), check with `ls ~/storage/downloads/*.patch`. To undo one patch, run `git apply -R` with
the same file.

## Step 5: Make the engine a standalone "nzp" game, and drop Vulkan

Stock FTEQW looks for Quake's `id1/pak0.pak` unless the build config says the engine is a one-game engine. The config
below is the stock one plus the NZ:P game definition.

It also removes the Vulkan renderer. On the test phone the engine kept starting Vulkan even with OpenGL ES selected in
the app (most likely because it starts its renderer before the app's `+vid_renderer gl` argument is processed), and the
Vulkan path crashed in `VK_SCR_GrabBackBuffer` on the Mali-G57. With Vulkan removed, only OpenGL ES is left.

```sh
cd ~/com.n0n3m4.diii4a/Q3E/src/main/jni/fteqw/engine/common
{ cat <<'EOF'
#define DISTRIBUTION "NZ:P"
#define DISTRIBUTIONLONG "NZ:P Team"
#define FULLENGINENAME "Nazi Zombies: Portable"
#define ENGINEWEBSITE "https://docs.nzp.gay/"
#define GAME_SHORTNAME "nzp"
#define GAME_FULLNAME "Nazi Zombies: Portable"
#define GAME_BASEGAMES "nzp"
#define GAME_PROTOCOL "NZP-REBOOT"
EOF
cat config_fteqw.h; } > config_nzp.h
sed -i 's|^//#undef VKQUAKE|#undef VKQUAKE|' config_nzp.h
grep -n 'undef VKQUAKE' config_nzp.h     # must show one line starting with #undef
```

## Step 6: Configure and build

```sh
cd ~/com.n0n3m4.diii4a/Q3E/src/main/jni
export CPATH=$HOME/com.n0n3m4.diii4a/Q3E/src/main/jni/depincs/opengl

LIBM=$(ls $PREFIX/lib/libm.so /system/lib64/libm.so 2>/dev/null | head -1)
LIBDL=$(ls $PREFIX/lib/libdl.so /system/lib64/libdl.so 2>/dev/null | head -1)
echo "libm=$LIBM libdl=$LIBDL"          # both must be non-empty

OFF=""
for g in DOOM3 RTCW QUAKE3 QUAKE2 QUAKE1 TDM DOOM3BFG DOOM ETW REALRTCW JK URT \
         OPENMOHAA WOLF3D SKINDEEP QC ICARUS SERIOUSSAM XASH3D SOURCE; do
  OFF="$OFF -DGAME_$g=OFF"
done

cmake -S . -B ~/fte-build -G Ninja -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_POLICY_VERSION_MINIMUM=3.5 -DLIBM=$LIBM -DLIBDL=$LIBDL \
  -DDIII4A_PRODUCT_FLAVOR=github -DGAME_FTEQW=ON -DALSOFT_REQUIRE_OPENSL=OFF $OFF \
  -DCMAKE_PLATFORM_NO_VERSIONED_SONAME=ON -DZLIB_LIBRARY_RELEASE=/system/lib64/libz.so \
  -DCMAKE_SKIP_RPATH=ON -DCMAKE_DISABLE_FIND_PACKAGE_GnuTLS=ON \
  -DFTE_BUILD_CONFIG=$HOME/com.n0n3m4.diii4a/Q3E/src/main/jni/fteqw/engine/common/config_nzp.h

termux-wake-lock
ninja -C ~/fte-build -j4 fteqw
```

> [!TIP]
> The first build compiles a few hundred files, so it takes a while. Keep Termux in the foreground, because Android can
> kill background jobs. If you run low on memory, use `-j2`.

<details>
<summary><b>What do the extra flags do?</b></summary>

| Flag | Why |
|---|---|
| `CMAKE_POLICY_VERSION_MINIMUM=3.5` | Termux has CMake 4.x, which rejects some bundled libraries' old minimum version |
| `LIBM` / `LIBDL` | The bundled Lua can't find `libm`/`libdl` by name in Termux |
| `ALSOFT_REQUIRE_OPENSL=OFF` | OpenAL then uses its Oboe backend instead of OpenSL |
| `CMAKE_PLATFORM_NO_VERSIONED_SONAME=ON` | Android's toolchain doesn't version library names (`libpng.so.16`). Termux's CMake does, and the app only has plain names |
| `ZLIB_LIBRARY_RELEASE=/system/lib64/libz.so` | Termux's zlib is `libz.so.1`. Android's is `libz.so` |
| `CMAKE_SKIP_RPATH=ON` | Keeps Termux build-folder paths out of the library |
| `CMAKE_DISABLE_FIND_PACKAGE_GnuTLS=ON` | Termux has GnuTLS, Android doesn't. Without this the library fails to load with `cannot locate symbol "crypto_gnutls"`. The side effect is no HTTPS/DTLS, same as the official Android build |
| `FTE_BUILD_CONFIG` | Uses the `config_nzp.h` from Step 5 |

</details>

### Check the result

```sh
readelf -d ~/fte-build/fteqw/libfteqw.so | grep -E "NEEDED|RPATH|RUNPATH"
strings ~/fte-build/fteqw/libfteqw.so | grep -m1 'gfx/charset.png'
```

`NEEDED` should be exactly:

```text
libz.so libbz2.so libjpeg.so libfreetype.so libvorbisfile.so libvorbis.so libogg.so
libpng.so libandroid.so liblog.so libEGL.so libm.so libdl.so libc.so
```

There must be no `libc++_shared.so`, no `liboboe.so`, and no names ending in a version number. The `strings` line prints
`gfx/charset.png` when the NZ:P patch is in.

## Step 7: Install the library into the app

```sh
mkdir -p ~/storage/shared/diii4a/fteqw
cp ~/fte-build/fteqw/libfteqw.so ~/storage/shared/diii4a/fteqw/
ls -l ~/storage/shared/diii4a/fteqw/libfteqw.so
```

In idTech4A++:

1. Open the **debug preferences** and turn on **Load local engine library**. The app copies the file in at each launch.
2. On the **FTEQW** page, set the game/mod to `nzp`. The app then starts the engine with `-nzp`, which selects the NZ:P
   game from Step 5. If you can't find that field, put `-nzp` in the command-line field.
3. The "Renderer backend" radio button no longer matters, because OpenGL ES is the only renderer left.

## Step 8: Game data

Everything goes in `diii4a/fteqw/nzp/`.

```sh
cd ~/storage/shared/diii4a/fteqw
curl -L -o ~/pc-nzp-assets.zip https://github.com/nzp-team/assets/releases/download/newest/pc-nzp-assets.zip
unzip -o ~/pc-nzp-assets.zip
ls nzp | head -30        # should show the asset folders; there must NOT be an nzp/nzp
```

Then the game code. **Use `nzp-fte-qc.zip` from this project, not NZ:P's `standard-nzp-qc.zip`** (reason
[below](#why-an-older-game-code-build)).

```sh
cd ~/storage/shared/diii4a/fteqw/nzp
[ -f progs.dat ] && mv progs.dat progs.dat.standard
unzip -o ~/storage/downloads/nzp-fte-qc.zip
ls -la qwprogs.dat csprogs.dat menu.dat
```

> [!IMPORTANT]
> The `progs.dat` rename matters. If a `progs.dat` and a `qwprogs.dat` sit in the same folder, FTEQW picks `progs.dat`
> for single player.

Start the game from idTech4A++. The NZ:P menu should load.

---

## Fixes for things you will probably notice

| Symptom | Go to |
|---|---|
| You can only look left and right, not up and down | [Fix 1](#fix-1-only-horizontal-mouse-look-works) |
| The picture is too dark | [Fix 2](#fix-2-the-picture-is-too-dark) |
| **Settings you change come back to default after a restart** | [Fix 3](#fix-3-the-settings-you-change-come-back-to-default-after-a-restart) |
| You have to drag far past a menu button to select it | [Fix 4](#fix-4-the-menu-cursor-is-offset) |
| Crash on start or a broken picture | [Fix 5](#fix-5-crash-on-start-or-it-looks-wrong-vulkan) |
| Starting a map sends you back to the menu | [Fix 6](#fix-6-starting-a-map-from-the-menu-sends-you-back-to-the-menu) |

### Fix 1: Only horizontal mouse (look) works

You can't look up or down.

Go to **Options → Controls → Invert Mouse** and switch it to **Disabled**, then press **APPLY**.

Console alternative: type `m_pitch` to see the value. The engine default is `0.022`. If it is negative, set
`m_pitch 0.022`. If it is positive and the look still feels backwards, set `m_pitch -0.022`.

### Fix 2: The picture is too dark

Open the console and type:

```text
gamma 1.5
```

In this engine a **higher** `gamma` is **brighter**. Try values between 1.3 and 2. If that is not enough, add
`brightness 0.1` or raise `contrast`. With [Fix 3](#fix-3-the-settings-you-change-come-back-to-default-after-a-restart)
in place these are remembered, so you only set them once.

### Fix 3: The settings you change come back to default after a restart

**What was going wrong** (found by reading the engine and the NZ:P menu code):

1. NZ:P's menus save with the command `saveconfig user_settings.cfg`, **but FTEQW never loads that file at startup**.
   It only runs `default.cfg`, `config.cfg` / `fte.cfg` and `autoexec.cfg`.
2. FTEQW's own "save settings" step lives in its quit prompt. The Android app just ends the process, so that prompt never runs.
3. The assets pack ships a `nzp/config.cfg` that was written for NZ:P's other engine. It is executed on every start and
   sets values such as `gamma "0.6"`, `m_pitch "0"` and `sensitivity "7.5"`. This is the likely reason for the dark picture
   and the dead vertical look on first start.

> [!IMPORTANT]
> In NZ:P's own menus a page is only saved when you press **APPLY** (Controls, Video, Audio, Gamepad). Pressing **BACK**
> does not save. Do this first, whichever fix you use below.

**Fix A: no rebuild needed.** Works with the library you already have. Make the game load the file its menus write:

```sh
cd ~/storage/shared/diii4a/fteqw/nzp
echo 'exec user_settings.cfg' > autoexec.cfg
```

`autoexec.cfg` runs after the assets pack's `config.cfg`, so your saved values win. Settings you change from the
console (not from the NZ:P menus) are not saved automatically with this fix. After changing one, type this in the console:

```text
saveconfig user_settings.cfg
```

**Fix B: the patched library (recommended).** `nzp-fteqw-settings.patch` is applied in [Step 4](#step-4-apply-the-nzp-patches)
and does three things:

- loads `user_settings.cfg` at startup (so Fix A is built in), after `config.cfg` and `autoexec.cfg`;
- saves any changed settings when the app is **paused** (Android can kill a paused app without warning), when you
  **close** the game, and when the in-game `quit` command runs;
- writes to the same `user_settings.cfg` that the NZ:P menus use, so there is only one settings file.

You can keep the `autoexec.cfg` from Fix A as well. It does no harm.

**Check that it works.** Change something in an NZ:P menu, press **APPLY**, close the game completely, start it again.
The setting should still be there. To look at the saved file:

```sh
find ~/storage/shared/diii4a -name user_settings.cfg -exec cat {} \;
```

It lists lines such as `m_pitch "0.022"` or `gamma "1.5"`. If the file does not exist at all after you pressed APPLY,
the game is not writing to that folder. Send me the `qconsole.log` (see [Reading the engine's own log](#reading-the-engines-own-log)).

### Fix 4: The menu cursor is offset

You have to drag far past the button to select it.

idTech4A++ sends touch to FTEQW as relative mouse movement. FTEQW added those raw pixel movements straight onto its menu
cursor, but the cursor lives in the engine's scaled 2D coordinates, and clamped it to the wrong size.
`nzp-fteqw-cursor.patch` ([Step 4](#step-4-apply-the-nzp-patches)) corrects that. Confirmed working on the test device.

### Fix 5: Crash on start or it looks wrong (Vulkan)

Fixed by the `#undef VKQUAKE` in [Step 5](#step-5-make-the-engine-a-standalone-nzp-game-and-drop-vulkan). If you skipped
that, redo it and rebuild.

### Fix 6: Starting a map from the menu sends you back to the menu

That is the `Builtin 601: not implemented` / `Mod is not compatible` error described in the next section. Use
`nzp-fte-qc.zip` and rename `progs.dat` as in [Step 8](#step-8-game-data).

---

## Why an older game-code build?

NZ:P's QuakeC has two builds: a "standard" one for NZ:P's own Vril engine, and an FTE one. Only the standard build is
published as a release. On 2026-09-23 NZ:P added JSON loading (`json_parse_file` and others) to the shared code. Those
functions are engine builtins that exist only in Vril. FTEQW has no `#601` server builtin (it is `setkeydest`, a menu
function), so any map start fails with:

```text
Mod forgot to ensure support for builtin 601
SV_Error: Builtin 601: not implemented.
Mod is not compatible.
```

The FTE build still compiles from NZ:P's source at commit `6ed6305` (2026-09-03), the last revision from before FTE
compilation was dropped from their build script. `nzp-fte-qc.zip` is that build. Because the assets you download are
newer than that code, a missing model or sound could in principle show up in the console. No such problem has been
reported so far.

<details>
<summary><b>Rebuild the QC yourself</b> (needs a PC with Linux x86-64 or macOS; <code>fteqcc-cli-lin</code> is not an ARM binary)</summary>

```sh
git clone https://github.com/nzp-team/quakec && cd quakec
git checkout 6ed6305
pip install pandas colorama fastcrc
python3 bin/qc_hash_generator.py -i tools/asset_conversion_table.csv -o source/server/hash_table.qc
mkdir -p build/fte
bin/fteqcc-cli-lin -O3 -DFTE -srcfile progs/ssqc.src
bin/fteqcc-cli-lin -DFTE -srcfile progs/csqc.src
bin/fteqcc-cli-lin -O3 -DFTE -srcfile progs/menu.src
ls build/fte     # qwprogs.dat csprogs.dat menu.dat
```

Recent `pandas` versions break `qc_hash_generator.py`: it writes the names unhashed and `fteqcc` then fails with
`Unknown value "misc"`. In `read_csv_data()`, replace the `for value in csv_data.values:` loop with:

```python
csv_data[csv_data.columns[0]] = csv_data[csv_data.columns[0]].astype(object)
for i in range(len(csv_data)):
    v = csv_data.iat[i, 0]
    original_lengths.append(len(v))
    original_names.append(v)
    csv_data.iat[i, 0] = int(crc16.ibm_3740(str.encode(v)))
```

</details>

---

## Troubleshooting

| Symptom | Cause / fix |
|---|---|
| `cmake`: `CANNOT LINK EXECUTABLE ... __hash_memory` | Out-of-sync Termux packages. Run `pkg upgrade -y`. If it persists, run `termux-change-repo`, then `pkg upgrade -y` and `apt reinstall libc++ cmake` |
| `Compatibility with CMake < 3.5 has been removed` | Missing `-DCMAKE_POLICY_VERSION_MINIMUM=3.5` |
| Lua: `libm` not found | Missing `-DLIBM` / `-DLIBDL` ([Step 6](#step-6-configure-and-build)) |
| `GLES/gl.h` not found | Run `export CPATH=...depincs/opengl` again in each new Termux session |
| `q3e_android.h` can't find `sys_android.h` | The `git show HEAD:...` line in [Step 2](#step-2-get-the-source-sparse-clone-about-285-mb-instead-of-13-gb) |
| A patch fails with `git apply --check` | You are not on tag `v1.1.0harmattan73`, or the patch was already applied. `git status` shows which files changed |
| App: `Unable to load library '...libfteqw.so': (null)` | The app's logger throws the real reason away. Test the library from Termux (below). The usual cause is `cannot locate symbol "crypto_gnutls"` (the GnuTLS flag) |
| App says no game files / wants Quake | The config from Step 5 is not used, or the game isn't `nzp` ([Step 7](#step-7-install-the-library-into-the-app)) |
| Crash in `VK_SCR_GrabBackBuffer` | Vulkan, see [Fix 5](#fix-5-crash-on-start-or-it-looks-wrong-vulkan) |
| Map start returns to the menu | Old game code, see [Fix 6](#fix-6-starting-a-map-from-the-menu-sends-you-back-to-the-menu) |
| Settings reset on restart | [Fix 3](#fix-3-the-settings-you-change-come-back-to-default-after-a-restart). Remember to press APPLY |
| Build dies silently | Android killed Termux. Run `termux-wake-lock`, keep Termux open, use `-j2` |

<details>
<summary><b>Testing the library from Termux</b></summary>

Save this as `~/dltest.c`:

```c
#include <dlfcn.h>
#include <stdio.h>
int main(int c, char **v) {
  void *h = dlopen(v[1], RTLD_NOW | RTLD_GLOBAL);
  if (!h) { printf("dlopen failed: %s\n", dlerror()); return 1; }
  puts("LOAD OK");
  return 0;
}
```

Then:

```sh
clang ~/dltest.c -o ~/dltest -ldl
LP=$(find ~/fte-build -name '*.so' -exec dirname {} \; | sort -u | tr '\n' ':')
LD_LIBRARY_PATH="$LP" ~/dltest ~/fte-build/fteqw/libfteqw.so
```

It prints `LOAD OK`, or the real error (for example the first missing symbol).

</details>

### Reading the engine's own log

Add `-condebug` to the launcher's command line, play until the problem happens, quit, then:

```sh
find ~/storage/shared/diii4a -name qconsole.log -exec tail -80 {} \;
```

> [!WARNING]
> Use `~/storage/shared/diii4a` (with the real folder name), **not** `~/storage/shared`. `find` doesn't descend into a
> symlink given as its starting point, so the shorter path prints nothing. The app's `logcat` does not contain the
> engine's console output.

### Rebuilding after a change

```sh
cd ~/com.n0n3m4.diii4a
export CPATH=$HOME/com.n0n3m4.diii4a/Q3E/src/main/jni/depincs/opengl
ninja -C ~/fte-build -j4 fteqw
cp ~/fte-build/fteqw/libfteqw.so ~/storage/shared/diii4a/fteqw/
```

Only changed files recompile. Changing `config_nzp.h` recompiles everything.

---

## Known limits

- No HTTPS/DTLS in the engine (no GnuTLS on Android).
- Vulkan is removed on purpose.
- The game code is pinned to 2026-09-03 and won't get newer NZ:P gameplay features until FTEQW gets the JSON builtins.
- The startup lines `GLES_GetSymbol: couldn't find glArrayElement` (and similar) and `libfteplug_GLTF.so not found` are harmless.
- The settings patch saves when the app is paused or closed. If Android kills the app during a crash, changes made since
  the last pause or APPLY are lost.

---

## Licences and credits

Everything here is free software, and most of it is under the GPL. The short version:

| Component | Licence | Full text / details |
|---|---|---|
| This repository's patches and README | GPL-2.0-or-later | [`LICENSE`](LICENSE) |
| FTEQW engine | GPL-2.0-or-later (code derived from id Software's Quake) | [`LICENSE`](LICENSE) |
| NZ:P engine changes (`nzp-team/fteqw`) | GPL-2.0 | [`LICENSE`](LICENSE) |
| idTech4A++ app and Android glue | GPL-3.0 | [`LICENSE-GPL-3.0.txt`](LICENSE-GPL-3.0.txt) |
| NZ:P game code (`nzp-team/quakec`, built into `nzp-fte-qc.zip`) | GPL-2.0 | inside `nzp-fte-qc.zip` under `licenses/nzp-fte-qc/` |
| NZ:P assets (`nzp-team/assets`, downloaded by you) | CC BY-SA 4.0, plus a few third-party items | [`LICENSE-CC-BY-SA-4.0.txt`](LICENSE-CC-BY-SA-4.0.txt), and the `nzp/licenses/` folder of the assets pack |
| OpenAL Soft (inside the idTech4A++ app) | LGPL-2.0-or-later | [`LICENSE-LGPL-2.0-OpenAL-Soft.txt`](LICENSE-LGPL-2.0-OpenAL-Soft.txt) |
| bzip2, libjpeg, libpng, FreeType, libogg, libvorbis, zlib | Permissive licences, FreeType under its GPL option | [`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md) |

### If you share a built `libfteqw.so`

A built `libfteqw.so` combines GPL-2.0-or-later engine code with GPL-3.0 Android glue, so it is distributed under
**GPL version 3** as a whole. If you give it to anyone, the GPL says they must be able to get the matching source.
For this project that source is:

1. idTech4A++ at tag `v1.1.0harmattan73` (commit `ce49510f`), <https://github.com/glKarin/com.n0n3m4.diii4a>;
2. the three `nzp-fteqw-*.patch` files, the three `sed` edits from Step 3 and the `config_nzp.h` from Step 5, all of which
   are in this repository;
3. this README, which is the build recipe.

Link to this repository when you share the library, keep `LICENSE`, the `LICENSE-*.txt` files and `THIRD_PARTY_LICENSES.md` together
with it, and keep the notices in `THIRD_PARTY_LICENSES.md` intact. The same applies to `nzp-fte-qc.zip`, whose source is
NZ:P's QuakeC at commit `6ed6305` (the zip carries its own licence and source notice).

> [!NOTE]
> This is a summary in good faith, not legal advice. When in doubt, read the licence texts above.

The NZ:P game assets are not part of this repository. Download them from the official NZ:P release page, which keeps
their licences and attributions together.

### Credits

- **FTEQW**: Spike and contributors (<https://www.fteqw.org/>), built on id Software's Quake source.
- **NZ:P fork and game**: the [NZ:P Team](https://github.com/nzp-team).
- **Android port**: glKarin's [idTech4A++](https://github.com/glKarin/com.n0n3m4.diii4a).
- Third-party libraries: see [`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md).
