# Comp releases

Welcome to the release page for **Comp**. Thank you for your interest.

> [!WARNING]
> **Comp is currently in a very early stage of development.**
> I do **not** recommend using it in production yet, as it may contain critical bugs and security vulnerabilities.

If you would like to try the project and help shape its future direction, I would be very grateful for your feedback.

I have been working on **Comp** for more than a year and a half. This release started as a simple personal side project, but after seeing some interest from the VFX community, I decided to package it into a more coherent form so that anyone interested can test it and share feedback.

Your feedback will help define the direction of future development, as well as the pace at which the project can move forward.

## Supported platforms

At the moment, **Comp** is available for Windows x64 and macOS arm64

A Linux build is also planned. Before starting the Linux port, I would like to reach a stable MVP on the current platforms and use that version as a baseline for parity testing.

The Windows build may work through Wine, but I have not tested this.

Development is currently done mostly on macOS. From time to time, I also use my old Windows PC to build and test the Windows version.

## Comp AI Worker Service

**Comp** is a standalone application that can also work as a client for a second component: **Comp AI Worker Service**.

The AI Worker Service is a separate command-line service that can process request queues over a local network. These requests are routed to neural network modules built into the service.

Currently, two modules are included:

- **Facebook/SAM3** — rough image segmentation
- **MatAnyone2** — mask edge refinement

These modules are C++ implementations of the corresponding inference pipelines, built using **libtorch**.

This means that the worker service does not require setting up the original full inference projects, installing Python, or managing a large number of dependencies. However, you still need to place the model weights and tokenizer files into the corresponding folders. Download links are listed below.

## CPU and GPU support

The AI Worker Service can run on both CPU and GPU. For obvious reasons, GPU processing is much faster.

Supported targets: CPU | CUDA | MPS

It is also possible to run the service with 16 GB of VRAM by using launch flags that limit the frame-history depth used by the neural networks.

I have not tested the service on 8 GB of VRAM.

In general, if a task does not fully fit into GPU memory, parts of it will be copied to CPU memory and processing will become very slow.

## Network usage

The AI Worker Service can serve multiple clients from different IP addresses. All requests are placed into a queue and returned to clients when they are ready.

However, network-based processing requires sending all frames for a task back and forth. These transfer costs alone may exceed the actual processing time.

The fastest setup is to run **Comp** and the AI Worker Service on the same machine. However, this requires enough RAM for both **Comp** cached playback and the AI Worker Service.

Background processing may also slow down compositing work running in parallel. **Comp** does not wait for the AI Worker Service to finish — you can continue editing your script while a task is being processed.

This split architecture, where AI functionality and compositing functionality are separated into two different applications, helps with stability, isolation, and collaborative workflows.

That said, built-in inference directly inside **Comp** would be noticeably faster and more convenient. On the other hand, it would likely be less stable and would increase workstation requirements. At the moment, it is still an open question which approach would be more beneficial for a typical **Comp** user.

## GPU rendering

At the moment, **Comp** does not use GPU acceleration for compositing operations. Only image drawing in the viewer is accelerated.

I have not yet chosen the backend for GPU compositing. The cross-platform **Qt RHI** looks like the simplest solution, but it would introduce Qt dependencies into the headless renderer.

One possible option would be to simply not support GPU acceleration in headless mode, but this is still open for discussion.

A more hardcore option would be to write three separate implementations for Metal, Vulkan and CUDA, this would provide speed, flexibility, and fewer limitations. However, before doing that, the current feature set needs to be stabilized so that rendering is predictable and can be used as a reference for comparison with the GPU implementation.

## Approximate roadmap for Comp

- Release the first alpha version.
- Collect feedback and fix bugs.
- Release the second alpha version.
- If no critical bugs are found, freeze this version as the reference for the Linux port.
- Hold a vote and choose a Linux distribution for development and testing.
- Release the second alpha version for Linux.
- Fix bugs and freeze that version as the reference.
- Develop and integrate GPU rendering identical to CPU rendering.
- Release the third alpha version.

## Approximate roadmap for AI Worker Service

- Finish implementing two new modules:
  - Corridor Crew keyer
  - DVD depth estimator

## Model weights

Place the files in the folders shown below.

### MatAnyone2 bundle

Folder:

```text
<worker-instalation-root>/family_bundles/mat2/models
```

Download:

```text
https://huggingface.co/kamaz01/MatAnyone2/blob/main/mat2_bundle.pt
```

### SAM3 tokenizer

Folder:

