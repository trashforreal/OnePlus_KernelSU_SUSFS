# SUSFS v2.3.0 fix-up for KernelSU-Next `c61d876`

`10_enable_susfs_for_ksu.patch` from the SUSFS v2.3.0 GKI tips does not apply cleanly to KernelSU-Next `dev` at `c61d876480976e553060789759cc4b54c9e7d816` (version 33312). `WildKernels/kernel_patches` has no `next/susfs_fix_patches/v2.3.0` directory, and the v2.2.0 fix patches fail on this commit.

`after-enable.patch` is applied by `.github/actions/build-kernel` after that enable patch (failed hunks ignored) when `SUSFS_VERSION` is `v2.3.0`. It was generated against that partial apply and round-tripped back onto a clean `c61d876` tree.

It does three things the stock enable patch could not finish:

- Drops the kprobe/syscall-hook objects (`hook/lsm_hook`, `syscall_*`, `arm64/syscall_hook`, and the other arch syscall hooks) and finishes the manual-hook init, setuid, sucompat, and umount paths. `infra/symbol_resolver.o` stays in `kernel/Kbuild`.
- Replaces the reboot kprobe with `ksu_supercall_reboot_handler`.
- Keeps Wild KSU toolkit reboot commands (`CHANGE_MANAGER_UID`, `GET_SULOG_DUMP_V2`, `CHANGE_KSUVER`, `CHANGE_SPOOF_UNAME`) and reports hook mode as `Inline (SuSFS)`.

`kernel/core/init.c` still includes `infra/symbol_resolver.h` and calls `ksu_init_symbol_resolver()` before `susfs_init()` and the other init helpers. `kernel/selinux/selinux.c` resolves `security_secctx_to_secid`, `security_secid_to_secctx`, and `security_release_secctx` with `find_kernel_symbol_exact()` when `CONFIG_ANDROID` is unset. These OnePlus GKI trees compile that path, so the final vmlinux link needs `symbol_resolver.o`. The stock enable patch tries to drop the object, but that hunk does not apply on this commit; this fix-up must not drop it either. Do not paper over the link with `#define CONFIG_ANDROID`: that branch of `selinux.c` does not define `ksu_security_secctx_to_secid`.

Regenerate this file if either pin in `versions/ksun-susfs.json` moves. Do not reuse it on a different KernelSU-Next commit.
