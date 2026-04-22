---
title: "First Patch to the Linux kernel: Deduplicating code in the amdgpu driver"
date: 2026-04-18 12:45:00 -0300
categories: [Kernel, Patches]
tags: [kernel, linux, amdgpu, drm, patch, contribution]
---

Ladies and gentlemen, welcome to the main course. Everything you've read so far was merely an appetizer for whats about to come.
After all that setup and acclimation, it is finally time for the first patch to the one and only Linux kernel.

Once again, the material you're about to read was developed by André Jun Hirata, me, Gabriel Dimant and Guilherme Santos Gabriel.

## The chosen subsystem
The teaching staff prepared a relation of possible issues we could write pathces from.

We chose a code duplication issue in the amdgpu subsystem located @ `drivers/gpu/drm/amd/amdgpu/`.

Realize that at no point the word "tutorial" showed up. There is no tutorial this time aroumd, this is the real world. We're about to do something no one has ever done before.

## The Issue
Opening up the proposed files, `gmc_v10_0.c` and `gmc_v11_0.c`, we noticed that two functions were identical:

* `gmc_v10_0_get_vm_pde` / `gmc_v11_0_get_vm_pde`
* `gmc_v10_0_get_vm_pte` / `gmc_v11_0_get_vm_pte`

The main hypothesis is that v_11 spawned from a copy of v_10 and these functions were never adapted.

## Solution Attempt 1 - Make v_10 a generic
The first and naive approach we made was to simply make it so that the v11 functions turned into calls to their v_10 counterparts

## Why Soultion 1 is a Non-solution
While doing the aforementioned would solve the specific code duplication issue, it would introduce a false dependency:
were someone to alter the v_10 version of the functions, who are adapted to v_10 hardware, v_11 would silently fail due to a false v_10 hardware dependency

## Solution 2 - Using the Actual Helper File
This is a more refined, theoretically correct and difficult approach.
The directory has a dedicated `amdgpu_gmc.c` helper file for GMC scripts, which conveniently fit the issue we had.
We moved the implementations of `amdgpu_gmc_nv_get_vm_pde` and `amdgpu_gmc_nv_get_vm_pte` to the file, with these respective names,
and made it so that v_10 and v_11 called them. No dependencies were added.

## Difficulties encountered
1. Naming conflicts: the header file `amdgpu_gmc.h` already had the "generic" function signatures for `amdgpu_gmc_get_vm_pde` and `amdgpu_gmc_get_vm_pte`,
which resulted in a failed compilation. The solution was to add the prefix `nv` to the helper file functions indicating that they're related to the NV10/NV11 hardware.

2. Missing includes: The `MTYPE_NC`, `MTYPE_WC`, `MTYPE_CC` and `MTYPE_UC` constants existed in the v_10 file, but not in the helper file.
The solution was to simply add them.

## The Patch
Having these issues solved, we could finally send our beautifull commit:

```
drm/amdgpu: unify gmc v10 and v11 get_vm_pde and get_vm_pte into common helpers

gmc_v10_0_get_vm_pde, gmc_v10_0_get_vm_pte and their v11 counterparts
are identical. Move the shared implementation to amdgpu_gmc.c as
amdgpu_gmc_nv_get_vm_pde and amdgpu_gmc_nv_get_vm_pte, and update both
gmc_v10_0 and gmc_v11_0 to use the common helpers.

No functional changes intended. BUG_ON preserved from original
gmc_v10_0 and gmc_v11_0 implementations.

Signed-off-by: Andre Jun Hirata <andrejhirata@usp.br>
Co-developed-by: Gabriel Dimant <gabriel.dimant@usp.br>
Signed-off-by: Gabriel Dimant <gabriel.dimant@usp.br>
Co-developed-by: Guilherme Gabriel <guilhermesangabriel@usp.br>
Signed-off-by: Guilherme Gabriel <guilhermesangabriel@usp.br>
```

The patch was sent to the maintainers and the mailing list of the amdgpu subsystem:

- **Maintainers:** Alex Deucher e Christian König
- **List:** `amd-gfx@lists.freedesktop.org`

It can be found @ [lore.kernel](https://lore.kernel.org/amd-gfx/20260418201545.20673-1-andrejhirata@usp.br/)

## Result
Unfortunately, the patch was rejected by Christian König with the following message:

```
Well again this is not something we want to do.
Those functions are intentionally separated.

Regards,
Christian.
```

What we take from this is that not every code duplication is an issue. In this case, it was intended.
This does bring forward the question of how does one know if a duplication is a product of an accident or intentional. We remain oblivious.