```text
<worker-instalation-root>/family_bundles/sam3/assets
```

Download:

```text
https://github.com/facebookresearch/sam3/blob/main/sam3/assets/bpe_simple_vocab_16e6.txt.gz
```

### SAM3 model

Folder:

```text
<worker-instalation-root>/family_bundles/sam3/models
```

Download:

```text
https://huggingface.co/facebook/sam3.1/blob/main/sam3.1_multiplex.pt
```

Before placing this file into the `models` folder, rename it to:

```text
sam3_bundle.pt
```

## Example worker launch command

Example command for launching the worker in `serve` mode with reduced VRAM usage:

```bash
<worker-instalation-path>\worker serve \
  --family-bundle mat2=<worker-instalation-path>/family_bundles/mat2 \
  --family-bundle sam3=<worker-instalation-path>/family_bundles/sam3 \
  --target cuda \
  --bind-host 127.0.0.1 \
  --port 19282 \
  --temp-dir <path-to-temp-dir>/worker_service \
  --cpu-threads 0 \
  --verbose \
  --max-propagation-history-depth 2 \
  --max-mem-frames 2
```

On macOS, use `--target mps` instead of `--target cuda`.

Replace all placeholder paths with paths that match your local installation.

## Windows security warning

If you run the application on Windows, you may see a warning that the app has been blocked from launching.

This happens because the application is currently unsigned. I do not have a code-signing certificate yet.

You can still launch the application by explicitly allowing it to run in the same warning dialog.

## Related projects, papers, and dependencies

- [MatAnyone2][matanyone2-github] ([project page][matanyone2-page])
- [Facebook/SAM3][sam3-github] ([Meta research page][sam3-meta], [SAM3.1 model weights][sam31-hf])
- [OpenCV][opencv-home] ([GitHub][opencv-github])
- [V-BM3D / VBM3D paper][vbm3d-paper]
- [VBM3D implementation paper][vbm3d-ipol]
- [JSON][json-org] ([ECMA-404][json-ecma404])
- [libtorch / PyTorch C++ API][libtorch-cpp]
- [C++17 / ISO/IEC 14882:2017][cpp17-iso] ([cppreference C++17 overview][cpp17-cppreference])
- [Qt RHI][qt-rhi]
- [Qt Advanced Docking System — ADS][qt-ads]
- [nlohmann/json][nlohmann-json]
- [ExprTk][exprtk]
- [Tcl/Tk][tcltk]
- [OpenEXR][openexr] / [Imath][imath]
- [OpenMP][openmp]

[matanyone2-github]: https://github.com/pq-yang/MatAnyone2
[matanyone2-page]: https://pq-yang.github.io/projects/MatAnyone2/
[sam3-github]: https://github.com/facebookresearch/sam3
[sam3-meta]: https://ai.meta.com/research/sam3/
[sam31-hf]: https://huggingface.co/facebook/sam3.1
[opencv-home]: https://opencv.org/
[opencv-github]: https://github.com/opencv/opencv
[vbm3d-paper]: https://www.eurasip.org/Proceedings/Eusipco/Eusipco2007/Papers/A1L-G03.pdf
[vbm3d-ipol]: https://arxiv.org/abs/2001.01802
[json-org]: https://json.org/
[json-ecma404]: https://ecma-international.org/publications-and-standards/standards/ecma-404/
[libtorch-cpp]: https://docs.pytorch.org/cppdocs/
[cpp17-iso]: https://www.iso.org/standard/68564.html
[cpp17-cppreference]: https://cppreference.com/cpp/17
[qt-rhi]: https://doc.qt.io/qt-6/topics-graphics.html
[qt-ads]: https://github.com/githubuser0xFFFF/Qt-Advanced-Docking-System
[nlohmann-json]: https://github.com/nlohmann/json
[exprtk]: https://www.partow.net/programming/exprtk/index.html
[tcltk]: https://www.tcl-lang.org/
[openexr]: https://github.com/AcademySoftwareFoundation/openexr
[imath]: https://github.com/AcademySoftwareFoundation/Imath
[openmp]: https://www.openmp.org/specifications/


## Downloads

Packaged testing releases are published here:

```text
https://github.com/ukmsz/Comp-releases/releases
```

Release packages include the application archive and SHA256 checksums.

## Community and Support

- LinkedIn: https://www.linkedin.com/in/yuri-radaev
- Buy Me a Coffee: https://buymeacoffee.com/yuri.r
- Discord: https://discord.gg/caJSDWwXX

