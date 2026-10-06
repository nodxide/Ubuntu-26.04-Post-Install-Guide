# Graphics

Ubuntu 26.04 Desktop uses **Wayland by default**, while X11 applications remain supported through **XWayland**.

The graphics stack consists of several independent layers: GPU hardware, kernel drivers, graphics APIs, video acceleration, and application-level acceleration. Verify each layer independently when troubleshooting.

## Identify the GPU

Identify the installed graphics hardware:

```bash
lspci | grep -Ei 'vga|3d|display'
```

For detailed information, including the active kernel driver:

```bash
lspci -nnk | grep -A3 -Ei 'vga|3d|display'
```

Pay attention to the `Kernel driver in use` field.

## OpenGL

Install the diagnostic utilities:

```bash
sudo apt install mesa-utils
```

Check the active OpenGL renderer:

```bash
glxinfo -B
```

Inspect:

```text
OpenGL vendor string
OpenGL renderer string
OpenGL version string
```

The renderer is particularly important. A software renderer may indicate that hardware acceleration is not being used.

## Vulkan

Install Vulkan diagnostics:

```bash
sudo apt install vulkan-tools
```

Verify Vulkan:

```bash
vulkaninfo --summary
```

Inspect the reported GPU and driver information rather than only checking whether the command succeeds.

> [!Important]
> OpenGL and Vulkan are separate graphics APIs. Working OpenGL does not guarantee that Vulkan is configured correctly.

## VA-API

VA-API provides hardware-accelerated video decoding and encoding.

Install the diagnostic utility:

```bash
sudo apt install vainfo
```

Check available profiles:

```bash
vainfo
```

Actual support depends on the GPU, driver, codec, application, and operation. A GPU supporting VA-API does not necessarily provide hardware acceleration for every codec.

## NVIDIA

For NVIDIA systems, verify the driver and GPU state:

```bash
nvidia-smi
```

Then independently verify OpenGL:

```bash
glxinfo -B
```

And Vulkan:

```bash
vulkaninfo --summary
```

A successful `nvidia-smi` result does not prove that the desktop or a particular application is rendering through the NVIDIA GPU.

For hybrid graphics systems, also verify which GPU is being used by the application.

## Wayland

Check the current session:

```bash
echo "$XDG_SESSION_TYPE"
```

A Wayland session should report:

```text
wayland
```

Check the Wayland display:

```bash
echo "$WAYLAND_DISPLAY"
```

For XWayland applications:

```bash
echo "$DISPLAY"
```

A Wayland desktop can therefore run both native Wayland applications and X11 applications through XWayland.

## Graphics Stack

The graphics stack can be viewed as:

```text
GPU
 ↓
Kernel driver
 ↓
Graphics stack
 ├── OpenGL
 ├── Vulkan
 └── VA-API
      ↓
Application
```

Do not conflate:

- GPU driver installation
- OpenGL rendering
- Vulkan support
- hardware video decoding
- hardware video encoding
- application-specific GPU acceleration

For example, a browser can use GPU-accelerated rendering while a video player still performs software decoding.

## Diagnostics

When investigating a graphics problem, collect the relevant information first:

```bash
lspci -nnk | grep -A3 -Ei 'vga|3d|display'
glxinfo -B
vulkaninfo --summary
echo "$XDG_SESSION_TYPE"
echo "$WAYLAND_DISPLAY"
echo "$DISPLAY"
```

For NVIDIA:

```bash
nvidia-smi
```

For video acceleration:

```bash
vainfo
```

Record the results before changing drivers or configuration. This makes it possible to determine which layer is actually responsible for the problem.

## Driver Changes

Avoid repeatedly installing, removing, or switching GPU drivers without a specific reason.

Before changing the graphics stack, identify:

- GPU model
- active kernel driver
- OpenGL renderer
- Vulkan support
- session type
- whether the problem affects all applications or only one

> [!Important]
> Diagnose graphics problems layer by layer. Do not replace a working driver simply because an application reports a rendering problem. The issue may instead be related to the graphics API, compositor, XWayland, video acceleration, or the application itself.

## X11 Compatibility

Do not switch the entire desktop session to X11 unless a specific application or workflow has a demonstrated Wayland compatibility problem.

First determine whether the affected application is:

- native Wayland;
- running through XWayland;
- using OpenGL or Vulkan;
- relying on hardware video acceleration.

Prefer resolving application-specific compatibility issues rather than replacing the entire desktop graphics session.