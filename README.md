# Keyword spotting with `micro_speech`

EE 446 lab materials for keyword spotting on an Arduino target. The repository contains three model-preparation notebooks, an Arduino sketch, and the submitted lab PDF.

## What it does

`arduino/micro_speech.ino` runs an embedded TensorFlow Lite model (`g_model`, 19,104 bytes) with TensorFlow Lite Micro. Its model input is an INT8 buffer of 1,960 micro-feature values: 49 slices of 40 features. The configured output labels are `silence`, `unknown`, `yes`, and `no`.

Audio features are generated from 16 kHz input using 30 ms windows at a 20 ms stride. The command recognizer averages results over a 1,000 ms window, uses a threshold of 200, requires at least three results, and suppresses repeated detections for 1,500 ms.

For a new `yes`, `no`, or `unknown` detection, the Arduino responder logs `Heard ...` through the TensorFlow Lite error reporter and lights the board LEDs: green for `yes`, red for `no`, and blue for `unknown`.

## Model

The local notebook source sets `WANTED_WORDS = "yes,no"`, `MODEL_ARCHITECTURE = "tiny_conv"`, and `PREPROCESS = "micro"`. It downloads a pretrained `speech_micro_train_2020_05_10` checkpoint, converts a SavedModel to TensorFlow Lite with INT8 input and output, and exports the quantized model as a C/C++ byte array. The Arduino sketch accepts an INT8 model and registers DepthwiseConv2D, Conv2D, FullyConnected, Reshape, and Softmax operators.

## Hardware and tools

- The lab PDF identifies the deployment board as an Arduino Nano 33 BLE; the Arduino audio and responder implementations are compiled only when `ARDUINO_ARDUINO_NANO33BLE` is defined.
- Audio is captured through `PDM.h`, configured as mono at 16 kHz with gain 20. The checkout does not identify the microphone component or state that the board is a Nano 33 BLE Sense, so this README does not make either claim.
- The sketch requires an Arduino environment that supplies `TensorFlowLite.h`, the TensorFlow Lite Micro headers used by the sketch, and `PDM.h`. No Arduino library version, board-core version, Arduino CLI configuration, or successful build log is included.
- The notebooks download their model/data dependencies at runtime. There is no pinned Python environment or requirements file in the checkout.

## Run the Arduino sketch

This procedure is derived from the source and is not verified by a build artifact in this repository.

1. Use an Arduino setup for the Nano 33 BLE target that provides the headers above.
2. Open `arduino/micro_speech.ino` with the companion source files in `arduino/`, build it, and upload it to the board.
3. After the sketch starts, it waits for PDM samples, continuously generates features, and invokes the embedded model when new audio arrives.
4. Speak `yes` or `no` near the PDM audio input. The responder's LED behavior is defined above. The lab PDF asks for keyword identification to be observed in the Arduino IDE Serial Monitor; the source itself emits the detection through TensorFlow Lite's error reporter rather than calling `Serial` directly.

## Credits and license

The root [LICENSE](LICENSE) is MIT and names Sparsh Dadhich (2026). The Arduino source files carry TensorFlow Authors Apache-2.0 headers. The local notebook identifies the model workflow as derived from TensorFlow Lite Micro's `micro_speech` example (v2.4.1).
