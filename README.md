# LTX_AlphaGen 1.0 for Nuke 17

LTX_AlphaGen is a Nuke node that makes alpha mattes with LTX-2.5 Alpha Gen. It works on the edge band of a SAM mask: the SAM mask gives the inside and the outside, and Alpha Gen gives the edge (hair, motion blur, soft and semi-transparent detail).

It runs on Nuke's own libtorch, so nothing else needs to be installed.

Author: Peter Mercell ([petermercell.com](https://petermercell.com)), with help of Claude (Anthropic). LTX-2.5 and the Alpha Gen IC-LoRA are by Lightricks.

**Licence:** LTX-2.x Community License Agreement (see `LICENSE` and the Licence section at the end). Companies with $10M or more in annual revenue need a paid licence from Lightricks before use.

## 1. What you need

| | |
|---|---|
| OS | Linux with glibc 2.34 or newer: Rocky / Alma / RHEL 9 |
| Nuke | 17.0 or 17.1 (both builds are in this folder) |
| GPU | NVIDIA, 24 GB recommended. A run at 1920x1088 peaks at about 12 GB, so a 16 GB card works at 2K and below. |
| Driver | NVIDIA 570 or newer (Nuke 17's libtorch uses CUDA 12.8). The CUDA toolkit is **not** needed. |
| RAM | 64 GB recommended. While the models are loaded, 24 GiB of RAM is pinned for the transformer. |
| Disk | Space for the three LTX model files, plus the cache: one half-float EXR per frame. |

## 2. Download the models (three files from Lightricks)

The prompt embedding is built into the plugin. Only these three files are needed:

| file | from |
|---|---|
| `ltx-2.5-22b-distilled-transformer-bf16.safetensors` | [Lightricks/LTX-2.5](https://huggingface.co/Lightricks/LTX-2.5), folder `diffusion_models/` |
| `ltx-2.5-video-vae-conv-bf16.safetensors` | [Lightricks/LTX-2.5](https://huggingface.co/Lightricks/LTX-2.5), folder `vae/` |
| `ltx-2.5-22b-ic-lora-alpha-gen-0.9.safetensors` | [Lightricks/LTX-2.5-22b-IC-LoRA-Alpha-Gen](https://huggingface.co/Lightricks/LTX-2.5-22b-IC-LoRA-Alpha-Gen) (gated: log in to Hugging Face and accept the terms first) |

- Take the **conv** VAE (`...-video-vae-conv-bf16`), not `ltx-2.5-video-vae-bf16`.
- Take the **distilled bf16** transformer, not the dev, int8 or nvfp4 versions.
- Put the files in one folder. Subfolders are fine, because the folder is searched recursively.
- The models are under the LTX-2.x community licence. Each user downloads them and accepts the licence on Hugging Face.

With the Hugging Face CLI (after `huggingface-cli login` and accepting the LoRA's terms on its page):

```bash
M=/path/to/ltx_models
huggingface-cli download Lightricks/LTX-2.5 diffusion_models/ltx-2.5-22b-distilled-transformer-bf16.safetensors --local-dir $M
huggingface-cli download Lightricks/LTX-2.5 vae/ltx-2.5-video-vae-conv-bf16.safetensors --local-dir $M
huggingface-cli download Lightricks/LTX-2.5-22b-IC-LoRA-Alpha-Gen ltx-2.5-22b-ic-lora-alpha-gen-0.9.safetensors --local-dir $M
```

## 3. Install

1. Copy the `LTX_AlphaGen` folder anywhere, for example `~/.nuke/LTX_AlphaGen` or a shared tools disk.
2. Check the PC and the models:
   ```bash
   /path/to/LTX_AlphaGen/check.sh /path/to/ltx_models
   ```
   It checks the driver, the GPU memory, the RAM, glibc and the three model files, and ends with `ready`.
3. Make Nuke load the plugin, in one of two ways:
   - add this to `~/.nuke/init.py`:
     ```python
     import nuke
     nuke.pluginAddPath('/path/to/LTX_AlphaGen')
     ```
   - or set `NUKE_PATH=/path/to/LTX_AlphaGen` (separate several paths with `:`).

   `init.py` in the folder picks the 17.0 or 17.1 build to match the running Nuke.
4. Start Nuke. The node is in **Nodes > AI > LTX_AlphaGen**.

## 4. Use

1. Connect the plate to `plate`, and the SAM mask to `mask`. The mask is optional; without it you get the plain Alpha Gen matte.
2. Set `mask channel` (inside means value >= 0.5) and `plate colourspace` (linear Rec.709, ACEScg or sRGB).
3. Set `models folder` to the folder with the three files, and give each node its own `cache folder`.
4. Press **Process**. A progress window shows the frames being read, the models loading and each denoise step. **Cancel** stops the run within a few seconds.
5. The node then shows the plate with the matte in alpha.
   - `band in`, `band out` and `feather` are applied live from the mask input.
   - Changing them needs no new Process.

Shot length and size:
- Alpha Gen is made for up to about 145 frames at up to 1920x1088.
- Longer shots are processed in overlapping windows.
- With `working scale` set to auto, larger plates are scaled down to a 2K long side. 4K plates are scaled by 0.5.

### Knobs

| knob | |
|---|---|
| models folder | the folder with the three LTX files |
| cache folder | where the matte EXRs and `alphagen.json` go; use one folder per node |
| use input range / frame range | the frames to process |
| working scale | auto = full size up to a 2K long side, else 0.75 / 0.5 / 0.25; or a fixed scale |
| plate colourspace | linear / ACEScg / sRGB |
| mask channel | channel of the `mask` input |
| band in / band out / feather | the edge band around the mask edge, in plate pixels (default 8 / 48 / 4); live |
| alpha | edge band gate (live) / Alpha Gen matte (no gate) / as processed |
| VRAM limit (GB) | the GPU memory Process plans for (default 20, never more than is free) |
| GPU | CUDA device number |
| keep models loaded | off: RAM and VRAM are freed when Process ends. On: the next Process starts at once. |
| Process / Release models | run, or free the models now |
| output | plate + matte in alpha / premultiplied plate / matte only |
| warn when the plate changed | compares the plate with the one that was processed |
| Advanced | fps (0 = project fps), seed, LoRA strength, window frames, window overlap, encode slice, decode tile |

**If a matte is wrong on some frames**, change `seed` on the Advanced tab and process again. The defaults are good for most shots.

### Cache folder

- `alphagen.####.exr` files are half float, in the plate format.
  - Channel `A` is the gated matte.
  - Channel `raw.A` is the ungated Alpha Gen matte.
  - A Read node can read them.
- `alphagen.json` is written last.
  - A cancelled or failed Process leaves no `alphagen.json`, and the node then shows "not processed".

The node shows the plate with a warning in these cases:
- the frame is outside the processed range
- the plate or its format changed
- Nuke is in proxy mode

## 5. Troubleshooting

| problem | fix |
|---|---|
| Node not in the menu | Check that `nuke.pluginAddPath` points at the `LTX_AlphaGen` folder (the one with `init.py`). The Script Editor shows `LTX_AlphaGen: no build for Nuke ...` if the Nuke version is neither 17.0 nor 17.1. |
| `GLIBC_2.34 not found` | The OS is too old; use Rocky / RHEL 9. |
| "libtorch reports no CUDA device" | Update the NVIDIA driver to 570 or newer; check `nvidia-smi`. |
| "cannot find ... under ..." | Run `check.sh` on the models folder; check the exact file names in section 2. |
| Out of GPU memory | Lower `VRAM limit`, or lower `working scale`, or close other GPU work. Process already retries with smaller windows and decode tiles by itself. |
| Out of RAM / the machine swaps | Turn `keep models loaded` off and close other heavy apps; 24 GiB is pinned during Process. |
| Slow first Process | The models are read from disk. A fast local SSD helps; with `keep models loaded` on, later runs skip this. |

## Licence

LTX_AlphaGen is a Derivative of LTX-2.x and is distributed **exclusively under the LTX-2.x Community License Agreement** by Lightricks Ltd. The full text is in `LICENSE`. `NOTICE.md` lists what was changed and the third-party code (nlohmann/json, MIT).

By downloading or using LTX_AlphaGen you agree to that Agreement. This includes its use restrictions (Attachment A) and Lightricks' [Acceptable Use Policy](https://static.lightricks.com/legal/ltx-acceptable-use-policy.pdf).

- **Companies with annual revenues of $10M or more** (the whole group counts) need a paid commercial licence from Lightricks **before** using LTX_AlphaGen. The only exception is purely non-commercial testing or evaluation as defined in Section 2.2. Contact: licensing@ltx.io.
- The model weights are not included. Each user downloads them and accepts Lightricks' terms on Hugging Face.
- LTX_AlphaGen is free, provided as is, without warranty. It is not made, endorsed or supported by Lightricks.
