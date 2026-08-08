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

#### Raw Explanation

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

#### Raw Explanation

**About the project:**

- Just like Document Classifier, except for the fact that it is now a generic image classifier. For each image it will spit out multiple tags which the image appears to contain.
- It currently supports around 2000 tags. This tag set has been designed by PMs based on user analysis.
- It is a part of the overall Gallery Search Pipeline.
- It is an essential part, since most of the searches in samsung mobiles (as per user-trial survey 2024-25) is single keyword based. For such cases, results from image tagger are sufficient.

**What I did?**

- Like document classifier, I helped deploy this project in Samsung Flagship devices.
- I faced similar challenges of accuracy issues like document classifier.
- The error used to get reported in this format: "A particular search result is not appearing", or maybe "A wrong result is appearing".
- My task was to first narrow down the cause of the error in the entire pipeline. It can be a query parsing issue, or a database filling issue, or incorrect preprocessing of image, or improper post processing of model outputs, or ondevice accuracy drop, or model's innate incapability.
- If its issue from my C++ pipeline, then I solved them.

#### Refined Explanation

1. [SDE] End-to-End Root Cause Analysis
    - [Problem]
      - User-reported issues were often high-level: "Image X is not appearing for query Y" or "This image should not appear for this query"
      - The actual issue could originate from multiple independent components.
    - [Solution]
      - Built a systematic debugging process that validates each stage independently: query parsing, database population, indexing, image preprocessing, model inference, postprocessing.
      - Isolated failures to the responsible subsystem before escalating or fixing.
2. [SDE] Deployment Accuracy Regressions
    - [Problem]
      - Model quality observed on-device occasionally differed from evaluation results.
    - [Solution]
      - Compared intermediate outputs across environments.
      - Verified preprocessing and postprocessing consistency.
      - Added validation checks to identify deployment-specific discrepancies.
3. [SDE] Cross-Team Investigation
    - [Problem]
      - Many failures spanned multiple teams and components.
    - [Solution]
      - Collected evidence from different pipeline stages.
      - Narrowed ownership boundaries before escalation.
      - Coordinated with search, indexing, and model teams.
4. [ML] Understanding Search Quality Failures
    - [Problem]
      - A poor search result does not necessarily indicate a model failure.
      - Potential causes included: preprocessing issues, indexing issues, retrieval issues, model limitations.
    - [Solution]
      - Performed error analysis across the entire pipeline.
      - Distinguished model errors from system-level errors.
      - Prevented unnecessary model retraining efforts.
5. [ML] Training vs Deployment Consistency
    - [Problem]
      - Even small deployment differences can cause accuracy degradation.
    - [Solution]
      - Compared intermediate activations and outputs.
      - Verified image preprocessing correctness.
      - Ensured inference behavior matched reference environment.
6. [ML] Multi-Label Classification Quality
    - [Problem]
      - Images often contain multiple valid concepts.
      - Errors could arise from: missing tags, low-confidence predictions, postprocessing thresholds.
    - [Solution]
      - Investigated prediction distributions.
      - Analyzed confidence thresholds.
      - Evaluated downstream impact on retrieval quality


## Gallery Platform Services | Gallery Search - Galaxy S26

#### Raw Explanation

**Nature of work in Samsung Gallery Search Pipeline (in general):**

