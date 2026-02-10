# libaom-pwdc: AV1 Encoder with Photonic Wavelength Division Compression

A modified AV1 encoder (libaom) with the entropy coding stage instrumented for **Photonic Wavelength Division Compression (PWDC)** — a research project by LUXBIN.

## What This Is

The AV1 video codec encodes video in stages:

```
Raw video → Transform (DCT) → Quantize → Entropy code → Bitstream
```

This project modifies the **entropy coding** stage — the final stage that converts quantized coefficients into a compressed bitstream. The stock AV1 encoder uses an arithmetic range coder. We instrument it with PWDC wavelength channel statistics to validate a photonic alternative.

## How It Works

Every symbol encoded by AV1's entropy coder gets mapped to a **wavelength channel** (0-127), inspired by Wavelength Division Multiplexing in fiber optics:

- Symbol value + alphabet size → wavelength channel
- Channel hit counts are tracked per frame
- Boolean symbol distributions (0/1) are separately tracked
- Arithmetic coding output size is recorded for comparison

The goal: prove that PWDC's simpler symbol-table approach can achieve comparable compression to arithmetic coding, with faster decode (table lookup vs range normalization).

## Building

Requires: cmake, a C compiler (clang/gcc)

```bash
# Clone libaom from Google source
git clone --depth 1 https://aomedia.googlesource.com/aom libaom-build

# Copy PWDC entropy encoder over the original
cp entenc.c libaom-build/aom_dsp/entenc.c

# Build
mkdir libaom-build/build && cd libaom-build/build
cmake .. -DCMAKE_BUILD_TYPE=Release -DENABLE_TESTS=0 -DENABLE_EXAMPLES=1
make -j4 aomenc
```

## Files

| File | Description |
|------|-------------|
| `entenc.c` | PWDC-instrumented entropy encoder (drop-in replacement) |
| `entenc_original.c` | Original libaom range coder (for comparison) |
| `entenc.h` | Entropy encoder header (unchanged) |
| `entenc_original.h` | Original header backup |
| `bitwriter.h` | AV1 bitwriter wrapper (unchanged, shows API) |
| `entcode.h` | Common entropy coding definitions (unchanged) |

## Phase Roadmap

- **Phase 1** (current): Instrument range coder with PWDC statistics
- **Phase 2**: Add PWDC symbol-table encoding alongside arithmetic coding, compare output sizes
- **Phase 3**: Replace arithmetic coding with PWDC when within 1% efficiency

## Related

- [luxbin-photonic-codec](https://github.com/nichechristie/luxbin-photonic-codec) — Interactive web demo of PWDC compression
- [luxbin-lang](https://github.com/nichechristie/luxbin-lang) — LUXBIN Light Language with photonic encoding primitives

## License

Modified files: BSD 2-Clause (same as libaom)
Original libaom: Copyright (c) Alliance for Open Media
