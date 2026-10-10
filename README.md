# gpu-powerctl 
made by Ayush (goat)

**Manual NVIDIA discrete-GPU power control for Linux laptops.** `gpu-powerctl` provides `status`, `off`, and `on` commands. The first release is conservative: it supports one detected NVIDIA PCI display controller and refuses to guess if it finds more than one.

> **Experimental, hardware-dependent, root-level operations.** This project can detach the NVIDIA GPU from its driver. When detached, CUDA, gaming, external displays routed through the dGPU, and other NVIDIA-dependent workloads will not work until the device is rebound. Test on a machine where you can recover from a failed rebind. Keep work saved and do not run `off` while using the GPU.

## Why

On the author's(AYUSHHH THE GOAT) ASUS TUF Gaming A15 (FA506ICB; Ryzen 7 4800H; NVIDIA RTX 3050 Mobile; Arch Linux; Hyprland; `nvidia-open`), the documented sequence was:

1. Unbind NVIDIA audio function `0000:01:00.1` (if present).
2. Unbind GPU function `0000:01:00.0` from the NVIDIA driver.
3. Set each function's `power/control` to `auto`.
4. Verify `power/runtime_status` reports `suspended`.

The author reported that unbinding alone left the GPU active, while unbinding plus runtime PM `auto` reached `suspended` on this system. This is a single-machine result, not a guarantee for other laptops.

## Requirements

- Linux with PCI sysfs and runtime power-management interfaces.
- Bash, `fuser` (Arch package: `psmisc`), and standard core utilities.
- A loaded NVIDIA driver for normal `off`/`on` operation.
- Root privileges for `off` and `on`.
- An NVIDIA PCI display controller; the script currently handles exactly one.

## Install from a checkout

```bash
sudo install -Dm755 bin/gpu-powerctl /usr/bin/gpu-powerctl
```

For a temporary test before installing system-wide:

```bash
./bin/gpu-powerctl status
sudo ./bin/gpu-powerctl off
./bin/gpu-powerctl status
sudo ./bin/gpu-powerctl on
./bin/gpu-powerctl status
```

## Usage

```bash
gpu-powerctl status
sudo gpu-powerctl off
sudo gpu-powerctl on
gpu-powerctl status
```

`status` shows driver binding, `power/control`, and `power/runtime_status`. `off` checks known NVIDIA device nodes with `fuser`, unbinds the detected NVIDIA audio sibling (if present) and GPU, sets runtime power management to `auto`, and waits up to 10 seconds for the GPU to report `suspended`. `on` sets power control to `on` and asks the kernel to probe the GPU and audio functions for their normal drivers.

No passwordless sudoers rule is installed. Do not add a broad `NOPASSWD` rule for a script you are actively developing. If you choose to configure one yourself, understand that a root-owned, non-user-writable executable and strict argument handling are essential.

## Safety and limitations

- The script refuses to act when it detects multiple NVIDIA PCI display controllers. Multi-GPU selection/configuration is not implemented yet.
- The audio sibling is only considered when it is an NVIDIA PCI audio-class function in the same PCI slot. Other functions, such as USB-C controllers, are intentionally left alone.
- The `fuser` check is a best-effort guard, not a complete guarantee that no GPU client exists. It can miss workloads that don't hold the usual device nodes or race with new processes. Close GPU applications before running `off`.
- Driver rebinding may fail because of driver, firmware, kernel, or device-specific behavior. Inspect `journalctl -k -b` if it does.
- `runtime_status=suspended` indicates the kernel reports runtime suspension; it does **not** prove the physical GPU consumes zero watts or quantify battery savings.
- The script does not install udev rules, run a daemon, switch automatically on AC/battery, or configure sudoers.
- It does not promise compatibility with every NVIDIA laptop or every NVIDIA driver variant. Initial validation is limited to the author's reported setup.

## Recovery

If `on` fails, save the output and inspect:

```bash
lspci -nnk
journalctl -k -b --no-pager | tail -200
```

A reboot may restore the normal driver binding. Do not add automatic startup actions until the manual path is tested repeatedly on the target machine.

## Development checks

```bash
bash -n bin/gpu-powerctl
shellcheck bin/gpu-powerctl
```

These checks validate shell syntax/static patterns; they do not test real PCI power behavior. Do not test `off` on your only working GPU or during a workload that depends on it.

## Roadmap

- Test matrix across laptop models, kernels, and NVIDIA driver variants.
- Explicit device selection for systems with multiple NVIDIA GPUs.
- More complete active-client detection and rollback tests.
- Arch package and AUR release after the upstream GitHub URL and checksums are finalized.

## License

MIT; see [LICENSE](LICENSE).
