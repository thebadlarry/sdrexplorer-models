# Models in release models-1.0.0

Written by `sdrexplorer-train publish`. Accuracy is the share of signal windows (one window = 4096 samples at the model's rate) classified correctly. *Real held-out* are off-air recordings the model never trained on (whole transmitter systems are held out, not just crops); *synthetic* is the generated test set.

| Profile | Version | Input | Classes | Real held-out | Synthetic | Files |
|---|---|---|---|---|---|---|
| `hf` | res_hf-v1 | 9.6 kHz × 4096 | 19 | — (synthetic only) | 88.1% | `hf-res_hf-v1.onnx` |
| `vu` | r1-res | 24 kHz × 4096 | 13 | 93.6% (2,908 windows, 18 recordings) | 96.2% | `vu-r1-res.onnx` |
| `vu48` | w2-res | 48 kHz × 4096 | 15 | 97.8% (3,516 windows, 18 recordings) | 93.8% | `vu48-w2-res.onnx` |

## `hf` res_hf-v1

Classes: noise, AM, NBFM, USB, LSB, CW, FSK2, FSK4, BPSK, QPSK, 8PSK, 16QAM, 64QAM, S4285-like, S4529-like, PSK31, FT8, AX25-300, MSK144.

| Class | Real held-out | Synthetic |
|---|---|---|
| noise | — | 100.0% |
| AM | — | 98.0% |
| NBFM | — | 97.0% |
| USB | — | 96.5% |
| LSB | — | 93.5% |
| CW | — | 98.0% |
| FSK2 | — | 98.0% |
| FSK4 | — | 100.0% |
| BPSK | — | 98.0% |
| QPSK | — | 51.0% |
| 8PSK | — | 53.5% |
| 16QAM | — | 57.5% |
| 64QAM | — | 51.5% |
| S4285-like | — | 96.0% |
| S4529-like | — | 98.0% |
| PSK31 | — | 98.5% |
| FT8 | — | 91.5% |
| AX25-300 | — | 100.0% |
| MSK144 | — | 97.5% |

Measured with the `onnx` backend.

## `vu` r1-res

Classes: noise, NBFM, P25-P1, DMR, NXDN48, NXDN96, AX25-1200, AX25-9600, POCSAG, carrier, other, FLEX, SmartNet.

| Class | Real held-out | Synthetic |
|---|---|---|
| noise | 100.0% | 100.0% |
| NBFM | 86.1% | 90.0% |
| P25-P1 | 99.5% | 91.0% |
| DMR | 89.2% | 90.5% |
| NXDN48 | 97.7% | 99.5% |
| NXDN96 | 90.3% | 96.5% |
| AX25-1200 | — | 100.0% |
| AX25-9600 | — | 99.5% |
| POCSAG | — | 98.0% |
| carrier | — | 100.0% |
| other | — | 86.0% |
| FLEX | 94.0% | 100.0% |
| SmartNet | — | 99.5% |

Measured with the `onnx` backend.

## `vu48` w2-res

Classes: noise, NBFM, P25-P1, DMR, NXDN48, NXDN96, AX25-1200, AX25-9600, POCSAG, carrier, other, FLEX, SmartNet, EDACS, EDACS-V.

| Class | Real held-out | Synthetic |
|---|---|---|
| noise | 100.0% | 100.0% |
| NBFM | 90.7% | 78.5% |
| P25-P1 | 99.2% | 88.0% |
| DMR | 99.1% | 84.0% |
| NXDN48 | 93.5% | 99.5% |
| NXDN96 | 99.7% | 92.5% |
| AX25-1200 | — | 100.0% |
| AX25-9600 | — | 100.0% |
| POCSAG | — | 96.0% |
| carrier | — | 99.5% |
| other | — | 87.5% |
| FLEX | 98.6% | 100.0% |
| SmartNet | — | 98.5% |
| EDACS | — | 99.5% |
| EDACS-V | — | 83.5% |

Measured with the `onnx` backend.
