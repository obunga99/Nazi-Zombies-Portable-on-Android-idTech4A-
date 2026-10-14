# Third-party licences and notices

This file lists the software that ends up in, loads into, or is used to build the files described in
[README.md](README.md), together with the licence each one is under. Where a licence requires the notice to travel with
the software (BSD-style licences), the full text is reproduced below.

> This is a good-faith summary, not legal advice. The authoritative text is always the one shipped with each project.

## 1. What is what

### 1.1 Files in this repository

| File | Licence | Notes |
|---|---|---|
| `nzp-fteqw-engine.patch` | GPL-2.0-or-later | Derived from NZ:P's changes to FTEQW (NZ:P fork, GPL-2.0). Modifies files that are GPL-2.0-or-later. |
| `nzp-fteqw-cursor.patch` | GPL-2.0-or-later | Modifies `in_generic.c` (FTEQW, GPL-2.0-or-later). |
| `nzp-fteqw-settings.patch` | GPL-2.0-or-later | Modifies `cl_main.c` (FTEQW, GPL-2.0-or-later) and `sys_android.c` (idTech4A++ glue, GPL-3.0). |
| `nzp-fte-qc.zip` | GPL-2.0 | Compiled from NZ:P's QuakeC (GPL-2.0). Source: see section 2. |
| `README.md` | GPL-2.0-or-later | Documentation for the above. |
| `LICENSE`, `LICENSE-*.txt` | n/a | Licence texts. |

### 1.2 Components of a built `libfteqw.so`

