# 🚗 AI-Based Dashboard Image Validation System

## 📘 Problem Statement

In automotive testing, **dashboard UI validation** involves comparing **reference** and **toCheck** images from regression test reports (e.g., `globalTestReport.html`) to detect visual, textual, or scaling mismatches.  
Manual inspection of these image pairs is **time-consuming** and **error-prone**, especially across variants and languages (like French and English).  

This project automates that process using **AI models** to calculate image mismatch percentages, categorize results, and generate an **annotated, human-readable HTML report**.

---

## 🎯 Objective

Develop a Python-based automated tool that:
1. Reads `globalTestReport.html` (generated from automotive UI tests).  
2. Detects and quantifies visual mismatches between image pairs using **CLIP** and **segment-wise comparison**.  
3. Detects **textual mismatches** using **EasyOCR** (with support for French and English).  
4. Handles **missing**, **scaled**, or **corrupt** images gracefully.  
5. Enhances low-resolution images using **Real-ESRGAN** before OCR.  
6. Generates a **new annotated HTML report** (`aiAnalysisReport_<variant>.html`) that:  
   - Groups image comparisons by **AI-determined status** (`OK`, `NOK`, `Investigate`, `Faulty`).  
   - Includes a **clickable summary table** at the top.  
   - Lists **scenario scripts** above each image section.  
   - Adds **KO segment extraction** for deeper scenario analysis.  

---

## 🧠 Approach Overview

| Stage | Description |
|-------|--------------|
| **1. Input Parsing** | Reads `config_path.txt` and `thresholds.txt` to configure variant, report paths, and thresholds. |
| **2. Image Extraction** | Extracts image pairs (`toCheck` vs. `Reference`) from `globalTestReport.html`. |
| **3. Preprocessing** | Enhances and enlarges images using **Real-ESRGAN** or bicubic scaling for better OCR/CLIP performance. |
| **4. Text Comparison** | Uses **EasyOCR** to detect and compare textual differences (especially for warning messages like “BUZ”). |
| **5. Visual Comparison** | Uses **CLIP (ViT-B/32)** to compute semantic similarity and mismatch percentages, both globally and segment-wise. |
| **6. Status Assignment** | Applies configurable thresholds to classify image pairs as `OK`, `Investigate`, `NOK`, or `Faulty`. |
| **7. Report Generation** | Inserts model results into HTML, builds a summary table, groups sections by status, and appends KO segment analysis. |

---

## 🧩 Folder Structure

Your repository follows this structure:

```
📂 ProjectRoot
 ┣ 📂 input/
 │   ┣ 📂 VariantName/
 │   │   ┣ globalTestReport.html
 │   │   ┣ 📂 images/
 │   │   ┗ other assets...
 ┣ 📂 output/
 │   ┣ aiAnalysisReport_<variant>.html
 │   ┣ output_<variant>.csv
 │   ┗ OCR_log_<variant>.txt
 ┣ 📜 main.ipynb
 ┣ 📜 thresholds.txt
 ┣ 📜 config_path.txt
 ┣ 📜 requirements.txt
 ┗ 📜 README.md
```

---

## ⚙️ Installation and Setup

### 1. Clone Required Dependencies

Some libraries are **not pip-installable** and must be cloned manually:

| Library | GitHub Repository | Path to Update in Code |
|----------|------------------|------------------------|
| **Real-ESRGAN** | [https://github.com/xinntao/Real-ESRGAN](https://github.com/xinntao/Real-ESRGAN) | `Path_to/Real-ESRGAN-master/realesrgan/weights/RealESRGAN_x4plus.pth` |
| **BasicSR (v1.4.2)** | [https://github.com/xinntao/BasicSR/tree/v1.4.2](https://github.com/xinntao/BasicSR/tree/v1.4.2) | Required by Real-ESRGAN |
| **CLIP** | [https://github.com/openai/CLIP](https://github.com/openai/CLIP) | `Path_to/CLIP-main/clip/model` |
| **EasyOCR** | [https://github.com/JaidedAI/EasyOCR](https://github.com/JaidedAI/EasyOCR) | `Path_to/EasyOCR-master/easyocr/weights` |

> ⚠️ **Note:** Update these paths in your main script wherever `"Path_to/"` appears.

### 2. Install Python Requirements

```bash
pip install -r requirements.txt
```

You may also need to install system-level dependencies for `torch`, `numpy`, and `Pillow` depending on your environment.

---

## 🚀 Running the Code

1. Place your input files:
   ```bash
   ./input/<VariantName>/globalTestReport.html
   ./input/<VariantName>/images/
   ```

2. Edit `config_path.txt`:
   ```ini
   variant = VariantName
   report_path_template = ./input/{variant}/globalTestReport.html
   image_dir_template = ./input/{variant}/images
   output_dir = ./output
   relative_image_path_template = ../input/{variant}/images
   ```

3. Edit `thresholds.txt` to define model thresholds per variant.

4. Run the script:
   ```bash
   python main.ipynb
   ```
   (or convert to `.py` if using as a standalone executable)

5. The tool will generate:
   - ✅ `aiAnalysisReport_<variant>.html` → annotated AI-enhanced HTML report  
   - 📊 `output_<variant>.csv` → model metrics summary  
   - 🧾 `OCR_log_<variant>.txt` → OCR detection logs  

---

## 📊 Output Highlights

- **Clickable Summary Table:** Navigate directly to `OK`, `NOK`, `Investigate`, or `Faulty` sections.
- **Grouped Comparisons:** Images are grouped under each AI status.
- **Scenario Context:** Each image block includes its corresponding test scenario.
- **KO Segment Extraction:** Lists KO checkpoints from the original report.
- **Enhanced Visuals:** Text-rich or low-res images are super-resolved using Real-ESRGAN.

---

## 🧠 Key Libraries and Their Roles

| Library | Purpose |
|----------|----------|
| **PyTorch** | Runs all neural network computations (CLIP, Real-ESRGAN, etc.) |
| **OpenAI CLIP** | Measures visual-semantic similarity between image pairs |
| **Real-ESRGAN** | Super-resolves blurry images for better OCR and visual analysis |
| **EasyOCR** | Detects and compares on-screen text (supports multilingual OCR) |
| **BeautifulSoup4** | Parses and manipulates `globalTestReport.html` |
| **Pandas** | Builds the summary CSV of image pair results |
| **Pillow (PIL)** | Handles image loading, resizing, and format conversions |
| **NumPy** | Image-to-array transformations and numerical operations |

---

## 📎 Notes

- Works with both **Renault** and **Nissan** regression test formats.
- Supports **English + French OCR**.
- CUDA acceleration is optional but improves performance.
- Duplicate section and header cleanup is handled automatically.
- The tool gracefully skips missing or incompatible image pairs.

---

## 🧾 Example Output

**Generated HTML Report Structure:**

```
AI Global Report Analysis
├── Summary Table (clickable)
├── Status Legend
├── Report:
│   ├── OK Image Comparisons
│   ├── Investigate Image Comparisons
│   ├── NOK Image Comparisons
│   └── Faulty Image Comparisons
└── Scenario Segments with KO Checks
```

---

## 👨‍💻 Author

**Kanishk Mishra**  
*Former* AI Intern, Renault-Nissan Technology & Business Centre India (RNTBCI)  
Specializing in AI-driven automotive UI validation and image-based test automation.
