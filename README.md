# Nazi Zombies: Portable on Android (idTech4A++ + FTEQW)

Play [Nazi Zombies: Portable](https://github.com/nzp-team/nzportable) on an Android phone **without a PC**.
The trick: build a custom `libfteqw.so` (the FTEQW engine with NZ:P's engine changes) inside **Termux**,
and drop it into the already-installed **idTech4A++** app using its "Load local engine library" option.
idTech4A++ already provides touch controls, so nothing else needs porting.

Tested on a Samsung Galaxy A32 5G (arm64, Android 12, Mali-G57, 1600x720 landscape).

---

## How the pieces fit

| Piece | What it is | Where it comes from |
|---|---|---|
| idTech4A++ app | Android launcher with touch controls; loads an engine `.so` | GitHub release **v1.1.0harmattan73** (use the GitHub APK, not F-Droid, which is older) |
| `libfteqw.so` | FTEQW engine, built on your phone | Built in Termux from idTech4A++'s source tree + our patches |
| NZ:P assets | models, maps, sounds | `pc-nzp-assets.zip` from the NZ:P assets repo |
| NZ:P game code | `qwprogs.dat`, `csprogs.dat`, `menu.dat` | `nzp-fte-qc.zip` (an FTE build of NZ:P's QuakeC, see "Why an older game-code build") |

Why not just compile `nzp-team/fteqw`? That repo has no glue for the idTech4A++ app
(`sys_android.c`, `gl_vidandroid.c`, the `q3e/` layer). Those only exist in idTech4A++'s own copy of FTEQW, so we
take **its** source and apply NZ:P's engine changes on top as a patch.

---

## Files you need (besides this README)

| File | Purpose |
|---|---|
| `nzp-fteqw-engine.patch` | NZ:P's engine changes (fog command rewrite, `charset` instead of `conchars`, `scripts/` write sandbox, `saveconfig` from gamecode, `EF_FULLBRIGHT`, bolt model paths) applied to idTech4A++'s FTEQW |
| `nzp-fteqw-cursor.patch` | Menu cursor fix for touch (see Fix 3) |
| `nzp-fte-qc.zip` | NZ:P game code built for FTEQW |

Put all three in your phone's **Downloads** folder.

---

## Step 0: Install the app

Install the **GitHub** APK of idTech4A++ **v1.1.0harmattan73** from
<https://github.com/glKarin/com.n0n3m4.diii4a/releases>. The `.so` must come from the same version as the app.

## Step 1: Termux setup

Use the current Termux from F-Droid or GitHub (not the old Play Store build).

```sh
termux-setup-storage
pkg update -y && pkg upgrade -y
pkg install -y git clang cmake ninja make python binutils ndk-sysroot vulkan-headers unzip curl
```

If a package name isn't found, skip it and carry on; the build tells you if something is really missing.

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

Three small edits to the source tree. Each one fixes a build failure that comes from building on Termux
instead of the Android NDK:

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
git apply --check --whitespace=nowarn ~/storage/downloads/nzp-fteqw-engine.patch \
  && git apply --whitespace=nowarn ~/storage/downloads/nzp-fteqw-engine.patch
git apply --check --whitespace=nowarn ~/storage/downloads/nzp-fteqw-cursor.patch \
  && git apply --whitespace=nowarn ~/storage/downloads/nzp-fteqw-cursor.patch
```

If a file in Downloads got a different name (for example `nzp-fteqw-engine (1).patch`), check with
`ls ~/storage/downloads/*.patch`. To undo a patch, run `git apply -R` with the same file.

## Step 5: Make the engine a standalone "nzp" game, and drop Vulkan

Stock FTEQW looks for Quake's `id1/pak0.pak` unless the build config says the engine is a one-game engine.
This config is the stock one plus the NZ:P game definition. It also removes the Vulkan renderer. On the test phone
the engine kept starting Vulkan even with OpenGL ES selected in the app (most likely because it starts its renderer
before the app's `+vid_renderer gl` argument is processed), and the Vulkan path crashed in `VK_SCR_GrabBackBuffer`
on the Mali-G57. With Vulkan removed, only OpenGL ES is left.

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

The first build compiles a few hundred files, so it takes a while. Keep Termux in the foreground
(Android can kill background jobs). If you run low on memory, use `-j2`.

What the extra flags do:

| Flag | Why |
|---|---|
| `CMAKE_POLICY_VERSION_MINIMUM=3.5` | Termux has CMake 4.x, which rejects some bundled libraries' old minimum version |
| `LIBM` / `LIBDL` | The bundled Lua can't find `libm`/`libdl` by name in Termux |
| `ALSOFT_REQUIRE_OPENSL=OFF` | OpenAL then uses its Oboe backend instead of OpenSL |
| `CMAKE_PLATFORM_NO_VERSIONED_SONAME=ON` | Android's toolchain doesn't version library names (`libpng.so.16`); Termux's CMake does, and the app only has plain names |
| `ZLIB_LIBRARY_RELEASE=/system/lib64/libz.so` | Termux's zlib is `libz.so.1`; Android's is `libz.so` |
| `CMAKE_SKIP_RPATH=ON` | Keeps Termux build-folder paths out of the library |
| `CMAKE_DISABLE_FIND_PACKAGE_GnuTLS=ON` | Termux has GnuTLS, Android doesn't. Without this the library fails to load with `cannot locate symbol "crypto_gnutls"`. The side effect is no HTTPS/DTLS, same as the official Android build |
| `FTE_BUILD_CONFIG` | Uses the `config_nzp.h` from Step 5 |

### Check the result

```sh
readelf -d ~/fte-build/fteqw/libfteqw.so | grep -E "NEEDED|RPATH|RUNPATH"
strings ~/fte-build/fteqw/libfteqw.so | grep -m1 'gfx/charset.png'
```

`NEEDED` should be exactly: `libz.so libbz2.so libjpeg.so libfreetype.so libvorbisfile.so libvorbis.so libogg.so
libpng.so libandroid.so liblog.so libEGL.so libm.so libdl.so libc.so`. There must be no `libc++_shared.so`, no
`liboboe.so`, and no names ending in a version number. The `strings` line prints `gfx/charset.png` when the NZ:P patch is in.

## Step 7: Install the library into the app

```sh
mkdir -p ~/storage/shared/diii4a/fteqw
cp ~/fte-build/fteqw/libfteqw.so ~/storage/shared/diii4a/fteqw/
ls -l ~/storage/shared/diii4a/fteqw/libfteqw.so
```

In idTech4A++:
1. Open the **debug preferences** and turn on **Load local engine library**. The app copies the file in at each launch.
2. On the **FTEQW** page, set the game/mod to `nzp`. The app then starts the engine with `-nzp`, which selects the NZ:P game from Step 5. (If you can't find that field, put `-nzp` in the command-line field.)
3. The "Renderer backend" radio button no longer matters, because OpenGL ES is the only renderer left.

## Step 8: Game data

Everything goes in `diii4a/fteqw/nzp/`.

```sh
cd ~/storage/shared/diii4a/fteqw
curl -L -o ~/pc-nzp-assets.zip https://github.com/nzp-team/assets/releases/download/newest/pc-nzp-assets.zip
unzip -o ~/pc-nzp-assets.zip
ls nzp | head -30        # should show the asset folders; there must NOT be an nzp/nzp
```

Then the game code. **Use `nzp-fte-qc.zip` from this project, not NZ:P's `standard-nzp-qc.zip`** (reason below).

```sh
cd ~/storage/shared/diii4a/fteqw/nzp
[ -f progs.dat ] && mv progs.dat progs.dat.standard
unzip -o ~/storage/downloads/nzp-fte-qc.zip
ls -la qwprogs.dat csprogs.dat menu.dat
```

The `progs.dat` rename matters. If a `progs.dat` and a `qwprogs.dat` sit in the same folder, FTEQW picks `progs.dat`
for single player.

Start the game from idTech4A++. The NZ:P menu should load.

---

## Fixes for things you will probably notice

### Fix 1: Only horizontal mouse (look) works, you can't look up or down
Go to **Options → Controls → Invert Mouse** and switch it to **Disabled**.

If you prefer the console: type `m_pitch` to see the value. The engine default is `0.022`. If it is negative, set
`m_pitch 0.022`. If it is positive and the look still feels backwards, set `m_pitch -0.022`.

### Fix 2: The picture is too dark
Open the console and type:
```
gamma 1.5
```
In this engine a **higher** `gamma` is **brighter**. Try values between 1.3 and 2. If that is not enough, add
`brightness 0.1` or raise `contrast`. These settings are saved when you quit the game normally, so you only set them once.

### Fix 3: The menu cursor is offset (you have to drag far past the button to select it)
idTech4A++ sends touch to FTEQW as relative mouse movement. FTEQW added those raw pixel movements straight onto its
menu cursor, but the cursor lives in the engine's scaled 2D coordinates, and clamped it to the wrong size. That is
what `nzp-fteqw-cursor.patch` corrects (Step 4). This patch is **not yet confirmed on a device**. If the cursor is
worse after applying it, undo it with `git apply -R` and rebuild.

### Fix 4: It crashed on start or looks wrong (Vulkan)
Fixed by the `#undef VKQUAKE` in Step 5. If you skipped that, rebuild with it.

### Fix 5: Starting a map from the menu sends you back to the menu
That is the `Builtin 601: not implemented` / `Mod is not compatible` error described below. Use `nzp-fte-qc.zip`
and rename `progs.dat` as in Step 8.

---

## Why an older game-code build?

NZ:P's QuakeC has two builds: a "standard" one for NZ:P's own Vril engine, and an FTE one. Only the standard build is
published as a release. On 2026-09-23 NZ:P added JSON loading (`json_parse_file` and others) to the shared code.
Those functions are engine builtins that exist only in Vril. FTEQW has no `#601` server builtin (it is `setkeydest`,
a menu function), so any map start fails with:

```
Mod forgot to ensure support for builtin 601
SV_Error: Builtin 601: not implemented.
Mod is not compatible.
```

The FTE build still compiles from NZ:P's source at commit `6ed6305` (2026-09-03), the last revision from before FTE
compilation was dropped from their build script. `nzp-fte-qc.zip` is that build.
Because the assets you download are newer than that code, a missing model or sound could in principle show up
in the console. No such problem has been reported so far.

### Rebuild the QC yourself (needs a PC/Linux x86-64 or macOS; `fteqcc-cli-lin` is not an ARM binary)

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

Recent `pandas` versions break `qc_hash_generator.py`: it writes the names unhashed and `fteqcc` then fails
with `Unknown value "misc"`. In `read_csv_data()`, replace the `for value in csv_data.values:` loop with:

```python
csv_data[csv_data.columns[0]] = csv_data[csv_data.columns[0]].astype(object)
for i in range(len(csv_data)):
    v = csv_data.iat[i, 0]
    original_lengths.append(len(v))
    original_names.append(v)
    csv_data.iat[i, 0] = int(crc16.ibm_3740(str.encode(v)))
```

---

## Troubleshooting

| Symptom | Cause / fix |
|---|---|
| `cmake`: `CANNOT LINK EXECUTABLE ... __hash_memory` | Out-of-sync Termux packages. `pkg upgrade -y`; if it persists, `termux-change-repo`, then `pkg upgrade -y` and `apt reinstall libc++ cmake` |
| `Compatibility with CMake < 3.5 has been removed` | Missing `-DCMAKE_POLICY_VERSION_MINIMUM=3.5` |
| Lua: `libm` not found | Missing `-DLIBM`/`-DLIBDL` (Step 6) |
| `GLES/gl.h` not found | Run `export CPATH=...depincs/opengl` again in each new Termux session |
| `q3e_android.h` can't find `sys_android.h` | The `git show HEAD:...` line in Step 2 |
| App: `Unable to load library '...libfteqw.so': (null)` | The app's logger throws the real reason away. Test the library from Termux (see "Testing the library from Termux" below). The usual cause is `cannot locate symbol "crypto_gnutls"` (the GnuTLS flag) |
| App says no game files / wants Quake | Config from Step 5 not used, or the game isn't `nzp` (Step 7) |
| Crash in `VK_SCR_GrabBackBuffer` | Vulkan, see Fix 4 |
| Map start returns to the menu | Old game code, see Fix 5 |
| Build dies silently | Android killed Termux. `termux-wake-lock`, keep Termux open, use `-j2` |

### Testing the library from Termux

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

then:
```sh
clang ~/dltest.c -o ~/dltest -ldl
LP=$(find ~/fte-build -name '*.so' -exec dirname {} \; | sort -u | tr '\n' ':')
LD_LIBRARY_PATH="$LP" ~/dltest ~/fte-build/fteqw/libfteqw.so
```

It prints `LOAD OK`, or the real error (for example the first missing symbol).

### Reading the engine's own log

Add `-condebug` to the launcher's command line, play until the problem happens, quit, then:

```sh
find ~/storage/shared/diii4a -name qconsole.log -exec tail -80 {} \;
```

Use `~/storage/shared/diii4a` (with the real folder name), **not** `~/storage/shared`. `find` doesn't descend into a
symlink given as its starting point, so the shorter path prints nothing. The app's `logcat` does not contain the
engine's console output.

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
- The cursor patch (Fix 3) is not yet confirmed on a device.

## Licences and credits

FTEQW, NZ:P and idTech4A++ are all GPL. If you share a built `libfteqw.so`, also share (or link to) the source and the
patches that went into it. Engine: FTEQW by Spike and contributors, NZ:P fork by the NZ:P Team.
Android port: glKarin's idTech4A++. This guide only describes how the pieces fit together.
