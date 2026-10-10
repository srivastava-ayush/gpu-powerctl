# Changelog

## 0.1.0 — first public preview

- Add manual `status`, `off`, and `on` commands.
- Detect a single NVIDIA PCI display controller instead of hardcoding the author's BDF.
- Detect an NVIDIA audio-class sibling in the same PCI slot.
- Refuse ambiguous multi-NVIDIA systems.
- Check common NVIDIA device nodes with `fuser` before unbinding.
- Report runtime power state and warn that suspension is not proof of zero physical power draw.
