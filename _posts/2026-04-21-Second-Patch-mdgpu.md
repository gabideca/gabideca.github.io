---
title: "Second Patch in the Linux Kernel: Using 'guard' Instead of Manual lock+unlock"
date: 2026-04-21 23:55:01 -0300
categories: [Kernel, Patches]
tags: [kernel, linux, amdgpu, drm, patch, contribution]
---

Following the previous patch, here we disclose our second attempt at an accepted commit.
Once again, as you're probably used to by now, this was made in partnership with André Jun Hirata and Guilherme Santos Gabriel.

## The Subsystem
Once again, we checked the curated list and chose a new driver.
This time around, we went for the amdgpu driver located @ `drivers/gpu/drm/amd/pm/`.
The suggestion did not include the file path, but contained a hint: look for files using `mutex_lock` and `mutex_unlock` and replace them for the more modern `guard`.

## Locating the File
Naturally, the first step was to actually locate the target file we'd tackle.
To do so, we ran a grep command looking for the biggest ammount of `mutex_lock`s

```bash
grep -rn "mutex_lock" | awk -F: '{print $1}' | sort | uniq -c | sort -nr | awk '{print $2, $1}'
```

The first ranked file was `drivers/usb/gadget/function/uvc_configfs.c`, with 124 occurnces. However, it did not have a dedicated maintainer,
which made us pivot to `drivers/gpu/drm/amd/pm/amdgpu_dpm.c`, with 103 occurences.

## The Anatomy of the File
Looking through the script, three main patterns of mutex emerged.

The first one is a mutex call to protect a simple call before a functions' return:
```
@@ -46,10 +47,9 @@ int amdgpu_dpm_get_sclk(struct amdgpu_device *adev, bool low)
       if (!pp_funcs->get_sclk)
               return 0;

       mutex_lock(&adev->pm.mutex);
       ret = pp_funcs->get_sclk((adev)->powerplay.pp_handle,
                                low);
       mutex_unlock(&adev->pm.mutex);

       return ret;
}
```

The second is the `goto` pattern:
```
@@ -80,13 +79,12 @@ int amdgpu_dpm_set_powergating_by_smu(struct amdgpu_device *adev,
       enum ip_power_state pwr_state = gate ? POWER_STATE_OFF : POWER_STATE_ON;
       bool is_vcn = block_type == AMD_IP_BLOCK_TYPE_VCN;

       mutex_lock(&adev->pm.mutex);

       if (atomic_read(&adev->pm.pwr_state[block_type]) == pwr_state &&
                       (!is_vcn || adev->vcn.num_vcn_inst == 1)) {
               dev_dbg(adev->dev, "IP block%d already in the target %s state!",
                               block_type, gate ? "gate" : "ungate");
               goto out_unlock;
       }

       switch (block_type) {
@@ -115,9 +113,6 @@ int amdgpu_dpm_set_powergating_by_smu(struct amdgpu_device *adev,
       if (!ret)
               atomic_set(&adev->pm.pwr_state[block_type], pwr_state);

out_unlock:
       mutex_unlock(&adev->pm.mutex);

       return ret;
}
```

And the third is an unlock before the end of a function:
```
@@ -126,9 +121,9 @@ int amdgpu_dpm_set_gfx_power_up_by_imu(struct amdgpu_device *adev)
       struct smu_context *smu = adev->powerplay.pp_handle;
       int ret = -EOPNOTSUPP;

       mutex_lock(&adev->pm.mutex);
       ret = smu_set_gfx_power_up_by_imu(smu);
       mutex_unlock(&adev->pm.mutex);

       msleep(10);
```

The replacement for these patterns were straight forward, with a small caveat for the second one, 
requiring more attention as the logic was a bit more complex, 
and the third one, requiring `scoped_guard`

## Difficulties faced
A single function, `int amdgpu_dpm_force_performance_level`, gave us a hard time. 
It uses mutex in a way that does not apply to any of the identified patterns which brought a considerable complexity overhead.
In hopes of not breaking functionality, we opted for the conservative decision of not changing it at all.

```
...
else if ((current_level & profile_mode_mask) &&
		 !(level & profile_mode_mask))
		amdgpu_dpm_exit_umd_state(adev);

	mutex_lock(&adev->pm.mutex);

	if (pp_funcs->force_performance_level(adev->powerplay.pp_handle,
					      level)) {
		mutex_unlock(&adev->pm.mutex);
		/* If new level failed, retain the umd state as before */
		if (!(current_level & profile_mode_mask) &&
		    (level & profile_mode_mask))
			amdgpu_dpm_exit_umd_state(adev);
		else if ((current_level & profile_mode_mask) &&
			 !(level & profile_mode_mask))
			amdgpu_dpm_enter_umd_state(adev);

		return -EINVAL;
	}

	adev->pm.dpm.forced_level = level;

	mutex_unlock(&adev->pm.mutex);

	return 0;
}
```

## The Patch
The following is the final commit:

```
drm/amd/pm: Use guard(mutex) instead of manual lock+unlock

Use guard() and scoped_guard() for handling mutex lock instead of
manually locking and unlocking the mutex. This prevents forgotten
locks due to early exits and removes the need of gotos.

Signed-off-by: Andre Jun Hirata <andrejhirata@usp.br>
Co-developed-by: Gabriel Dimant <gabriel.dimant@usp.br>
Signed-off-by: Gabriel Dimant <gabriel.dimant@usp.br>
Co-developed-by: Guilherme Gabriel <guilhermesangabriel@usp.br>
Signed-off-by: Guilherme Gabriel <guilhermesangabriel@usp.br>

```

The patch was sent to the maintainer and the amdgpu mailing list.

- **Maintainers:** Kenneth Feng
- **List:** `amd-gfx@lists.freedesktop.org`

It can be found @ [lore.kernel](https://lore.kernel.org/amd-gfx/20260421015506.9230-1-andrejhirata@usp.br/T/#u)

## Result
As of the time of writing this post, we have not received an answer yet.
