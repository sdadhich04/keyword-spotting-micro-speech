# Keyword Spotting (micro_speech) — Arduino Nano 33 BLE Sense

**EE 446: Tiny Machine Learning for Ultra Low-Power Edge Computing | University of Washington, Spring 2026**

TensorFlow Lite Micro's classic keyword-spotting pipeline — detecting the words "yes" and "no" from streaming microphone audio — explored offline in Python and deployed to an Arduino Nano 33 BLE Sense.

---

## What it does

1. **Audio representation** — explores time-domain waveforms, FFTs, spectrograms, and MFCC-style Mel features for short speech clips
2. **Model conversion** — takes a pretrained keyword-spotting model, converts it to TensorFlow Lite, and applies full integer post-training quantization
3. **On-device inference** — the quantized model runs continuously on the Arduino, listening to the PDM microphone and printing "yes" / "no" / "unknown" / "silence" detections over Serial

---

## Repository contents

```
Lab5_EE446.pdf                                    ← Written report
notebooks/
  TinyML_Lab5_Google_Colab.ipynb                  ← Colab version (audio recording via browser mic)
  TinyML_Lab5_Google_Colab_v2.ipynb               ← Second Colab run/pass (kept as a separate artifact — see note below)
  TinyML_Lab5_Local_Env.ipynb                     ← Local Jupyter version (records from laptop mic, no Colab-specific commands)
arduino/                                          ← Full TFLite Micro "micro_speech" sketch
  micro_speech.ino                                ← Main sketch
  main_functions.h                                ← Shared setup/loop logic
  audio_provider.h + arduino_audio_provider.cpp    ← Microphone capture (Arduino Nano 33 BLE Sense target)
  audio_provider.h + sparkfun_edge_audio_provider.cpp   ← Microphone capture (SparkFun Edge target — inactive on Arduino builds)
  command_responder.h + arduino_command_responder.cpp   ← Output handling (Arduino target)
  command_responder.h + sparkfun_edge_command_responder.cpp  ← Output handling (SparkFun Edge target — inactive on Arduino builds)
  feature_provider.cpp / .h                        ← Converts raw audio into model input features
  recognize_commands.cpp / .h                      ← Smooths per-frame model output into stable command detections
  micro_features_*.cpp / .h                        ← Feature generator, model settings, embedded model weights, and reference "yes"/"silence" feature data
```

> The `arduino_*` and `sparkfun_edge_*` source files are both present because this is the standard TFLite Micro multi-platform example layout — each pair implements the same header for a different board, guarded by platform macros. For the Nano 33 BLE Sense, only the `arduino_*.cpp` files are active; the `sparkfun_edge_*.cpp` files compile to no-ops and can be left in place or removed.

**Note on the two Colab notebooks:** `TinyML_Lab5_Google_Colab.ipynb` and `_v2.ipynb` are not identical -- they differ in run/output state (1000+ lines of diff). Both are kept since it wasn't clear which run represents the final submitted results; open both if you need to compare. Also worth knowing: despite the filename, the saved outputs in these notebooks were executed on a local machine, not in an actual Colab session (the embedded execution metadata doesn't match a Colab environment) -- functionally the results are the same either way, but treat the "Colab" label as historical/nominal rather than a guarantee of where it was run.

---

## Quick start

### Run the notebook

```bash
jupyter notebook notebooks/TinyML_Lab5_Local_Env.ipynb
```

The local-env notebook installs its own small audio dependencies on top of the standard TinyML environment (TensorFlow, NumPy, SciPy, Matplotlib, Jupyter). Use the Colab notebooks instead if you'd rather run in-browser with no local setup.

### Flash the Arduino sketch

1. Install the `Arduino_TensorFlowLite` library via Arduino Library Manager (or the `tflite-micro-arduino-examples` library)
2. Copy the entire `arduino/` folder contents into a sketch folder (Arduino IDE requires the folder name to match the `.ino` filename — rename the folder to `micro_speech/` if needed)
3. Open `micro_speech.ino`, select **Arduino Nano 33 BLE Sense** as the board, upload
4. Open Serial Monitor — say "yes" or "no" near the board's microphone to see detections

---

## Hardware

- **Arduino Nano 33 BLE Sense** (Nordic nRF52840, onboard PDM microphone)

---

## Authors

Sparsh Dadhich — University of Washington, ECE / Neuroscience

---

## License

MIT — see [LICENSE](LICENSE). This covers the author's own code, notebooks, and documentation in this repo. The `arduino/` sketch is adapted from TensorFlow's Apache-2.0-licensed micro_speech example; TensorFlow/TFLite Micro itself remains under its own Apache 2.0 license.
