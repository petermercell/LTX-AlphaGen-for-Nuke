# NOTICE: LTX_AlphaGen

LTX_AlphaGen (the Nuke node, and the `alphagen_edge` command-line tool built from the same source) is a **Derivative of LTX-2.x** as defined in the LTX-2.x Community License Agreement by Lightricks Ltd. It is distributed **exclusively under the terms of that Agreement**. The complete text, including Attachment A (Use Restrictions), is in the file `LICENSE` next to this one.

LTX-2.x, LTX-2.5 and the LTX-2.5 Alpha Gen IC-LoRA are by Lightricks Ltd. LTX_AlphaGen is not made, endorsed or supported by Lightricks. "LTX" is used only to say what the tool is built on.

## Notice to everyone who downloads or uses LTX_AlphaGen

These notices are required by sections 3.1, 3.5 and 4 of the Agreement.

1. **Your use is governed by the LTX-2.x Community License Agreement.** By downloading or using LTX_AlphaGen you agree to be bound by it. This includes:
   - the use restrictions in Section 4;
   - Attachment A, in their entirety;
   - Lightricks' Acceptable Use Policy at <https://static.lightricks.com/legal/ltx-acceptable-use-policy.pdf>.
2. **Companies with annual revenues of $10,000,000 or more** must obtain a paid commercial licence from Lightricks **before** any use of LTX_AlphaGen.
   - The revenue counts the whole group: subsidiaries, affiliates and companies under common control.
   - The only exception is use solely for a Non-Commercial Purpose as defined in Section 2.2 of the Agreement.
   - Contact Lightricks at licensing@ltx.io.
   - Do not download or use LTX_AlphaGen in such a company without that licence.
3. **The model weights are not included.** Each user downloads them from Lightricks on Hugging Face and accepts Lightricks' terms there.
4. You remain responsible for your use of the Outputs. This includes Attachment A item 5 (disclosing machine-generated content) and any AI regulations that apply to you (Section 6 of the Agreement).
5. If you redistribute LTX_AlphaGen, you must:
   - include `LICENSE` and this `NOTICE.md`;
   - pass on these notices;
   - follow Section 3 of the Agreement.

## What was changed (Section 3.3)

Copyright (c) 2026 Peter Mercell (petermercell.com) for these modifications. They were written by Peter Mercell with help of Claude (Anthropic). They are licensed under the LTX-2.x Community License Agreement (Section 3.6).

- **C++/libtorch reimplementation** of these LTX-2.x components, based on Lightricks' Python code in <https://github.com/Lightricks/LTX-2>:
  - the video transformer: `src/ltx_transformer.*`, from `ltx_core.model.transformer`;
  - the convolutional video VAE: `src/vae.*`, from `ltx_core.model.video_vae`;
  - the IC-LoRA sampling pipeline: `src/pipeline.*`, from `ltx_pipelines.ic_lora`.
- **Changed to run in limited GPU memory:**
  - transformer blocks are streamed from pinned host RAM;
  - the reference video is encoded in causal time slices;
  - decoding is tiled;
  - long shots run in overlapping temporal windows;
  - an out-of-memory fallback retries with smaller windows and tiles.
- **Prompt embedding built in.** The Alpha Gen prompt embedding (`alphagen_prompt.safetensors`) is built into the plugin binary. It is an Output of the LTX-2.5 text encoder for the Alpha Gen prompt.
- **Additions by Peter Mercell:**
  - the Nuke node and its disk cache;
  - the edge-band gate around a mask;

No safety, disclosure, watermarking or provenance feature of LTX-2.x has been removed or disabled.

## Third-party code

**nlohmann/json** (`third_party/nlohmann/json.hpp`, compiled into the binaries), MIT License:

```
MIT License

Copyright (c) 2013-2023 Niels Lohmann <https://nlohmann.me>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

The Nuke node runs on the libtorch, CUDA and Qt libraries that ship with Nuke. None of them are included here.

## No warranty

LTX_AlphaGen is provided "as is", without warranty of any kind, in line with Sections 9 and 10 of the Agreement.
