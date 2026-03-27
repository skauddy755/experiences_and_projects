# Internship (3 months) | NIC - Dhanbad District Administration

Duration: AUG 2020 - NOV 2020

- Developed e-samadhan, a grievance redressal portal, and PMU, a project monitoring system, with secure backends
and data dashboards, digitizing administrative processes; maintaining citizen services during COVID-19; and
enabling officials to track grievances and DMFT project progress.
- Implemented the platforms using PHP (CodeIgniter), MySQL, JavaScript, HTML, CSS, and deployed them on
Apache servers, incorporating authentication mechanisms and analytics features.

# Internship (2 months) | Samsung R&D Institute India - Bangalore

Duration: MAY 2022 - JUL 2022

## Under Display Camera (UDC)

- Designed a software pipeline to generate synthetic flare and clean image pairs, creating training data to improve
photo quality in Under-Display Cameras (UDC).
- Project Domain:
  - Visual Intelligence for Samsung Flagship Camera for under-the-display camera.
- Objective:
  - Synthetic Data generation for UDC Flare removal and PSFI.
- Goals:
  - Literature survey about available flare removal methods.
  - Python based implementation of selected method based on above study.
- Highlights of Activities:
  - Understanding of wave optics for UDC.
  - Generate Random Dirty Aperture Mask (to simulate real-life lenses).
  - Generate PSF given aperture and defocus phase.
  - Feed camera properties to generate synthetic flares using PSF.
- Tech Stacks: Python, C++, OpenCV, and PyTorch for image processing and ML integrations.

# Full Time (2 years 10 months) | Samsung R&D Institute India - Bangalore

Duration: AUG 2023 - PRESENT

## Depth Estimation | Portrait Mode - Galaxy S24

- Problem Description:
  - Depth Estimation is an important component to achieve good Bokey output.
  - In particular, I worked on monocular depth estimation. This avoids the use of multiple cameras to achieve stereo image pairs.
  - I used dual-pixel data along with RGB image, to give the model additional cues for better depth estimation.
  - We experimented with efficientNet and MobileNetv2 models.
  - Since portrait mode is heavily biased towards human subjects, we introduced a specialized branch to better capture human-specific depth priors like face structure and body contours, improving perceptual quality of bokeh.
  - Since, it was difficult to provide absolute depth. We rather focused on providing a relative depth (which is just enough for the Bokeh usecase). For this purpose we used Scale and Shift invariant Loss.
- Challenges faced and their attempted resolutions:
  1. [ML] Low-light scenarios.
     - Low SNR → noisy RGB inputs → degraded depth predictions
     - Dual-pixel signal also becomes unreliable in extreme low light
     - [Solution-1] Noise-aware training:
       - Explicitly simulate sensor noise models (Poisson + Gaussian noise)
       - Helps model generalize beyond just “seen data”
     - [Solution-2] Exposure / brightness invariance
       - During training used augmentations like gamma correction, brightness scaling, contrast stretching.
       - This ensured augmentations matched real sensor characteristics.
  2. [ML] Difficulty to capture fine-grained textures. Solutions: Capturing more data on the scenario.
     - CNNs lose high-frequency details.
     - Hair, edges, thin structures → poorly captured.
     - Depth maps become over-smoothed.
     - [Solution-1] Multi-scale feature fusion
       - Use skip connections (like U-Net / FPN style)
       - Combine low-level (edges) with high-level (semantics)
     - [Solution-2] Experiment with edge-aware losses
       - Penalize errors more at boundaries
       - Used Gradient Loss
     - [Solution-3] Auxiliary Supervision
       - Added tasks like segmentation, and edge-detection.
       - Particularly for humans.
  3. [SDE] Hardware Accelerations (CPU vs GPU vs NPU)
     - CPU-only inference too slow. GPU/NPU support varied across devices.
     - [Solution-1] Integrated with vendor SDKs / custom delegates. Implemented dynamic backend selection across CPU/GPU/NPU to ensure optimal performance across heterogeneous hardware configurations.
  4. [SDE] Model Integration into C++ Pipeline
     - Model trained in Python (PyTorch/TensorFlow). But Needed deployment in C++ runtime
     - [Solution-1] Bridged Python-trained models to a C++ inference pipeline via ONNX/TFLite conversion and custom runtime wrappers, ensuring seamless integration with the camera stack. (1) Converted model to onnx / tflite. (2) Wrote custom C++ wrappers for inference engines.
     - Ensured input/output tensor consistencies.
  5. [SDE] Debugging and observability
     - Hard to debug. Silent failures in the pipeline.
     - [Solution] Improved observability by introducing intermediate tensor logging and offline replay tools, significantly reducing debugging time for production issues.
  6. [SDE] Stability & Edge Case Handling
     - Sudden exposure changes. Motion Blur.
     - Could crash or produce unstable outputs
     - [Solution] Improved system robustness by adding input validation and fallback mechanisms, ensuring graceful degradation (disable depth if confidence is low) under edge-case scenarios.

