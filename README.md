<h1 align="center">PinpointQA: A Benchmark for Small Object-Centric Spatial Understanding in Indoor Videos</h1>

<p align="center">
  <a href="https://pinpointqa.github.io/">
    <img src="https://img.shields.io/badge/Project%20Page-Visit%20Website-4f46e5?style=for-the-badge&logo=googlechrome&logoColor=white" alt="Project Page"/>
  </a>
  <a href="https://github.com/PinpointQA/pinpointqa.github.io">
    <img src="https://img.shields.io/badge/Code-Repository-181717?style=for-the-badge&logo=github&logoColor=white" alt="Code Repository"/>
  </a>
</p>

<p align="center">
  <img src="assets/Overview.jpg" alt="PinpointQA Overview" width="100%"/>
</p>

<p align="center">
  <em>
    PinpointQA is a benchmark for small object-centric spatial understanding in indoor videos,
    with four progressive tasks: Target Presence Verification (TPV), Nearest Reference Identification (NRI),
    Fine-Grained Spatial Description (FSD), and Structured Spatial Prediction (SSP).
  </em>
</p>

> **Important:** This repository hosts the **project page** and releases **evaluation code** and **utility scripts** for PinpointQA. It does **not** redistribute original ScanNet++ / ScanNet200 scene assets or converted video files. Please obtain source data from the respective datasets under their licenses and use the provided conversion tools to prepare local videos for reproduction.

## 🧭 Overview

**PinpointQA** benchmarks whether multimodal models can localize small indoor objects from video and express their position with sufficient precision. The benchmark is built from ScanNet++ and ScanNet200 and contains **1,024 scenes** and **10,094 QA pairs**, organized as a progressive capability chain from target presence verification to structured spatial prediction.

This repository contains:

- The **project website** (`index.html`, `styles.css`, `script.js`, `assets/`)
- **Evaluation code**, **prompt template**, and **video preparation tools**

Ground-truth annotations are constructed from aligned 3D geometry and instance-level labels; evaluated models receive only sampled RGB video frames.

## 📰 News

- **2026**: PinpointQA project page and evaluation code are available in this repository.

## 📂 Repository Structure

```text
pinpointqa.github.io/
├── assets/                 # Figures and appendix for the project page
├── tools/
│   ├── convert_mkv_to_mp4.py
│   ├── convert_sens_to_mp4.py
│   └── convert_test_jsonl_to_gt_dir.py
├── eval.py
├── eval_utils/
│   ├── constants.py
│   ├── benchmark.py
│   ├── scoring.py
│   ├── reporting.py
│   ├── task3_judge.py
│   └── ...
├── eval_prompt.txt
├── index.html              # Project page (GitHub Pages)
├── script.js
├── styles.css
├── LICENSE
└── README.md
```

## ⚙️ Evaluation

### 1. Environment Setup

The evaluation code depends on the Python standard library and the OpenAI SDK, which is required by the current implementation for **FSD** scoring.

```bash
pip install openai
```

Task 3 (**FSD**) uses an OpenAI judge model, so you also need an API key:

```bash
export OPENAI_API_KEY=YOUR_OPENAI_API_KEY
```

On Windows PowerShell:

```powershell
$env:OPENAI_API_KEY="YOUR_OPENAI_API_KEY"
```

> **Note:** In the current implementation, `eval.py` initializes the FSD judge at startup, so `openai` and `OPENAI_API_KEY` are required for the full evaluation script.

### 2. Evaluation Workflow

The recommended workflow is:

1. Obtain the released PinpointQA `test.jsonl` (benchmark annotations).
2. Convert `test.jsonl` into scene-level ground-truth JSON files.
3. Organize your model predictions as one JSON file per scene.
4. Run `eval.py`.

### 3. Convert `test.jsonl` to `gt_dir`

The benchmark annotations are distributed in JSONL format, while the evaluator expects **scene-level GT JSON files**. We provide a conversion script:

```bash
python tools/convert_test_jsonl_to_gt_dir.py \
  --input_jsonl /path/to/test.jsonl \
  --output_dir /path/to/gt_dir \
  --overwrite
```

After conversion, `gt_dir` should look like this:

```text
gt_dir/
├── scene0000_00.json
├── scene0001_00.json
└── ...
```

Each GT file follows the evaluator's expected format, i.e. a top-level JSON object containing a `qa_pairs` list.

### 4. Prepare Predictions

The evaluator expects **one prediction JSON file per scene**, named as `<scene_name>.json`.

A recommended layout is:

```text
project_root/
├── eval.py
├── eval_prompt.txt
├── gt_dir/
│   ├── scene0000_00.json
│   ├── scene0001_00.json
│   └── ...
├── pred_dir/
│   ├── scene0000_00.json
│   ├── scene0001_00.json
│   └── ...
└── outputs/
```

Each prediction file should contain a top-level `qa_pairs` list. The evaluator matches GT and predictions by:

- **scene file name**
- **sample `id` inside `qa_pairs`**

#### Prediction file example

```json
{
  "dataset_info": {
    "model": "your_model_name"
  },
  "qa_pairs": [
    {
      "id": "scene0000_00_task1_0001",
      "question_type": "presence",
      "model_outputs": "Yes"
    }
  ]
}
```

### 5. Run Evaluation

```bash
python eval.py \
  --gt_dir /path/to/gt_dir \
  --pred_dir /path/to/pred_dir \
  --output_dir /path/to/output_dir \
  --eval_prompt ./eval_prompt.txt \
  --openai_api_key YOUR_OPENAI_API_KEY
```

If `OPENAI_API_KEY` is already set in your environment, you can omit `--openai_api_key`.

### 6. Output Files

The evaluator writes:

```text
output_dir/
├── results.jsonl
├── summary.json
└── per_scene_scores/
    ├── scene0000_00.json
    ├── scene0001_00.json
    └── ...
```

- `results.jsonl`: item-level evaluation records
- `summary.json`: compact overall summary, including micro and macro averages
- `per_scene_scores/`: one score file per scene

### 7. Scoring Summary

- **TPV**: exact-match yes/no scoring
- **NRI**: exact-match option-letter scoring
- **FSD**: LLM-as-a-judge scoring with the provided prompt template
- **SSP**: programmatic soft scoring over support surface, object identity, relation, and distance

## 🧰 Video Preparation Tools

The source assets from ScanNet++ and ScanNet v2 / ScanNet200 are not distributed as ready-to-use MP4 videos. For convenience, we provide:

- `tools/convert_mkv_to_mp4.py`
- `tools/convert_sens_to_mp4.py`

These scripts help convert source recordings into standard MP4 videos for downstream inference and evaluation pipelines.

<details>
<summary><strong>Citation</strong></summary>

Citation details will be added upon publication.

</details>
