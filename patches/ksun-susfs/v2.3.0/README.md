# SUSFS v2.3.0 fix-up for KernelSU-Next `c61d876`

`10_enable_susfs_for_ksu.patch` from the SUSFS v2.3.0 GKI tips does not apply cleanly to KernelSU-Next `dev` at `c61d876480976e553060789759cc4b54c9e7d816` (version 33312). `WildKernels/kernel_patches` has no `next/susfs_fix_patches/v2.3.0` directory, and the v2.2.0 fix patches fail on this commit.

`after-enable.patch` is applied by `.github/actions/build-kernel` after that enable patch (failed hunks ignored) when `SUSFS_VERSION` is `v2.3.0`. It was generated against that partial apply and round-tripped back onto a clean `c61d876` tree.

It does three things the stock enable patch could not finish:

- Drops the kprobe/syscall-hook objects and finishes the manual-hook init, setuid, sucompat, and umount paths.
- Replaces the reboot kprobe with `ksu_supercall_reboot_handler`.
- Keeps Wild KSU toolkit reboot commands (`CHANGE_MANAGER_UID`, `GET_SULOG_DUMP_V2`, `CHANGE_KSUVER`, `CHANGE_SPOOF_UNAME`) and reports hook mode as `Inline (SuSFS)`.

Regenerate this file if either pin in `versions/ksun-susfs.json` moves. Do not reuse it on a different KernelSU-Next commit.