## Document Classification | Gallery Search - Galaxy S25

#### Raw Explanation:

```
- This is used for searching document iamges in your gallery. Category includes day-2-day items like: passport, credit card, notes, books, maps, barcode, legal documents, presentation slides, etc.
- Here, I did not work much on ML related stuff. I was mostly into C++ integration, and corresponding handling in Android side. To handle DB updates and batch processing of images as background services.
- I struggled with trying to match PC side accuracy with on-device. Turned out it was due to difference in interpolation strategies of PIL and opencv. Since, the model shared to us was extremely sensitive to such changes. So, I had to visit the official source code of PIL and opencv, and come up with my custom resize function. This ensured that signals going to the model are the same in both the environments.
- The previous on-device pipeline used a vision-only document classifier model. But the newly shared model was a VLM and it also used OCR. Here were two challenges: (1) To avoid redundancy I had to sync up with a team who already had OCR Solution on device. I had to ask them to expose APIs and discuss appropriate input and output shapes to ensure accurate OCR inference. (2) These OCR outputs needed to be passed to a tokenizer before sending it to the document classifier VLM. In python side, they had used AutoTokenizer from huggingface. But I did not have the same priviledge in on-device C++ pipeline. So, I had to research a bit and came accross sentencepiece tokenizer which had exact specs as the huggingface tokenizer. This open source code needed be augmented with some additional APIs, compiled as a separate .so file, and then used inside the document classifier ondevice c++ inference pipeline.
- The OCR was an additional cue that was sent to the model. Model developers (on seeing the scale of the challenges with ondevice) initially decided to compromise the accuracy by leting go of OCR. However, I pitched in and propsed the above mentioned solutions. This increased the overall classification accuracy by 4%.
- Furthermore, I proposed pipeline optimizations to further improve memory footprint by 5% and overall latency by 10%.
- I also actively contributed to the evaluation and testing process.
```

#### Refined Explanations

- [One-line sumary] Worked on on-device document classification for gallery search, integrating a vision-language model (VLM) with OCR signals in a C++ inference pipeline on mobile devices.
- Challenges:
  1. [SDE] Cross-platform inconsistency (Major Debugging Win)
      - Significant accuracy drop on-device vs Python baseline. Root cause initially unclear.
      - [RCA] Traced issue to image preprocessing mismatch. Identified interpolation differences between PIL (training) and opencv (deployment)
      - [Solution] Studied source implementation of both libraries. Implemented custom resize function in C++ to match training pipeline exactly.
      - [Impact] I debugged a non-obvious accuracy drop to preprocessing inconsistencies and implemented a custom resize operator to ensure bit-level alignment with the training pipeline.
  2. [SDE] Integrating OCR into VLM Pipeline
      - VLM required OCR input, but OCR existed as a separate system. There was no standardized system.
      - [Solution] Collaborated with OCR team to Define API contracts and align input/output formats.
      - [Impact] Enabled multimodal inference on-device
  3. [SDE] Tokenization Challenge (HuggingFace - C++ gap)
      - Python used AutoTokenizer. No equivalent on-device C++ pipeline.
      - [Solution] Identified sentencepiece as compatible tokenizer. Extended open-source implementation: Added required APIs and compiled into `.so` binary. Integrated into inference pipeline.
      - [Impact] Bridged the gap between Python-based tokenization and C++ deployment by adapting and integrating a SentencePiece tokenizer into the on-device pipeline.
  4. [Ownership] Ownserhip and Product Impact
      - Team planned to remove OCR due to deployment complexity.
      - [Solution] Proposed the above mentioned E2E solution for OCR integration along with tokenization. Improved classification accuracy by 4%
  5. [SDE] Performance Optimizations
      - [Problem] Legacy codebase. Hence, over time, the pipeline had accumulated inefficiencies due to incremental change.
      - [Solution] Systematic pipeline simplification and removal of inefficiencies.
        - There were multiple stages doing similar transformations—like resizing or normalization—more than once due to legacy additions over time. I consolidated those into a single step.
        - Intermediate buffers were being copied across stages unnecessarily. I reduced copies by reusing buffers where possible.
        - There were conditional branches for older models/features that were no longer in use but still part of execution flow. I removed those to simplify the pipeline.
        - I improved data flow between modules to avoid repeated format conversions and unnecessary transformations.
      - [Impact] Optimized on-device inference pipeline by refactoring legacy code, removing redundant processing steps, and eliminating unused code paths, resulting in ~10% latency reduction and ~5% lower memory footprint.
      - [Summarized Explanation] The optimizations were less about algorithmic changes and more about cleaning up a legacy pipeline that had accumulated redundant steps over time. I focused on removing duplicate preprocessing, reducing unnecessary memory copies, and simplifying execution flow, which collectively improved both latency and memory usage.


## Image Tagger | Gallery Search - Galaxy S25

