# NLP Assignment 2: Word Segmentation

**Name:** Ritweek Raj  
**Roll Number:** U24AI067  
**Assignment:** Word Segmentation using Greedy and Dynamic Programming Algorithms  

---

## 📌 Project Overview

This assignment addresses the **Word Segmentation Problem** (inserting spaces back into concatenated text) using a snapshot of the Brown Corpus. The task is to evaluate and compare two algorithms:
1. **Greedy Longest-Match Segmentation**
2. **Dynamic Programming Segmentation** using unigram log probabilities derived from word frequencies.

The performance of both algorithms is measured on **1,000 test cases** using **Exact Match Accuracy** and **Levenshtein Edit Distance**.

---

## 📂 Project Structure

```
Assignment_2/
├── text_segmentation_dataset.json  # Vocabulary with counts and 1000 test cases
├── assignment2.ipynb               # Fully executed Jupyter notebook with step-by-step logic
├── report.pdf                      # PDF version of the final project report
└── README.md                       # This documentation file
```

---

## 🛠️ Key Implementation Details

### 1. Log Probabilities (Avoiding Underflow)
Instead of multiplying raw probability fractions, which leads to floating-point underflow for longer strings, we convert frequencies to base-10 log probabilities:
$$\log_{10} P(\text{word}) = \log_{10}(\text{frequency}) - \log_{10}(\text{total\_corpus\_words})$$
$$P(\text{segmentation}) = \sum \log_{10} P(\text{word}_i)$$

### 2. Out-of-Vocabulary (OOV) Smoothing
For substrings or characters not present in the 1,500-word vocabulary, we assign a length-based penalty:
$$\log_{10} P(\text{OOV}) = -12.0 \times \text{len}(\text{word})$$
This ensures vocabulary words are heavily favored while maintaining robust segmentation paths.

### 3. Dynamic Programming Formulation (Backward Induction)
- **DP State**: Let `dp[i]` be the maximum log probability of segmenting the suffix `text[i:]`.
- **Recurrence**:
  $$dp[i] = \max_{i < j \le n} \left( \log_{10} P(\text{text}[i:j]) + dp[j] \right)$$
- **Base Case**: `dp[n] = 0.0` (empty suffix).
- We maintain a `parent` array where `parent[i] = j` stores the optimal split boundary to reconstruct the words.

---

## 📊 Final Results

Here is the comparison of the two approaches across all 1000 test cases:

| Metric | Greedy Algorithm | DP Algorithm |
| :--- | :---: | :---: |
| **Accuracy (Exact Match)** | **69.10%** | **98.20%** |
| **Average Edit Distance** | **1.2900** | **0.0280** |
| **Execution Time (1000 cases)** | ~0.18 seconds | ~0.40 seconds |

---

## 💡 Key Findings & Conclusions

1. **Why DP Outperforms Greedy**:
   - **Greedy** makes locally optimal choices at each index (matching the longest possible prefix). If a local choice leaves a sequence of characters that cannot be segmented into vocabulary words (or segments them poorly), it cannot backtrack or correct itself.
   - For example, in `itthatthecitytakestepstothisproblem`, the Greedy algorithm incorrectly segments `"takestepstothis..."` as `"takes t eps..."` because `"takes"` is in the vocabulary and is longer than `"take"`.
   - **Dynamic Programming** evaluates all segmentation options globally. It correctly determines that $P(\text{"take"}) \times P(\text{"steps"}) > P(\text{"takes"}) \times P(\text{"t"}) \times P(\text{"eps"})$ and chooses the correct segmentation.
2. **Smoothing Role**:
   - Incorporating OOV smoothing prevents the DP algorithm from rejecting segmentations with out-of-vocabulary characters entirely, making it highly robust.
