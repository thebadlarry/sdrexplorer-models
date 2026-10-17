# sdrexplorer-models

Trained models for the sdrexplorer signal classifier (`sdrexplorer-classifier`): small 1-D
convolutional networks that label a signal's modulation or protocol from its raw I/Q samples.

This repository holds the **description** of each model release: the manifest, each model's metadata and
accuracy, and these model cards. The model files themselves are the **files of the GitHub release** with the
same tag (`models-1.0.0`, …), so the repository's history stays small however many times the models are
retrained.

You don't normally need to touch this repository: sdrexplorer's `install.sh` installs the current release
along with the classifier. See [Installing](#installing) for the other ways.

## License

The models (the weights in the release files, and the metadata here) are licensed under the
[Creative Commons Attribution 4.0 International license](LICENSE) (CC BY 4.0). You may use, share and
adapt them for any purpose, including commercially, as long as you give credit, for example:

> Signal classifier models from sdrexplorer-models (models-1.0.0), CC BY 4.0.

The code that trains and runs them is GPL-3.0-or-later, in the sdrexplorer repositories.

## The models

| Profile | Input | Classes | Used for |
|---|---|---|---|
| `hf` | 9.6 kHz × 4096 samples (427 ms) | 19: noise, AM, NBFM, USB, LSB, CW, FSK2, FSK4, BPSK, QPSK, 8PSK, 16QAM, 64QAM, S4285-like, S4529-like, PSK31, FT8, AX25-300, MSK144 | HF signals |
| `vu` | 24 kHz × 4096 (171 ms) | 13: noise, NBFM, P25-P1, DMR, NXDN48, NXDN96, AX25-1200, AX25-9600, POCSAG, carrier, other, FLEX, SmartNet | V/UHF signals up to ~14 kHz wide |
| `vu48` | 48 kHz × 4096 (85 ms) | 15: the 13 of `vu`, then EDACS (control channel) and EDACS-V (analog working channel) | V/UHF signals 14–40 kHz wide (wide pagers, wide FM, EDACS) |

[MODELS.md](MODELS.md) has each release's accuracy, overall and per class.

All three are the same architecture, `ResAMC`: a 1-D residual network (a stem convolution, 11 residual
blocks of 1×7 convolutions with batch normalization, global average pooling and two dense layers; about
1.8 M parameters, 7 MB as ONNX). It was designed for this project so that every layer runs natively on
small NPUs; the definition is `sdrexplorer_train/model.py` in sdrexplorer-train.

**Input:** one window of complex baseband, the signal centered, as a float32 tensor `[1, 2, 1, 4096]`
(channel 0 = I, channel 1 = Q), normalized to unit RMS. **Output:** `logits`, one per class, in the order of
`classes` in the manifest. The classifier does the slicing, normalizing and voting over windows.

### Training data

- **Synthetic signals** from sdrexplorer-train's generators, with the protocols' real sync words,
  deviations, framing and pulse shaping, passed through a fading channel model with frequency offset and
  noise (SNR −10…20 dB for HF, −4…20 dB for V/UHF).
- **`vu` and `vu48` only: real off-air recordings** of land-mobile and paging signals. Every recording's
  label was confirmed by an independent decoder (a valid color code, NAC, RAN or sync with clean error
  checks) before it was used. The recordings themselves are not published: they hold other people's
  transmissions.
- The `hf` model is trained on synthetic signals only.

### Evaluation

Accuracy on **real held-out recordings** counts: whole transmitter systems (every recording sharing a
frequency or a system id) are kept out of training and used only to test, so a model can't score well by
recognizing a transmitter it has heard. Synthetic test accuracy is also listed, but it overstates real
performance and is never used to choose a model.

### Limitations

- `hf` has never been measured on real signals. Its PSK / QAM order (QPSK vs 8PSK, 16QAM vs 64QAM) is weak,
  and S4285-like / S4529-like are approximations of those modems, not the real waveforms.
- The real training recordings are North American V/UHF signals: US channel plans, deviations and
  equipment. Elsewhere, expect lower accuracy on the real-data classes.
- Some classes have little or no real data to test on yet: POCSAG, SmartNet, carrier, "other" and the
  AX.25 classes have no real held-out recordings, and NXDN48 has few. `vu48`'s EDACS classes are trained on
  synthetic signals only.
- The classes are a closed set: a signal of a kind the model doesn't know is given the nearest known class
  (or "other"). In the app, sync-word checks confirm the digital modes.
- A model only says what a signal looks like; it decodes nothing.

## Installing

- **With sdrexplorer's installer** (the usual way): `install.sh` installs the release named in this
  repository's `manifest.json` whenever it installs the classifier, and checks every file against its
  sha256. `--no-models` leaves them out. Models you trained and deployed yourself with sdrexplorer-train are
  never replaced.
- **By hand:** download the release files, check them against `SHA256SUMS`, and put each in
  `<models folder>/<profile>/<version>/model.onnx` (the models folder is the classifier's `models_dir`
  path key, by default `~/.local/share/sdrexplorer/models`). Copy `models/<profile>/<version>/meta.json`
  and `eval.json` beside it, and add the profile's entry from this `manifest.json` (without `files`) to the
  models folder's `manifest.json`.

## Layout

```
manifest.json                         the release: per profile the classifier's entry + its files
                                      (release asset name, sha256, size, backend)
models/<profile>/<version>/meta.json  rate, window, classes, display names, bands
models/<profile>/<version>/eval.json  accuracy: real held-out, synthetic (counts only, no recording names)
MODELS.md                             the accuracy tables
LICENSE                               CC BY 4.0
```

The manifest's `files` map is keyed by the classifier's file name, and each file names the backend it is
for (`onnx` today). A later release can add compiled files for accelerators beside the ONNX file of
the same model, named `<profile>-<version>.<backend>.<ext>`; the ONNX file stays the CPU fallback.

## Making a release (maintainers)

1. Deploy the models to release with sdrexplorer-train (`sdrexplorer-train deploy …`; the gate applies).
2. `sdrexplorer-train publish <this checkout> --release models-X.Y.Z` writes the manifest, the cleaned
   metadata and MODELS.md here, and the release files into `dist/` (git-ignored).
3. Commit, tag `models-X.Y.Z`, push, and create the GitHub release with that tag, uploading every file in
   `dist/` (for example `gh release create models-X.Y.Z dist/*`).

A release changes the model files only by adding a new tag: the files of an existing release are never
replaced, so a manifest's checksums always match.

## Contributing recordings

The models improve most from real recordings of real signals. If you'd like to share some, see
"Sharing recordings for training" in sdrexplorer's CONTRIBUTING.md: the contribution tool keeps short crops
of the signal only and strips frequency, time, position and ids.
