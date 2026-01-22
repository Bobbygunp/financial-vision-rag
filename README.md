# 📊 Financial Vision-RAG: Multimodal 10-K Analyst

> **An end-to-end Vision RAG pipeline that "reads" complex financial documents as images, extracting GAAP metrics from unparsed PDF 10-K filings without OCR.**

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Model](https://img.shields.io/badge/Model-Llama_3.2_Vision-purple)
![Retrieval](https://img.shields.io/badge/Retrieval-ColPali-orange)
![Notebook](https://img.shields.io/badge/Format-Jupyter_Notebook-F37626)

## 💡 The Problem
Traditional RAG systems rely on text extraction (OCR), which often destroys the spatial context of financial documents.
* **Tables** get flattened into messy text streams.
* **Charts** and color-coded indicators are ignored entirely.
* **Footnotes** lose their association with specific rows.

## 🚀 The Solution
This project treats PDF pages as **images**. It uses a specialized Vision Retriever (**ColPali**) to find the most visually relevant page and a Vision Language Model (**Llama 3.2 11B**) to "look" at the page and extract data, preserving 100% of the layout context.

### Key Features
* **Visual Retrieval:** Uses `byaldi` (ColPali implementation) to index PDF pages as vector embeddings based on visual layout.
* **Vision-Reasoning:** Llama 3.2 Vision extracts data directly from charts, tables, and footnotes.
* **Interactive UI:** Contains an embedded Gradio app for real-time analysis directly within the notebook.
* **Failure Analysis:** Includes a documented experimentation log comparing retrieval depth strategies ($k=1$ vs $k=6$).

---

## 🔬 Performance Analysis & Engineering Trade-offs
*This project focuses on system optimization rather than just implementation. Extensive ablation studies were conducted to determine the optimal retrieval depth.*

### The "Attention Bottleneck" Discovery
During development, I experimented with feeding the model different numbers of retrieved pages (`k`) to solve "needle-in-a-haystack" queries.

| Retrieval Depth | Accuracy | Observation |
| :--- | :--- | :--- |
| **k=6 (High Recall)** | **50%** | **Failure:** The 11B model suffered from "Attention Dispersion." The noise from 5 irrelevant pages caused the model to confuse columns (e.g., extracting 2024 data instead of 2025). |
| **k=1 (High Precision)** | **70%** | **Success:** Restricting context to the single most relevant page forced the model to focus. While it missed some deep footnotes, the extraction fidelity for main tables increased significantly. |

### Visual Bias Mitigation
Early iterations showed the model preferred extracting data from colorful bar charts rather than accurate data tables.
* **Fix:** Engineered a "Negative Constraint" prompt protocol (*"Hierarchy of Truth: Tables > Charts"*) which successfully suppressed visual hallucinations and improved reasoning accuracy by 40%.

---

## 🛠️ Tech Stack
* **Language:** Python
* **Models:** `meta-llama/Llama-3.2-11B-Vision-Instruct`, `vidore/colpali-v1.2`
* **Libraries:** `transformers`, `torch`, `gradio`, `byaldi`, `pillow`
* **Hardware:** Developed on NVIDIA L4 GPU (Google Colab).

---

## 💻 How to Run

### Option 1: Google Colab (Recommended)
1. Download the `vision_rag.ipynb` file.
2. Upload it to Google Colab.
3. **Runtime > Change runtime type > T4 GPU** (or A100/L4 if available).
4. Run the first cell to install dependencies (`!pip install ...`).
5. Run the final cell to launch the Gradio UI.

### Option 2: Local Execution
*Note: Requires ~24GB VRAM for full precision.*
```bash
# Clone the repo
git clone [https://github.com/yourusername/financial-vision-rag.git](https://github.com/yourusername/financial-vision-rag.git)

# Launch Jupyter
jupyter notebook vision_rag.ipynb

# Download the pdf from the below link
https://drive.google.com/file/d/193avc6RlaRn0UkwErmmUEMUFK-Ek7o14/view?usp=sharing