| Component | Licence | How it is used |
|---|---|---|
| FTEQW engine (Spike and contributors; code derived from id Software's Quake) | GPL-2.0-or-later | Compiled into `libfteqw.so`. Source files carry per-file headers. |
| NZ:P engine changes (NZ:P Team, `nzp-team/fteqw`) | GPL-2.0 | Applied through `nzp-fteqw-engine.patch`. |
| idTech4A++ Android glue (`q3e/`, `sys_android.c`, `gl_vidandroid.c`; glKarin) | GPL-3.0 (the project states "License: GPLv3") | Compiled into `libfteqw.so`. |
| Combined `libfteqw.so` | GPL-3.0 in effect | GPL-2.0-or-later code can be combined with GPL-3.0 code; the combined work as a whole is distributed under GPL-3.0. |
| SHA-1 (`common/sha1.c`, Steve Reid) | Public domain | Embedded in the engine source. |
| MD5 (`common/md5.c`, from Libgcrypt) | LGPL-2.1-or-later | Embedded in the engine source. |
| XZ Embedded decoder (Lasse Collin, `common/fs_xz.c`) | Public domain | Embedded in the engine source. |
| zlib | zlib licence | Linked dynamically from Android's system `libz.so`. Not redistributed. |
| bzip2 / libbz2 | bzip2 licence (BSD-style, section 3.1) | Linked dynamically; the `libbz2.so` comes from the idTech4A++ app. |
| libjpeg 9f (IJG) | IJG licence (section 3.2) | Linked dynamically; `libjpeg.so` comes from the idTech4A++ app. |
| libpng 1.6.44 | PNG Reference Library License v2 (section 3.3) | Linked dynamically; `libpng.so` comes from the idTech4A++ app. |
| FreeType | FreeType licence **or** GPL-2.0-or-later, at your choice (section 3.4) | Linked dynamically; `libfreetype.so` comes from the idTech4A++ app. We use it under the GPL option. |
| libogg, libvorbis, libvorbisfile (Xiph.Org) | BSD-3-Clause (sections 3.5 and 3.6) | Linked dynamically; the `.so` files come from the idTech4A++ app. |

FTEQW's own source also contains a few other small pieces with their own headers. Check the file headers in the
FTEQW source if you need a complete audit.

### 1.3 Things used at runtime but not part of `libfteqw.so`

| Component | Licence | Notes |
|---|---|---|
| idTech4A++ app (glKarin, based on com.n0n3m4.diii4a) | GPL-3.0 | Installed separately by you. |
| OpenAL Soft | LGPL-2.0-or-later (with BSD-3-Clause parts) | Shipped inside the idTech4A++ app. FTEQW opens it at runtime with `dlopen("libopenal.so")`. Licence text: `LICENSE-LGPL-2.0-OpenAL-Soft.txt`. |
| NZ:P QuakeC (`nzp-team/quakec`) | GPL-2.0 | Source of `nzp-fte-qc.zip`. |
| NZ:P game assets (`nzp-team/assets`) | CC BY-SA 4.0, plus third-party items (below) | You download these yourself. They are **not** part of this repository. The assets pack carries its own `nzp/licenses/` folder. Licence text: `LICENSE-CC-BY-SA-4.0.txt`. |
| Termux, clang, CMake, Ninja, git | Various | Build tools only. Nothing from them is shipped. |

Third-party items inside the NZ:P assets pack (from its `nzp/licenses/attributions.txt`):

- Zander Noriega, "Dragged Through Hellfire Abomination": CC-BY 3.0
- Pro Sensory, "Halloween": CC0
- tricksntraps, "Ascend", "Flying Temple", "Glory Days", "Starting Over": CC0
- Hansjörg Malthaner, "Glass Bell Sounds": CC-BY-SA 3.0

## 2. Source code for the GPL parts

If you give someone a built `libfteqw.so` or `nzp-fte-qc.zip`, the GPL requires that they can get the matching source.
The complete corresponding source is:

| Binary | Source |
|---|---|
| `libfteqw.so` | idTech4A++ at tag `v1.1.0harmattan73` (commit `ce49510f`): <https://github.com/glKarin/com.n0n3m4.diii4a>, plus the three `nzp-fteqw-*.patch` files and the three `sed` edits and `config_nzp.h` from README Steps 3 and 5 of this repository |
| `nzp-fte-qc.zip` | NZ:P QuakeC at commit `6ed6305` (2026-09-03): <https://github.com/nzp-team/quakec>, built as described in the README section "Rebuild the QC yourself" |

## 3. Licence texts that must travel with binaries

### 3.1 bzip2 and libbzip2

```text

--------------------------------------------------------------------------

This program, "bzip2", the associated library "libbzip2", and all
documentation, are copyright (C) 1996-2019 Julian R Seward.  All
rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:

1. Redistributions of source code must retain the above copyright
   notice, this list of conditions and the following disclaimer.

2. The origin of this software must not be misrepresented; you must 
   not claim that you wrote the original software.  If you use this 
   software in a product, an acknowledgment in the product 
   documentation would be appreciated but is not required.

3. Altered source versions must be plainly marked as such, and must
   not be misrepresented as being the original software.

4. The name of the author may not be used to endorse or promote 
   products derived from this software without specific prior written 
   permission.

THIS SOFTWARE IS PROVIDED BY THE AUTHOR ``AS IS'' AND ANY EXPRESS
OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED.  IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY
DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE
GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

Julian Seward, jseward@acm.org
bzip2/libbzip2 version 1.0.8 of 13 July 2019

--------------------------------------------------------------------------
```

### 3.2 libjpeg (Independent JPEG Group)

From the `README` of the bundled libjpeg (release 9f of 14-Jan-2024):

```text
LEGAL ISSUES
============

In plain English:

1. We don't promise that this software works.  (But if you find any bugs,
   please let us know!)
2. You can use this software for whatever you want.  You don't have to pay us.
3. You may not pretend that you wrote this software.  If you use it in a
   program, you must acknowledge somewhere in your documentation that
   you've used the IJG code.

In legalese:

The authors make NO WARRANTY or representation, either express or implied,
with respect to this software, its quality, accuracy, merchantability, or
fitness for a particular purpose.  This software is provided "AS IS", and you,
its user, assume the entire risk as to its quality and accuracy.

This software is copyright (C) 1991-2024, Thomas G. Lane, Guido Vollbeding.
All Rights Reserved except as specified below.

Permission is hereby granted to use, copy, modify, and distribute this
software (or portions thereof) for any purpose, without fee, subject to these
conditions:
(1) If any part of the source code for this software is distributed, then this
README file must be included, with this copyright and no-warranty notice
unaltered; and any additions, deletions, or changes to the original files
must be clearly indicated in accompanying documentation.
(2) If only executable code is distributed, then the accompanying
documentation must state that "this software is based in part on the work of
the Independent JPEG Group".
(3) Permission for use of this software is granted only if the user accepts
full responsibility for any undesirable consequences; the authors accept
NO LIABILITY for damages of any kind.

These conditions apply to any software derived from or based on the IJG code,
not just to the unmodified library.  If you use our work, you ought to
acknowledge us.

Permission is NOT granted for the use of any IJG author's name or company name
in advertising or publicity relating to this software or products derived from
it.  This software may be referred to only as "the Independent JPEG Group's
software".

We specifically permit and encourage the use of this software as the basis of
commercial products, provided that all warranty or liability claims are
assumed by the product vendor.


The Unix configuration script "configure" was produced with GNU Autoconf.
It is copyright by the Free Software Foundation but is freely distributable.
The same holds for its supporting scripts (config.guess, config.sub,
ltmain.sh).  Another support script, install-sh, is copyright by X Consortium
but is also freely distributable.
```

### 3.3 libpng

```text
COPYRIGHT NOTICE, DISCLAIMER, and LICENSE
=========================================

PNG Reference Library License version 2
---------------------------------------

 * Copyright (c) 1995-2024 The PNG Reference Library Authors.
 * Copyright (c) 2018-2024 Cosmin Truta.
 * Copyright (c) 2000-2002, 2004, 2006-2018 Glenn Randers-Pehrson.
 * Copyright (c) 1996-1997 Andreas Dilger.
 * Copyright (c) 1995-1996 Guy Eric Schalnat, Group 42, Inc.

The software is supplied "as is", without warranty of any kind,
express or implied, including, without limitation, the warranties
of merchantability, fitness for a particular purpose, title, and
non-infringement.  In no event shall the Copyright owners, or
anyone distributing the software, be liable for any damages or
other liability, whether in contract, tort or otherwise, arising
from, out of, or in connection with the software, or the use or
other dealings in the software, even if advised of the possibility
of such damage.

Permission is hereby granted to use, copy, modify, and distribute
this software, or portions hereof, for any purpose, without fee,
subject to the following restrictions:

 1. The origin of this software must not be misrepresented; you
    must not claim that you wrote the original software.  If you
    use this software in a product, an acknowledgment in the product
    documentation would be appreciated, but is not required.

 2. Altered source versions must be plainly marked as such, and must
    not be misrepresented as being the original software.

 3. This Copyright notice may not be removed or altered from any
    source or altered source distribution.


PNG Reference Library License version 1 (for libpng 0.5 through 1.6.35)
-----------------------------------------------------------------------

libpng versions 1.0.7, July 1, 2000, through 1.6.35, July 15, 2018 are
Copyright (c) 2000-2002, 2004, 2006-2018 Glenn Randers-Pehrson, are
derived from libpng-1.0.6, and are distributed according to the same
disclaimer and license as libpng-1.0.6 with the following individuals
added to the list of Contributing Authors:

    Simon-Pierre Cadieux
    Eric S. Raymond
    Mans Rullgard
    Cosmin Truta
    Gilles Vollant
    James Yu
    Mandar Sahastrabuddhe
    Google Inc.
    Vadim Barkov

and with the following additions to the disclaimer:

    There is no warranty against interference with your enjoyment of
    the library or against infringement.  There is no warranty that our
    efforts or the library will fulfill any of your particular purposes
    or needs.  This library is provided with all faults, and the entire
    risk of satisfactory quality, performance, accuracy, and effort is
    with the user.

Some files in the "contrib" directory and some configure-generated
files that are distributed with libpng have other copyright owners, and
are released under other open source licenses.

libpng versions 0.97, January 1998, through 1.0.6, March 20, 2000, are
Copyright (c) 1998-2000 Glenn Randers-Pehrson, are derived from
libpng-0.96, and are distributed according to the same disclaimer and
license as libpng-0.96, with the following individuals added to the
list of Contributing Authors:

    Tom Lane
    Glenn Randers-Pehrson
    Willem van Schaik

libpng versions 0.89, June 1996, through 0.96, May 1997, are
Copyright (c) 1996-1997 Andreas Dilger, are derived from libpng-0.88,
and are distributed according to the same disclaimer and license as
libpng-0.88, with the following individuals added to the list of
Contributing Authors:

    John Bowler
    Kevin Bracey
    Sam Bushell
    Magnus Holmgren
    Greg Roelofs
    Tom Tanner

Some files in the "scripts" directory have other copyright owners,
but are released under this license.

libpng versions 0.5, May 1995, through 0.88, January 1996, are
Copyright (c) 1995-1996 Guy Eric Schalnat, Group 42, Inc.

For the purposes of this copyright and license, "Contributing Authors"
is defined as the following set of individuals:

    Andreas Dilger
    Dave Martindale
    Guy Eric Schalnat
    Paul Schmidt
    Tim Wegner

The PNG Reference Library is supplied "AS IS".  The Contributing
Authors and Group 42, Inc. disclaim all warranties, expressed or
implied, including, without limitation, the warranties of
merchantability and of fitness for any purpose.  The Contributing
Authors and Group 42, Inc. assume no liability for direct, indirect,
incidental, special, exemplary, or consequential damages, which may
result from the use of the PNG Reference Library, even if advised of
the possibility of such damage.

Permission is hereby granted to use, copy, modify, and distribute this
source code, or portions hereof, for any purpose, without fee, subject
to the following restrictions:

 1. The origin of this source code must not be misrepresented.

 2. Altered versions must be plainly marked as such and must not
    be misrepresented as being the original source.

 3. This Copyright notice may not be removed or altered from any
    source or altered source distribution.

The Contributing Authors and Group 42, Inc. specifically permit,
without fee, and encourage the use of this source code as a component
to supporting the PNG file format in commercial products.  If you use
this source code in a product, acknowledgment is not required but would
be appreciated.
```

### 3.4 FreeType

FreeType is offered under two licences and you choose one. For `libfteqw.so`, the GPL-2.0-or-later option applies
(see `LICENSE` in this repository). The notice from the bundled `LICENSE.TXT`:

```text
FREETYPE LICENSES
-----------------

The FreeType  2 font  engine is  copyrighted work  and cannot  be used
legally without  a software  license.  In order  to make  this project
usable to  a vast majority of  developers, we distribute it  under two
mutually exclusive open-source licenses.

This means that *you* must choose  *one* of the two licenses described
below, then obey all its terms and conditions when using FreeType 2 in
any of your projects or products.

  - The FreeType License,  found in the file  `docs/FTL.TXT`, which is
    similar to the  original BSD license *with*  an advertising clause
    that forces  you to explicitly  cite the FreeType project  in your
    product's  documentation.  All  details are  in the  license file.
    This license is suited to products which don't use the GNU General
    Public License.

    Note that  this license  is compatible to  the GNU  General Public
    License version 3, but not version 2.

  - The   GNU   General   Public   License   version   2,   found   in
    `docs/GPLv2.TXT`  (any  later  version  can  be  used  also),  for
    programs  which  already  use  the  GPL.  Note  that  the  FTL  is
    incompatible with GPLv2 due to its advertisement clause.

The contributed  BDF and PCF  drivers come  with a license  similar to
that  of the  X Window  System.   It is  compatible to  the above  two
licenses (see files `src/bdf/README`  and `src/pcf/README`).  The same
holds   for   the   source    code   files   `src/base/fthash.c`   and
`include/freetype/internal/fthash.h`; they were part of the BDF driver
in earlier FreeType versions.

The gzip  module uses the  zlib license (see  `src/gzip/zlib.h`) which
too is compatible to the above two licenses.

The files `src/autofit/ft-hb.c` and `src/autofit/ft-hb.h` contain code
taken almost  verbatim from the  HarfBuzz file `hb-ft.cc`,  which uses
the 'Old MIT' license, compatible to the above two licenses.

The  MD5 checksum  support  (only used  for  debugging in  development
builds) is in the public domain.


--- end of LICENSE.TXT ---
```

### 3.5 libogg

```text
Copyright (c) 2002, Xiph.org Foundation

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:

- Redistributions of source code must retain the above copyright
notice, this list of conditions and the following disclaimer.

- Redistributions in binary form must reproduce the above copyright
notice, this list of conditions and the following disclaimer in the
documentation and/or other materials provided with the distribution.

- Neither the name of the Xiph.org Foundation nor the names of its
contributors may be used to endorse or promote products derived from
this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
``AS IS'' AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
A PARTICULAR PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE FOUNDATION
OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```

### 3.6 libvorbis and libvorbisfile

```text
Copyright (c) 2002-2020 Xiph.org Foundation

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:

- Redistributions of source code must retain the above copyright
notice, this list of conditions and the following disclaimer.

- Redistributions in binary form must reproduce the above copyright
notice, this list of conditions and the following disclaimer in the
documentation and/or other materials provided with the distribution.

- Neither the name of the Xiph.org Foundation nor the names of its
contributors may be used to endorse or promote products derived from
this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
``AS IS'' AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
A PARTICULAR PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE FOUNDATION
OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```
