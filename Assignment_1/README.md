# NLP Assignment 1: Hindi Text Preprocessing & Tokenization

## 1. Assignment Objective
The objective of this assignment is to build a robust, custom text preprocessing and tokenization pipeline for Hindi using Python's regular expressions (`re`). The pipeline splits monolingual paragraph corpora into individual sentences, and then tokenizes each sentence into word-level tokens while protecting specific structural entities (URLs, emails, dates, decimals) from being incorrectly split. Finally, it calculates corpus-level statistics and stores the clean tokenized output.

---

## 2. Dataset Information
We use two large-scale monolingual corpora for Hindi:
1. **IndicCorp V2 (Hindi Subset):** Curated by AI4Bharat, containing millions of sentences. We load the `indiccorp_v2` config and `hin_Deva` split in streaming mode.
2. **OSCAR-2301 (Hindi Subset):** A massive, filtered multilingual corpus. We attempt to load the `hi` subset of `oscar-corpus/OSCAR-2301`.

---

## 3. Hindi Language Used
- **Language:** Hindi (हिन्दी)
- **Script:** Devanagari (देवनागरी)
- **Unicode Block:** `\u0900` to `\u097F` (Devanagari Unicode range)
- **Key Delimiter:** Danda (`।` - U+0964) and Double Danda (`॥` - U+0965) are utilized as sentence delimiters and separated as individual tokens.

---

## 4. Tokenizer Design
The custom tokenizer is divided into two sequential parts:
1. **Sentence Tokenizer (`sentence_tokenizer`)**:
   - Identifies sentence-ending boundaries: Period (`.`), Question Mark (`?`), Exclamation (`!`), and Danda (`।`).
   - Before splitting, it scans for "protected entities" (URLs, emails, dates, and decimals) to ensure any delimiters inside them (e.g. `.` in URLs, decimals, and emails; `?` or `/` in URLs/dates) are not treated as sentence boundaries.
2. **Word Tokenizer (`word_tokenizer`)**:
   - Tokenizes a single sentence into words.
   - Preserves URLs, emails, dates, and decimals as single, atomic tokens.
   - Extracts Hindi words (using Devanagari range, excluding punctuation like `।`).
   - Extracts English words and standard numbers.
   - Treats each punctuation mark as an individual token.
   - Uses a fallback `\S` match to guarantee that no characters are silently dropped.

---

## 5. Regex Logic

### Protected Patterns
- **URL Pattern:** `https?://[^\s!।]*[^\s!?।.,:;\'"`\(\)\[\]\{\}]`
  Matches HTTP/HTTPS URLs including paths/queries, but excludes trailing sentence boundary punctuation.
- **Email Pattern:** `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`
  Standard email address pattern.
- **Date Pattern:** `\b\d{1,2}[-/.]\d{1,2}[-/.]\d{2,4}\b`
  Captures dates using `-`, `/`, or `.` as separators (e.g. `12/05/2025`).
- **Decimal Pattern:** `\b\d+\.\d+\b`
  Captures numbers with decimal points (e.g., `3.14`).

### Word Tokenizer Patterns
- **Hindi Word:** `[\u0900-\u0963\u0970-\u097F]+`
  Matches Devanagari letters and matras, excluding Devanagari numbers and the danda delimiters (`।` and `॥`).
- **English Word:** `[a-zA-Z]+`
  Matches standard Latin alphabets.
- **Numbers:** `[\d\u0966-\u096f]+`
  Matches both standard Western digits (`0-9`) and Devanagari digits (`०-९`).
- **Punctuation:** `[^\w\s\u0900-\u097F]|।|॥`
  Matches all symbols, non-word characters, and Devanagari dandas as individual tokens.

---

## 6. Folder Structure
```text
Assignment_1/
├── Assignment1.ipynb       # Jupyter Notebook containing code cells
├── README.md               # Documentation and details
├── requirements.txt        # Python package dependencies
├── indic_tokenized.parquet # Tokenized sentences (IndicCorpV2)
├── indic_stats.txt         # Corpus statistics (IndicCorpV2)
└── venv/                   # Python Virtual Environment
```

---

## 7. Installation Instructions
To set up the workspace:
```bash
# 1. Create a dedicated virtual environment
python3 -m venv venv
source venv/bin/activate

# 2. Install the exact compatible package versions
pip install -r requirements.txt

# 3. Register the virtual environment as a Jupyter kernel
python -m ipykernel install --user --name=nlp-assignment1 --display-name "Python (NLP Assignment 1)"
```

---

## 8. How to Run
You can run the Jupyter Notebook in two ways:

1. **Via UI (VS Code / JupyterLab)**:
   - Open `Assignment1.ipynb` in your editor.
   - Select the kernel **"Python (NLP Assignment 1)"** (or `nlp-assignment1`) from the kernel selector dropdown in the top-right corner.
   - Run the cells.

2. **Headlessly (CLI)**:
   - Run the following command inside the activated virtual environment:
     ```bash
     jupyter nbconvert --to notebook --execute --inplace Assignment1.ipynb
     ```

---

## 9. Output Files
- **`indic_tokenized.parquet`**: A Parquet file containing a single column `sentence` where each row is a space-joined tokenized Hindi sentence.
- **`indic_stats.txt`**: A text file summarizing statistics for the IndicCorp V2 corpus.
- **`oscar_tokenized.parquet`** (if run with access): Parquet file containing tokenized OSCAR-2301 sentences.
- **`oscar_stats.txt`** (if run with access): Text file summarizing statistics for the OSCAR corpus.

---

## 10. Corpus Statistics
Below are the calculated statistics from processing `50,000` sentences of **IndicCorp V2 (Hindi)**:

| Metric | Value |
| :--- | :--- |
| **Total Sentences** | 50,000 |
| **Total Words** | 919,334 |
| **Total Characters** | 3,403,766 |
| **Average Sentence Length** | 18.39 words |
| **Average Word Length** | 3.70 characters |
| **Type Token Ratio (TTR)** | 0.0535 |

---

## 11. Limitations & Authentication (OSCAR Gating)
- **OSCAR-2301 Access**: The OSCAR-2301 dataset is gated. To run it successfully:
  1. Visit the Hugging Face dataset page: [OSCAR-2301](https://huggingface.co/datasets/oscar-corpus/OSCAR-2301).
  2. Request access and accept their license agreement.
  3. Authenticate locally by running `hf auth login` (or `huggingface-cli login`) and entering your Hugging Face access token.
  4. The notebook catches loading failures gracefully and prints instructions rather than crashing, making it safe to run even if access is not yet approved.
