# Gaming

Ubuntu 26.04 provides a solid gaming environment through native Linux games, **Steam**, **Proton**, **Wine**, and **Vulkan**. Most modern Linux gaming relies on Vulkan and Mesa or NVIDIA's proprietary driver stack, while Proton provides compatibility for many Windows games.

## Steam

Install Steam from Ubuntu's repositories:

```bash
sudo apt install steam
```

Launch Steam and sign in.

For Windows games, enable Proton:

**Steam → Settings → Compatibility → Enable Steam Play for supported titles**

For broader compatibility, enable:

**Enable Steam Play for all other titles**

Select a Proton version when required. Start with the default Proton version and only change it when a specific game requires another version.

## Vulkan

Vulkan is the primary graphics API for many modern Linux games and is required by Proton for a large number of titles.

Verify Vulkan:

```bash
sudo apt install vulkan-tools
vulkaninfo --summary
```

If Vulkan reports the expected GPU and driver, the basic graphics stack is available.

> [!Important]
> Do not install random Vulkan drivers or packages without first identifying the GPU and active driver. Ubuntu's graphics stack already provides the appropriate packages for supported hardware.

## GameMode

GameMode can temporarily apply system optimizations while a game is running.

Install it:

```bash
sudo apt install gamemode
```

Verify:

```bash
gamemoded -t
```

For games that support launch options, use:

```text
gamemoderun %command%
```

GameMode is optional. It should not be treated as a replacement for correctly configured GPU drivers or graphics settings.

## Proton

Proton allows Windows games to run through Steam on Linux using Wine and additional compatibility components.

When a game does not work correctly:

1. Check its current Proton compatibility status.
2. Try the default Proton version.
3. Try another supported Proton version if necessary.
4. Check whether the problem is related to anti-cheat, launchers, codecs, or DRM.
5. Only then consider game-specific configuration.

Do not assume that a game failing under Proton indicates a broken GPU or Vulkan installation.

## Controllers

Most modern USB and Bluetooth controllers work through Linux's input stack and Steam Input.

For Steam games, Steam Input can provide controller mappings and compatibility for controllers that do not have native game support.

If a controller does not work, first verify that Ubuntu detects it before changing Steam or game configuration.

## Performance

Before changing system settings, establish a baseline.

Monitor:

```bash
mangohud %command%
```

Pay attention to:

- FPS and frame-time consistency
- GPU utilization
- CPU utilization
- GPU temperature
- VRAM usage
- RAM usage
- shader compilation stutter

A low FPS value does not automatically mean the GPU is the bottleneck. CPU saturation, shader compilation, VRAM exhaustion, thermal throttling, or an application-specific limitation can produce similar symptoms.

## Gaming Baseline

A minimal Ubuntu gaming setup can be:

```bash
sudo apt install steam gamemode vulkan-tools
```

Then verify Vulkan:

```bash
vulkaninfo --summary
```

Use Steam + Proton for Windows games, GameMode only when useful, and MangoHud for performance diagnostics.

> [!Warning]
> Avoid installing multiple competing driver stacks, manually replacing system graphics libraries, or applying undocumented kernel and launch parameters solely for gaming performance. Establish the actual bottleneck first, then make the smallest configuration change necessary.