- In samsung galaxy devices, there are various background services, like IPService, MediaService, FaceService, etc. which orchesterates various solutions.
- There is a central Database called SEC-MP DB, which is the master Database for all services.
- Individual services may have their internal DB, but they should iteratively do forward and reverse sync with SEC-MP DB.
- For example, IPService (in S25 and before) used to handle solutions like Image Tagger, Document Classifier, and Pet Clustering (clustering pet images). FaceService was responsible for face clustering. MediaService used to tackle other solutions related to samsung gallery search like - query parser, Natural Language Search, etc.
- Usually these Services are maintained and collaborated by multiple RnD centeres and HQ.
- Some teams work on solution provider (Native C++ deployment); some work on model development for these solutions (they don't care about on-device deployment); some teams work on service side code (orchestration - like DB handling and solution policy - of various solutions under their domain)
- Depending upon team dynamics, solutions are sometimes reorganized into different services.

**About the Project:**

- For S26, it was decided that PetClustering related solution will move from IPService to FaceService.
- So, we had to do the code migration.
- The tech-stack this time was Android Java.

**What did I do?**

1. Migration of Pet Clustering:
    - I personally was responsible for making sure all previous DB operations (backward compatibility with older devices) are migrated carefully into the new design pattern.
    - I specially was given the task for migrating Smart Switch related code from previous service to new service, while keeping the previous philosophies same.
    - This required a lot of aggressive testing, and resolving on-demand issue resolution.
    - This work was mainly on the SDE side. It had very less (or almost nothing) to do with ML.

#### Refined Explanation

1. [SDE] Service Ownership Migration
   - [Problem]
     - As part of a platform reorganization, a production image-clustering feature was migrated from one Android background service to another.
     - The challenge was not merely moving code, but ensuring that all existing functionality, data flows, and integrations continued to operate correctly under the new ownership model.
   - [Solution]
     - Analyzed existing service interactions and dependencies.
     - Migrated feature-specific persistence and synchronization logic into the new service architecture.
     - Validated compatibility across fresh installs, upgrades, and existing user states.
2. [SDE] Backward Compatibility & Data Integrity
   - [Problem]
     - Millions of devices may already contain metadata generated by older software versions.
     - Any incompatibility in schema interpretation, synchronization logic, or migration flow could lead to: (1) missing data, (2) inconsistent state, (3) feature regressions after upgrade.
   - [Solution]
     - Carefully studied legacy database interactions and business logic.
     - Preserved historical behavior where required.
     - Extensively validated upgrade scenarios to ensure seamless migration of existing user data.
3. [SDE] Smart Switch Migration
   - [Problem]
     - Users transferring data to a new device through Smart Switch expect their previously generated feature metadata to remain available. The migration introduced risk of incompatibility between: (1) Existing backup formats (2) new service architecture (3) restored device state
   - [Solution]
     - Owned migration of Smart Switch-related workflows.
     - Preserved existing transfer semantics and metadata handling philosophy.
     - Performed extensive end-to-end testing of backup and restore scenarios.
4. [SDE] Regression Prevention During Migration
   - [Problem]
     - Migration projects are particularly risky because functionality already exists and users expect identical behavior after the change.
     - Failures are often discovered only after deployment.
   - [Solution]
     - Designed aggressive test plans covering: (1) New installs (2) upgrades (3) device transfers.

In Short, The hardest part was preserving behavior rather than implementing new functionality. The feature already existed in production, and users already had persisted metadata, synchronization state, and Smart Switch backups. My responsibility was to ensure that moving the feature to a new service architecture remained completely transparent to users. Most of the work involved understanding legacy behavior, identifying hidden dependencies, validating migration paths, and preventing regressions across upgrade and device-transfer scenarios.

## Image Grouping | Gallery Search - Galaxy S26

#### Raw Explanation

**About the Project:**

- Image grouping is a feature in samsung gallery to group similar images into a single thumbnail (when seen in gallery).
- It helps remove clutter, and gives a good user experience.
- It is a legacy solution, present even in older samsung galaxy devices.
- However, traditionally, embeddings from Image Tagger model, was being used for doing clustering.
- This was okay, but it had cornered cases, where it failed.
- Since, image tagger was primarily trained for classification task, it gave more importance to the category of object rather than the actual appearance of the object.
- So, two pictures of different coloured cups, often had extremely close embeddings. While this was good for image classification task, it did not meet the requirements of image grouping efficiently.
- Often, we struggled with choosing a threshold for the cosine similarity between embeddings.
- Hence, for S26 it was decided to build a standalone dedicated image clustering solution, which is extremely lightweight.


**What I did?**

- The requirements given from HQ were often not very clear. So, first I had to do a lot of back and forth email to understand the exact requirement. This required me to make assumptions, communicate them clearly, and refine the understanding based on feedbacks. This was an iterative process.
- I was told "we want a better and lighter solution". But what does it mean to be "better"?
- Till now, there were no objective measures to evaluate the goodness of any solution.
- I did literature review to come up with objective metrics and subjective testing strategies.
- Then I created benchmark datasets which can be used to evaluate older and newer solutions.
- Having that done, again me along with my colleague did literature to get lightweight architectures for our usecase.
- We came up with custom loss functions (NTExent and Contrastive Loss) to train the model.
- Once, happy with our PC performance, we wrote its ondevice implementation and DB handling in the Android Service in Java.

#### Refined Explanation

1. [ML] Misalignment Between Classification and Clustering Objectives
   - [Problem]
     - The legacy image grouping solution reused embeddings generated by an image classification model.
     - These embeddings were optimized for semantic categorization rather than visual similarity.
     - As a result, visually different images belonging to the same category often received highly similar embeddings.
     - Example: Images of a red cup and blue cup were embedded closely because the model prioritized the concept of "cup" over appearance.
     - This led to unstable clustering behavior and difficulty in selecting robust similarity thresholds.
   - [Solution]
     - Reframed the problem from image classification to visual similarity learning.
     - Proposed a dedicated embedding model specifically optimized for image grouping.
     - Evaluated metric-learning approaches better suited for clustering use cases.
     - Replaced category-focused representations with embeddings designed to preserve visual similarity.
2. [ML] Lack of Objective Evaluation Metrics
   - [Problem]
     - Requirements from stakeholders were expressed qualitatively as:
       - "Make it better."
       - "Improve grouping quality."
     - There was no agreed-upon objective definition of what constituted a better grouping solution.
     - Existing evaluations were largely subjective and difficult to reproduce.
   - [Solution]
     - Conducted literature review on image clustering and similarity evaluation.
     - Identified objective clustering metrics suitable for the problem domain.
     - Designed subjective evaluation protocols to complement quantitative metrics.
     - Established a repeatable benchmarking framework for future comparisons.
3. [ML] Absence of Benchmark Datasets
   - [Problem]
     - No standardized benchmark existed to compare the legacy and proposed solutions.
     - Evaluations were inconsistent and often dependent on ad-hoc testing.
   - [Solution]
     - Created benchmark datasets covering representative real-world scenarios.
     - Included both positive and hard-negative examples.
     - Established a reproducible evaluation pipeline for comparing multiple model variants.
4. [ML] Learning Effective Similarity Representations
   - [Problem]
     - Traditional classification losses optimize category separation rather than similarity preservation.
     - The task required embeddings where visually similar images remain close while dissimilar images remain far apart.
   - [Solution]
     - Investigated metric-learning approaches from recent literature.
     - Experimented with:
       - NT-Xent Loss
       - Contrastive Loss
     - Trained lightweight embedding models specifically for clustering objectives.
5. [ML] Balancing Accuracy and On-Device Constraints
   - [Problem]
     - The new solution needed to improve grouping quality while remaining suitable for mobile deployment.
     - Larger models improved representation quality but increased latency and memory usage.
   - [Solution]
     - Conducted literature review of lightweight architectures.
     - Evaluated tradeoffs between clustering quality and deployment cost.
     - Selected models that satisfied both quality and mobile resource constraints.
6. [SDE] Ambiguous Requirements and Requirement Discovery
   - [Problem]
     - Initial requirements from HQ were high-level and qualitative.
     - Terms such as "better" and "lighter" lacked measurable definitions.
     - Without clear success criteria, implementation risk was high.
   - [Solution]
     - Conducted multiple rounds of technical discussions with stakeholders.
     - Explicitly documented assumptions and proposed interpretations.
     - Iteratively refined requirements based on stakeholder feedback.
     - Converted qualitative goals into measurable engineering objectives.
7. [SDE] Building Evaluation Infrastructure
   - [Problem]
     - No common framework existed for comparing image grouping solutions.
     - Regression detection and performance comparison were difficult.
   - [Solution]
     - Designed benchmarking workflows to evaluate multiple model variants consistently.
     - Created reusable evaluation datasets and testing procedures.
     - Enabled objective comparison of legacy and new solutions.
8. [SDE] Replacing a Production System Safely
   - [Problem]
     - The legacy grouping solution was already deployed in production devices.
     - Replacing it introduced risk of:
       - Regressions
       - Unexpected grouping behavior
       - User experience degradation
   - [Solution]
     - Established comparison baselines between old and new systems.
     - Performed extensive validation before rollout.
     - Used benchmark-driven evaluation to minimize deployment risk.
9. [SDE] Android Service Integration and Metadata Management
   - [Problem]
     - The newly trained model needed to integrate into existing Android service workflows.
     - Generated clustering metadata needed to be persisted and maintained correctly.
   - [Solution]
     - Implemented on-device inference pipeline.
     - Developed service-side database handling and metadata management logic.
     - Integrated the solution into the existing gallery infrastructure while maintaining compatibility with existing workflows.

## Smart Cropper | Gallery Search - Galaxy S26

#### Raw Explanation

**About the Project:**

- Image cropper is a solution which is used by many downstream tasks. Like thumbnail generation, gallery story creation, etc.
- Image Cropper was a project which was stopped almost 2 years ago, due to stable.
- The original authors of the project had long left the team.
- We got KT from second generation authors.
- Neverthless, I documented everything that was taught.
- After the project was reopened, it was asked to deploy the solution without relying on SNAP Layer, and secondly to run on CPU, but with reduced memory and inference timings, while maintaining the accuracy.
- In Samsung devices, we have an internal SNAP Layer, which takes care of all model loading and model execution under the hood. We as solution owners need to take care of the overall signals that is being passed to the model (from image reading, to preprocessing, and then hadling postprocessing once snap returns the raw outputs).
- However, due to some internal reorganization, we were asked to get rid of SNAP. This was a massive step, as it required significant architectural changes to the C++ Native codebase.
- Additionally, we were forbidden to use GPU, but we were tasked with reducing inference time by almost half, while maintaing the accuracy.

**What I did?**

- Legacy Cleanup:
  - At first we thought of reading the legacy codebase (which was too much complex due to extreme post-processing). The team was handling image cropping as an object detection task. More speficifically a salient object detection task.
  - We deligiently looked through the codebase to find scopes of optimizations, and we actually found many. We found that there were redundant processing and transformation of input and output image buffers. This might have been added to the codebase due to strict release timelines, and gradual additions of sphegetti code. We took pains to remove all that crap.
  - Just doing so brought down the overall inference time per image from 102 ms to around 34 ms (68% reduction).
- Architecture Changes:
  - I implemented the code changes from SNAP to direct reliance on Google's tensorflowlite libraries.
- Model Development:
  - Did literature survey to get models (nanodet shufflenet) that could suit our purpose.
  - Used existing data to do the training, evaluation, model conversion (TFLITE and FP32 and INT8 dynamic range quantization), and final deploying to on-device.
- Lottie Animation:
  - Additional requirement came from HQ that a separate solution (called Story Service - which is responsible for gallery story/memory creation) wants to use Lottie Animation for Recap feature. This required implementation of additional JNI APIs to our legacy image cropper solution, while keeping the soul of the existing codebase same (to avoid problems in other downstream solutions which might be using our solution APIs)
  - Did all that in record time, before the Fold-and-Flip launch.


#### Refined Explanation

- [SDE] Reviving an Abandoned Legacy System
  - [Problem]
    - The project had been inactive for nearly 2 years.
    - The original authors had already left the team.
    - Existing ownership continuity was weak, and the codebase was difficult to understand due to lack of updated documentation.
    - New requirements required major modifications on top of this legacy system.
  - [Solution]
    - Took knowledge transfer from second-generation maintainers.
    - Carefully documented the inherited architecture and pipeline behavior.
    - Built a working understanding of the complete inference and post-processing flow before introducing changes.

- [SDE] Legacy Code Cleanup & Performance Optimization
  - [Problem]
    - The legacy codebase had accumulated several redundant processing steps over time.
    - Multiple unnecessary image buffer transformations and repeated post-processing passes increased execution complexity.
    - These inefficiencies significantly impacted runtime performance.
  - [Solution]
    - Conducted detailed code-path analysis and profiling.
    - Identified redundant preprocessing, post-processing, and buffer transformations.
    - Simplified the execution pipeline while preserving output behavior.
    - Removed unused and duplicated code paths.
  - [Impact]
    - Reduced per-image processing latency from: 102 ms → 34 ms. Achieved approximately 68% latency reduction.
- [SDE] Inference Engine Migration (SNAP → TensorFlow Lite)
  - [Problem]
    - The legacy system depended on Samsung’s internal SNAP inference layer.
    - Due to organizational restructuring, the project had to remove this dependency entirely.
    - This introduced major architectural challenges:
    - direct model loading
    - tensor memory management
    - inference lifecycle ownership
  - [Solution]
    - Replaced SNAP dependency with direct TensorFlow Lite integration in native C++.
    - Implemented model loading, tensor preparation, and inference execution pipelines.
    - Preserved compatibility with existing preprocessing and post-processing modules.
- [SDE] API Stability During Feature Expansion
  - [Problem]
    - A downstream Story Service introduced a new requirement for Lottie-based recap generation.
    - This required exposing new JNI APIs from the native cropper solution.
    - Existing downstream consumers depended on stable APIs, so changes risked regressions.
  - [Solution]
    - Designed and implemented additional JNI APIs for the new use case.
    - Preserved backward compatibility for existing API consumers.
    - Isolated new functionality to minimize disruption to legacy integrations.

- [ML] Lightweight Model Selection for Edge Deployment
  - [Problem]
    - New requirements demanded: CPU-only execution, lower latency, reduced memory footprint, maintained cropping accuracy
  - [Solution]
    - Conducted literature survey on lightweight detection architectures.
    - Evaluated models such as: NanoDet ShuffleNet
    - Benchmarked tradeoffs between accuracy, latency, and model size.
- Model Compression and Deployment Optimization
  - [Problem]
    - FP32 models were too expensive under CPU-only constraints.
    - The challenge was reducing inference cost without significantly degrading crop quality.
  - [Solution]
    - Converted trained models to: TensorFlow Lite FP32, INT8 dynamic-range quantized variants
    - Benchmarked deployment tradeoffs and selected the most efficient variant.

In Short

The hardest part of this project was modernizing a dormant legacy vision system while meeting aggressive performance and architectural constraints. We had to remove dependency on an internal inference abstraction layer, migrate to direct TensorFlow Lite execution, optimize the codebase for CPU-only deployment, and significantly reduce latency—all while maintaining accuracy and preserving compatibility for multiple downstream consumers. Much of the work involved reverse-engineering legacy behavior, cleaning accumulated technical debt, and carefully balancing performance, maintainability, and product stability.

## VLM & Adaptor Finetuning | Gallery Search - Galaxy S26


# Publications & Mentorship

## ADORE | ICASSP 2026

- Co-authored **ADORE: Asymmetric Relational Distillation with Reranking for Instance Level Image Retrieval**, an instance-level image retrieval framework designed to achieve an accuracy–efficiency trade-off for resource-constrained devices.
- Contributed primarily to **experimentation, training/debugging, and ablation studies**, helping validate the proposed knowledge-distillation and asymmetric re-ranking components.
- Worked on identifying and resolving **training-related bugs** during experimentation and validating experimental results across different configurations.
- Contributed to the **final paper writing and refinement**, helping consolidate experimental findings and technical discussions.
- Took significant responsibility for the **ICASSP 2026 presentation**, including preparation of the final presentation deck and presenting the work at the conference.
- The paper evaluates the proposed approach on established instance-retrieval benchmarks including **ROxford5k and RParis6k** and reports improvements over prior asymmetric retrieval/reranking approaches. :contentReference[oaicite:0]{index=0}

## Mentorship | 2-Month Intern – Screenshot Classifier

- Mentored a **2-month intern** working on an on-device screenshot classification project, providing technical direction throughout the internship.
- Designed and structured the **experimentation roadmap**, breaking the project into actionable experiments and defining the sequence in which they should be performed.
- Guided the intern in developing practical engineering skills beyond ML experimentation, including understanding an existing codebase, debugging, evaluation, and working within real-world software constraints.
- Helped bridge the gap between **academic/project-level knowledge and production engineering**, introducing considerations such as requirements, implementation constraints, evaluation, and practical trade-offs.
- Reviewed and guided the intern's technical presentations and progress updates, ensuring that the work and experimental findings were clearly communicated to managers and the internship evaluation panel.
- Took responsibility for the intern's overall technical growth and project direction, adapting the mentoring approach based on progress and experimental outcomes.